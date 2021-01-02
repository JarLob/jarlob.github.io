---
layout:     post
title:      "HTTP Header Enrichment (pen)testing tutorial"
date:       2020-05-26 12:00:00
categories: HackAndTell MobileInternet
cover:      
permalink:  /en/blog/http-header-enrichment-tutorial
---
Since I believe [the issue I wrote previously about](https://jarlob.github.io/en/blog/mobile-data-leak) is not limited to Lithuania, but I physically cannot check all mobile operators in the world I decided to write a short tutorial you can follow to check the situation in your country.

1. Use any server you like that reflects back request headers to see what headers are injected. Below is an example with the postman service. (*Notice that the address is not https.* You may also try not only GET, but POST and other types of requests).<br/><br/>
```
curl http://postman-echo.com/get
```
Issue the request over cable internet and then over mobile internet to compare the results and see the injected headers. If you find some constant headers injected at this step, congratulations - you just caught your provider attaching a super cookie to your identity. But this is not the max that can be achieved. See step 4.  
2. If nothing suspicious was found:
    * Identify a mobile app that sometimes asks you to switch to mobile internet instead of WIFI. It can be some local Android/iOS application specific to your country.  
Next step is to reverse engineer it - decompile it in order to understand what addresses it calls or install it on a rooted phone to intercept HTTP traffic it makes.  
The goal is to identify the special addresses it calls over unencrypted http request that a mobile provider intercepts to inject special headers.<br/><br/>
It is possible that the web endpoint is a vulnerability itself. Since the mobile provider acts as a man in the middle there is a chance the response is not properly protected because of CORS misconfiguration. In that case the information returned by the http call can be stolen by a javascript from any malicious site.<br/><br/>
    * The other option identifying the special address is to check if a self-care web site of the mobile provider logs you in automatically when you make connection to it like `http://selfcare.purple-mobile.com`. (*Notice that the address is not https.*) It is also possible to find other special addresses that do automatic mobile user recognition like `http://topup.purple-mobile.com` or any other third party domain.

3. Once you have the address repeat step 1. but specify the identified web host as in example below:<br/><br/>
```
curl http://postman-echo.com/get --header "Host: selfcare.purple-mobile.com"
```
It is possible you'll be able to see the injected headers because the provider checks only the `Host:` header instead of the real destination.

4. If you found the headers, try to issue the identified http requests from step 2. with modified headers and see if it bypasses authorization logic in the web service. You may try it from the same mobile network to see if the spoofed value is not overwritten and from a non-mobile network provider. The service may accept your supplied header value even if you are connecting not from a mobile network. The technique allowed authorization bypass in [Tele2 Lithuania selfcare](http://mano.tele2.lt). It can be done with curl like:<br/><br/>
```
curl http://identified.address.com --header "X-Identified-Header: spoofed_value"
```
or by setting a Web Proxy of your choice to inject all request with your header. For example in Burp:
[![Burp match and replace header setting](burp_header_inject.png "Burp match and replace header setting")](burp_header_inject.png)