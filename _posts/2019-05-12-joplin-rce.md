---
layout:     post
title:      "Joplin ElectronJS based Client: from XSS to RCE"
date:       2019-05-12 12:00:00
categories: HackAndTell ElectronJS XSS RCE
cover:      electron.jpg
permalink:  /en/blog/joplin-electron-rce
---
One day I was reading the master's thesis ["Analysis of Electron-Based Applications..."](https://digi.lib.ttu.ee/i/file.php?DLID=9890&t=1) by Silvia Väli. She basically applied the [auditing checklist](https://www.blackhat.com/docs/us-17/thursday/us-17-Carettoni-Electronegativity-A-Study-Of-Electron-Security-wp.pdf) by Luca Carettoni, searched GitHub projects for some interesting keywords and then manually reviewed and tested for vulnerabilities. The hypothesis of her thesis was that since Electron framework has many insecure defaults there must be significant number of vulnerable applications. It appeared to be true and she scored a bunch of nice CVEs.

Personally I find vulnerabilities in Electron based applications interesting because of the lack of process isolation, that can easily lead from a "simple" XSS to Code Execution on the running machine. So I questioned myself how the developers fixed these vulnerabilities and started looking for commit details... It turns out many developers didn't mitigate the root cause of Code Execution and just fixed XSSes. It is just a matter of finding another XSS to pop a calc in the applications.

One of the applications, [Joplin](https://joplinapp.org/), was different. The case is curious because it shows how many gotchas a developer should keep in mind when using Electron framework. At first the developer [disabled HTML in Markdown at all](https://github.com/laurent22/joplin/commit/494e235e18659574f836f84fcf9f4d4fcdcfcf89#diff-c90b6228d7bed30727231ebdafd07753), but later he [disabled Node.js integration in the WebView used for Markdown and re-enabled HTML rendering](https://github.com/laurent22/joplin/commit/df302206ddefae9b9e6164c16dcb501dc1c02b5e). No wonder that soon somebody [reported a XSS again](https://github.com/laurent22/joplin/issues/740), but since this time the researcher didn't manage to escape the sandbox the issue was closed. However when I found the XSS myself I noticed that although node integration was disabled the context isolation wasn't enabled for the WebView. It reminded me about [a presentation](https://speakerdeck.com/masatokinugawa/electron-abusing-the-lack-of-context-isolation-curecon-en) by Masato Kinugawa. The researcher found that if context isolation is not enabled it is possible to override a JavaScript function from restricted rendering process, then trigger the privileged main process to call it. I've put the following payload into markdown editor:
```js
<img src onerror="Function.prototype.call=function(process){
process.mainModule.require('child_process').execSync('calc');
}
location.reload();">
```
and successfully popped a calc:

![Calculator in Joplin](joplin.png)

The malicious document could have been delivered to the victim machine through synchronization. Joplin supports synchronization with WebDAV providers, network shares or cloud storages like OneDrive (the attacker would need to compromise OneDrive credentials first in that case).

I contacted the project owner about the vulnerability. First, after some discussion I was told there are no resources to fix the issue. Doh.. I really hated the idea to disclose an unpatched vulnerability. But after some time a commit was made to [enable context isolation and adjustments were made to inter-process communication](https://github.com/laurent22/joplin/commit/72af5643828e8a220ea8ca5ff9831f42f01895b6) and the issue was fixed in [v1.0.109](https://github.com/laurent22/joplin/releases/tag/v1.0.109).

>P.S. I tried to register CVE for the vulnerability, but IMHO the process is broken and convoluted. First I had to wait few weeks for it to even start being processed. Then it got stuck for two months in the [registration excel](https://docs.google.com/spreadsheets/d/1PlDOsZ4Q36JU4Dz9zyBB2F3814dScppCRCe1muCT7JI). During the time nobody showed any initiative to contact me, although I left my email address in the registration. When I wrote an email asking why it is on hold I was told that they "need reference url(s) that explain the problem clearly ... (e.g. github issue reporting it, that kind of thing)". It is a security vulnerability so [there are no details in the issue](https://github.com/laurent22/joplin/issues/789) and apparently the description I have provided and [public acknowledgment in release notes](https://github.com/laurent22/joplin/releases/tag/v1.0.109) is not enough. I think it was the first and the last time I register a CVE :)

Timeline:  
2018.09.11 - Created an [issue](https://github.com/laurent22/joplin/issues/789), got contacts and sent an email to the project owner.  
2018.09.17 - Filled a registration form for CVE.  
2018.09.17 - Pinged the owner of Joplin and got a reply that there are no resources for the fix.  
2018.09.25 - Received a message that the issue is fixed and will be released in the next version.  
2018.09.27 - [v1.0.109](https://github.com/laurent22/joplin/releases/tag/v1.0.109) was released.  
2018.11.19 - Sent a question about CVE status.  