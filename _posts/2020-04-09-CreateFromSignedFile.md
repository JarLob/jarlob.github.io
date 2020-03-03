---
layout:     post
title:      "Authenticode verification vulnerability pattern"
date:       2020-04-09 12:00:00
categories: HackAndTell Cryptography
cover:      authenticode.png
permalink:  /en/blog/Authenticode-verification-vulnerability-pattern-CreateFromSignedFile
---
There is a common vulnerability pattern in various implementations of authenticode signature verification in .NET. It was found [multiple times](https://blogs.msdn.microsoft.com/ace_team/2009/01/25/vulnerabilities-in-web-applications-due-to-improper-use-of-crypto-part-3/) in [different](https://bugs.chromium.org/p/project-zero/issues/detail?id=1660) [products](https://www.reddit.com/r/netsec/comments/akrsi7/exploit_for_check_point_zonealarm_antivirus/efb3o86/) in the past and I have seen it myself during security audits.

It's name is `X509Certificate.CreateFromSignedFile`. Frankly saying it is hard to blame developers - .NET Framework libraries are quite mature and there is API for almost anything you ever needed. Yet as far as I know the only way to verify the signature is to P/Invoke `WinVerifyTrust`. It is inconvenient and hacky for an average .NET developer, so when they are tasked with adding authenticode signature verification of an executable the method `X509Certificate.CreateFromSignedFile` catches their eye.

On top of that internet is a mix of [good advises](https://stackoverflow.com/questions/24060009/checking-digital-signature-on-exe   ) and [misleading](https://www.drdobbs.com/extracting-digital-signatures-from-signe/184416921)(to say the least) [articles](https://www.kunal-chowdhury.com/2017/01/csharp-code-to-detect-whether-binaries-are-digitally-signed.html) (some even [blogged](https://blogs.msdn.microsoft.com/windowsmobile/2006/05/17/programmatically-checking-the-authenticode-signature-on-a-file/) by Microsoft). So the developers usually come with something like:
```cs
try
{
    var cert = X509Certificate.CreateFromSignedFile(filePath);
    var cert2 = new X509Certificate2(cert);
    return cert2.Verify();
}
catch // or catch CryptographicException
{
    return false;
}
```

Testing shows that the code correctly returns `true` for properly signed executables and `false` for the ones without a signature. When in fact it is vulnerable.  
The flaw here is that `X509Certificate.CreateFromSignedFile` doesn't verify if a signature of the executable is valid. It only extracts the certificate from the file and verifies the certificate itself. Attackers may tamper valid signed executables or transfer a valid certificate to an unsigned file. There are tools for transplanting certificates from one executable to another like [SigPirate](https://github.com/xorrior/Random-CSharpTools/tree/master/SigPirate) or [SigThief](https://github.com/secretsquirrel/SigThief).

I decided to look around for the pattern in popular open-source projects and have found the vulnerability in two of them.

The first one was in [Dynamo BIM by Autodesk](https://dynamobim.org/). The vulnerable code can be found [here](https://github.com/DynamoDS/Dynamo/blob/767577c6c0de187b64fc3dda3ff012f16c008208/src/DynamoUtilities/CertificateVerification.cs#L44). As a PoC I have built an unsigned .NET assembly and used SigPirate to transplant a certificate from one of Microsoft signed assemblies. Autodesk has issued CVE-2020-7079 for the vulnerability and released an [advisory](https://www.autodesk.com/trust/security-advisories/adsk-sa-2020-0001). The diff of the fix can be found [here](https://github.com/DynamoDS/Dynamo/pull/10421/files).

The other vulnerable code was in [SoundSwitch](https://github.com/Belphemur/SoundSwitch) project. The vulnerable code can be found [here](https://github.com/Belphemur/SoundSwitch/blob/2331d90669e63d14ea5c2784a71e33d39eaf59ff/SoundSwitch/Framework/Updater/SignatureChecker.cs#L29-L47). There were two options to bypass the check. The first one was to transplant the certificate from the official SoundSwitch executable. The second one - to create a self signed certificate with arbitrary `Subject` and `Issuer`:  
* Run powershell `New-SelfSignedCertificate -Type CodeSigningCert -certstorelocation cert:\CurrentUser\my -Subject "CN=Antoine Aflalo soundswitch" -Issuer "CN=Certum"`
* export the certificate to a file cert.pfx
* signtool.exe sign /p a /f cert.pfx Legit.exe

The signature wouldn't be valid on other machine, but it would pass `IsCertumSigned` and demonstrates possible `Subject` and `Issuer` collisions with other certificates. The diff of the fix can be found [here](https://github.com/Belphemur/SoundSwitch/commit/2723a860421055addceaafca8b42e9415bf1f679).

### Timeline
2020.02.19 - Report to the owner of SoundSwitch is sent.  
2020.02.20 - Report to Autodesk is sent.  
2020.03.01 - Autodesk merges a [fix](https://github.com/DynamoDS/Dynamo/pull/10421/files).  
2020.03.05 - Dynamo BIM version 2.5.2 is released.  
2020.04.01 - Autodesk releases the [advisory](https://www.autodesk.com/trust/security-advisories/adsk-sa-2020-0001) and assigns CVE-2020-7079.  
2020.04.04 - SoundSwitch version [5.0.2](https://github.com/Belphemur/SoundSwitch/releases/tag/v5.0.2) is released.  