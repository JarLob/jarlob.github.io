---
layout: page
title: About me 
---
I am Application Security Researcher, static code analysis enthusiast, former Application Security Engineer, former Software Engineer with a solid development background. I have been on many sides: as a software developer, internal security auditor, external reporter, bug bounty submitter and triager.

I have started as a desktop and backend C++/C# developer. While one part of me enjoyed creating programs, my other passion always was finding cracks and breaking things. After some time I've moved into the Information Security field where I feel I found a balance: I enjoy writing security tools and performing technical security assessments. In my free time I like researching security of third party products (both closed and open source).  

Open source projects I am or was actively developing:  

[Security Code Scan](https://security-code-scan.github.io/) - Security static code analysis for C# and VB.NET.  
[Electronegativity](https://github.com/doyensec/electronegativity) - Vulnerability patterns detector for JavaScript/TypeScript Electron applications.  

You can find me on [Twitter](https://twitter.com/yarlob), [GitHub](https://github.com/JarLob), [Linkedin](https://www.linkedin.com/in/yarlob/) and you can reach me at {{ site.email }}

### Disclosures:
Path traversal in [youtube-dl](https://securitylab.github.com/advisories/GHSL-2024-089_youtube-dl/) and [yt-dlp](https://securitylab.github.com/advisories/GHSL-2024-090_yt-dlp/) leading to RCE - [CVE-2024-38519](https://nvd.nist.gov/vuln/detail/CVE-2024-38519)  
Insufficient markdown sanitization in nuget.org - [CVE-2024-37304](https://securitylab.github.com/advisories/GHSL-2024-016_NuGetGallery/)  
LDAP injection in Redash - [CVE-2020-36144](https://securitylab.github.com/advisories/GHSL-2024-009_Redash/)  
Several memory access violations in stb_image and stb_vorbis - [CVE-2023-45676 and others](https://securitylab.github.com/advisories/GHSL-2023-145_GHSL-2023-151_stb_image_h/)  
Buffer Overflow in [uchardet](https://securitylab.github.com/advisories/GHSL-2023-105_uchardet/)  
Buffer Overflows in Notepad++ - [CVE-2023-40031, CVE-2023-40036, CVE-2023-40164, CVE-2023-40166](https://securitylab.github.com/advisories/GHSL-2023-092_Notepad__/)  
Stack exhaustion in jsonxx - [CVE-2022-23460](https://securitylab.github.com/advisories/GHSL-2022-049_Jsonxx/)  
Double free in jsonxx - [CVE-2022-23459](https://securitylab.github.com/advisories/GHSL-2022-048_Jsonxx/)  
Deserialization vulnerability in Orckestra C1 CMS - [CVE-2022-24789](https://securitylab.github.com/advisories/GHSL-2022-001_Orckestra_C1_CMS/)  
Arbitrary file write during TAR extraction in Apache Hadoop - [CVE-2022-26612](https://securitylab.github.com/advisories/GHSL-2022-012_Apache_Hadoop/)  
Path traversal in the OWASP Enterprise Security API (ESAPI)- [CVE-2022-23457](https://securitylab.github.com/advisories/GHSL-2022-008_The_OWASP_Enterprise_Security_API/)  
Partial path traversal in [Apache Felix Atomos](https://securitylab.github.com/advisories/GHSL-2022-007_Apache_Felix_Atomos/)  
Partial path traversal in Apache Karaf - [CVE-2022-22932](https://securitylab.github.com/advisories/GHSL-2022-005_006_Apache_Karaf/)  
Partial path traversal in [Apache Pinot](https://securitylab.github.com/advisories/GHSL-2022-004_Apache_Pinot/)  
Partial path traversal in Apache James Server - [CVE-2022-22931](https://securitylab.github.com/advisories/GHSL-2022-002_GHSL-2022-003_Apache_James_Server/)  
Path traversal in SharpZipLib - [CVE-2021-32840, CVE-2021-32841, CVE-2021-32842](https://securitylab.github.com/advisories/GHSL-2021-125-sharpziplib/)  
Path traversal in SharpCompress - [CVE-2021-39208](https://securitylab.github.com/advisories/GHSL-2021-082-sharpcompress/)  
Arbitrary File Creation, Arbitrary File Overwrite, Arbitrary Code Execution in npm/arborist- [CVE-2021-39135](https://github.com/advisories/GHSA-gmw6-94gg-2rc2)  
Arbitrary File Creation/Overwrite on Windows via insufficient relative path sanitization in npm/node-tar - [CVE-2021-37713](https://github.com/advisories/GHSA-5955-9wpr-37jh)  
Arbitrary File Creation/Overwrite via insufficient symlink protection due to directory cache poisoning using symbolic links in npm/node-tar - [CVE-2021-37712](https://github.com/advisories/GHSA-qq89-hq3f-393p)  
Unauthenticated file read in Emby - [CVE-2021-32833](https://securitylab.github.com/advisories/GHSL-2021-051-emby/)  
Unauthenticated arbitrary file read in Jellyfin - [CVE-2021-21402](https://securitylab.github.com/advisories/GHSL-2021-050-jellyfin/)  
Remote Code Execution and Local Elevation of Privileges in [GoSign App](https://blog.devsecurity.eu/en/blog/registru-centras-gosign-vulnerabilities)  
Weak JSON Web Token (JWT) signing secret in YApi - [CVE-2021-27884](https://securitylab.github.com/advisories/GHSL-2020-228-YMFE-yapi/)  
Undocumented template expression evaluation in the gajira-comment GitHub action - [CVE-2020-14189](https://securitylab.github.com/advisories/GHSL-2020-173-gajira-comment-action/)  
Undocumented template expression evaluation in the gajira-create GitHub action - [CVE-2020-14188](https://securitylab.github.com/advisories/GHSL-2020-172-gajira-create-action/)  
Remote code execution (RCE) and elevation of privileges (EoP) in SmartStoreNET - [CVE-2020-27996, CVE-2020-27997](https://securitylab.github.com/advisories/GHSL-2020-138-139-SmartstoreAG-SmartStoreNET/)  
Arbitrary code execution in DatabaseSchemaReader - [CVE-2020-26207](https://securitylab.github.com/advisories/GHSL-2020-141-martinjw-dbschemareader/)  
Arbitrary Code Execution in FastReports - [CVE-2020-27998](https://securitylab.github.com/advisories/GHSL-2020-143-FastReportsInc-FastReports/)  
SQL Injection in Mailtrain - [CVE-2020-24617](https://securitylab.github.com/advisories/GHSL-2020-132-Mailtrain/)  
Path traversal vulnerability in Adobe git-server - [CVE-2020-9708](https://securitylab.github.com/advisories/GHSL-2020-133-Adobe-Git-server/)  
Local privilege elevation vulnerability in Composer Windows installer - [CVE-2020-15145](https://github.com/composer/windows-setup/security/advisories/GHSA-wgrx-r3qv-332c)  
Authenticode signature validation bypass in [Autodesk Dynamo BIM (CVE-2020-7079) and SoundSwitch](/en/blog/Authenticode-verification-vulnerability-pattern-CreateFromSignedFile)  
Authorization bypass in [Tele2.lt self service website](https://blog.devsecurity.eu/lt/blog/tele2-savitarnos-autorizacijos-apejimas)  
Arbitrary code execution in [Resource.NET](https://fishcodelib.com/Resource.htm) [(not fixed)](/en/blog/dnspy-deserialization-vulnerability)  
Arbitrary code execution in [dnSpy](/en/blog/dnspy-deserialization-vulnerability)  
Path Traversal in [Aspose.ZIP for .NET](https://docs.aspose.com/display/zipnet/Aspose.ZIP+for+.NET+19.1+Release+Notes)  
RCE in [Joplin desktop client](https://github.com/laurent22/joplin/releases/tag/v1.0.109)  
SQL injection in [Xataface](https://github.com/shannah/xataface/releases/tag/2.2.3). [(The fix)](https://github.com/shannah/xataface/commit/eb4265e7188b715cc6c886f19f4332fd4b4346ca)  
SQL injection in [PHP-MySQLi-Database-Class](https://github.com/ThingEngineer/PHP-MySQLi-Database-Class/issues/823)  
