---
layout:     post
title:      "Vulnerable deserialization in dnSpy and Resource.NET"
date:       2019-05-13 12:00:00
categories: HackAndTell Deserialization dnSpy Resource.NET
cover:      bart-simpson-generator.png
permalink:  /en/blog/dnspy-deserialization-vulnerability
---
I find deserialization vulnerabilities somewhat special and beautiful. The fact you can execute your own code (sometimes remotely) in an application written in a managed language and it doesn't even take memory corruption or stack overflow is mind blowing. Because of that I'm very interested in this kind of vulnerabilities, especially in .NET based applications. No wonder a [blog post](https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2018/august/aspnet-resource-files-resx-and-deserialisation-issues/) by Soroush Dalili about deserialization of .NET resources caught my attention.

 The researcher noticed, that .NET resource file formats - both compiled (.resources) and not (.resx) store serialized objects inside of them. So if an attacker can tamper the file, he can execute arbitrary code during deserialization when the files are opened. Users don't think resource files can be malicious. Usually files with these extensions are opened in code editors, but I immediately thought about .NET decompilers. Unfortunately or thankfully :) I didn't notice that Soroush already found these vulnerabilities in [.NET Reflector](https://www.nccgroup.trust/uk/our-research/technical-advisory-code-execution-by-viewing-resource-files-in-net-reflector/) and [ILSpy](https://github.com/icsharpcode/ILSpy/issues/1196).

First I needed to create an executable for testing. So I've created an empty C# project in Visual Studio and added a string resource. It is available on [GitHub](https://github.com/JarLob/EvilResx). For PoC payload generation I've used [ysoserial.net](https://github.com/pwntester/ysoserial.net) by Alvaro Muñoz.

![Payload](resx.png)

After the compilation I've got the [EvilResx.exe](EvilResx.exe) for testing. There are differences in resource handling between decompilers. ILSpy for example doesn't deserialize resources:

![ILSpy resource view](ilspy_resource.png){: .align-left}

Telerik JustDecompile gives a warning before opening a resource file instead:

![JustDecompile warning](justdecompile_resource_warning.png){: .align-left}

When I opened [dnSpy](https://github.com/0xd4d/dnSpy) and expanded resources it didn't warn me, but a calculator popped up:

![dnSpy calc popped](dnspy_resource.png){: .align-left}

The developer of dnSpy pointed out there is a setting I wasn't aware of in options dialog with a note that it is unsafe:

![dnSpy options](dnspy_options.png){: .align-left}

But for some reason the setting was ON by default. He decided to remove the setting completely and immediately released a new version with the fix 5.0.11.

After that I tried to find what else potentially vulnerable resource editors are out there. One application I found was [Resource .NET](https://fishcodelib.com/Resource.htm). It allows editing .resx and .resources files. I used [EvilResx.ClickMe.resources](EvilResx.ClickMe.resources) from intermediate build folder and [ClickMe.resx](ClickMe.resx) from sources of my [PoC project](https://github.com/JarLob/EvilResx). However after contacting the owner he claimed it is a vulnerability in Microsoft .NET Framework [ResourceReader class](https://github.com/dotnet/corefx/blob/master/src/Common/src/CoreLib/System/Resources/ResourceReader.cs) :). I've sent him multiple ideas how it could be fixed:
1. See if the file contains a serialized object and show a warning.  
2. Check if the file has the [Mark of the Web](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/compatibility/ms537628(v=vs.85)) that indicates it was downloaded and show a warning (as Visual Studio does).  
3. Implement custom .resources parser as [ILSpy did](https://github.com/icsharpcode/ILSpy/commit/c17c3c739f339563749f73f0a4f2d1d65516c797).  

But as far as I know nothing was done.

Timeline:  
2018.12.17 - Reported the issue to the author of dnSpy.  
2018.12.18 - Six(!) hours later a new release v5.0.11 with a fix was made.  
2018.12.18 - Reported the vulnerabilty to the author of [Resource .NET](https://fishcodelib.com/Resource.htm).  
2018.12.18 - Resource .NET replied it is a vulnerability in Microsoft framework.  
2019.05.xx - Resource .NET vulnerability publicly disclosed.  
