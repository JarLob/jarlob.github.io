---
layout:     post
title:      "Path Traversal in Aspose.ZIP for .NET"
date:       2019-05-13 12:00:00
categories: HackAndTell PathTraversal Aspose.ZIP
cover:      aspose_zip-for-net.png
permalink:  /en/blog/aspose-zip-path-traversal
---
I had been getting newsletters from [Aspose](https://www.aspose.com/) for quite some time. They didn't bother me too often and I liked scrolling the emails to see what new libraries they have implemented. Once, a new library named [Aspose.ZIP for .NET](https://products.aspose.com/zip/net) caught my eye. I wondered if it is resistant to an obvious attack vector - path traversal, that was on hype recently because one company tried to re-brand the vulnerability name :)

It took me few minutes to find a [script](https://github.com/ptoomey3/evilarc) to prepare a zip file with a payload. I've put `../../../../../../../../../../../../../../../temp/evil.txt` in a payload and wrote a sample program using the Aspose library:
```cs
using (FileStream zipFile = File.Open(@"evil.zip", FileMode.Open))
{
    using (var archive = new Archive(zipFile))
    {
        archive.ExtractToDirectory(@"C:\temp\dump");
    }
}
```
Quick test revealed it was indeed vulnerable - the `evil.txt` was extracted to `c:\temp\` directory one level up from `c:\temp\dump\` (`/` was conveniently accepted on Windows). The multiple `../` were used to get to the root directory on current disk, since the directive is ignored if already in the root folder, so the payload was independent from the extraction directory. I used `/temp/`, but an attacker could specify `/Windows/System32/` (it would need to be executed in administrator context) or any other directory. It was tested on Windows, but would work on Linux too with obvious adjustments to the payload. I've managed to contact developers on official support forum, but the story doesn't end here...

After more than month a new version of [Aspose.ZIP for .NET v18.11](https://docs.aspose.com/display/zipnet/Aspose.ZIP+for+.NET+18.11+Release+Notes) was released. I was not given any credits (oh well) and the older version was still available for download too. Since I wasn't notified I've noticed it only after a week. Testing the new version revealed that this time an attacker could put an absolute path in a payload to extract the file to any directory. Aspose was notified and after three days a new version [Aspose.ZIP for .NET v18.11.1](https://docs.aspose.com/display/zipnet/Aspose.ZIP+for+.NET+18.11.1+Release+Notes) was released.

Simple payloads didn't work with the release, so I decided to decompile and look into the sources. Although the library appeared to be obfuscated I have managed to reverse-engineer the sanitization function and found a flaw there: a part of the algorithm was to have a single pass over extraction path and remove `../` from it. Needless to say it is easily bypassable with `....//temp/evil.txt`. After sanitization it becomes `../temp/evil.txt`. The new bypass was sent to developers again.

This time after few weeks Aspose provided me a private build for testing before the release. It could have been simpler if they could just provide me a non-obfuscated version of the extraction function... Anyway, I found yet another bypass: ` \temp\evil.txt` (notice the leading whitespace) and reminded the developers that my recommendation in the very beginning was not to sanitize the path, but normalize/canonicalize it and verify if the result is under expected directory, like:
```cs
public void Extract(string dirPath, string fileName)
{
    dirPath = Path.GetFullPath(dirPath);
    if (!dirPath.EndsWith(Path.DirectorySeparatorChar.ToString(), StringComparison.Ordinal))
        dirPath += Path.DirectorySeparatorChar;

    var filePath = Path.GetFullPath(Path.Combine(dirPath, fileName));
    if (!filePath.StartsWith (dirPath, StringComparison.Ordinal))
        throw new Exception("Path traversal");
}
```
There is an important gotcha - if the `dirPath` is not checked to end with a directory separator it is possible to partially bypass the verification. Let's say the extraction path is `c:\temp\extr`. Without trailing separator enforcement it is possible to traverse into any directory in `c:\temp\` starting with `extr`, like `c:\temp\extrnot\` with a payload `..\extrnot\evil.txt`.

After two weeks I've got another release candidate to test. They still sanitized the path, but used some my suggestions, so I didn't find a way to bypass it. The vulnerability was finally fixed in [Aspose.ZIP for .NET v19.1](https://docs.aspose.com/display/zipnet/Aspose.ZIP+for+.NET+19.1+Release+Notes).

Timeline:  
2018.10.04 - Issue found and reported by email security(at)aspose.com without a reply.  
2018.10.10 - Successfully reported in a private Aspose forum conversation.  
2018.11.21 - Aspose.ZIP for .NET v18.11 is released.  
2018.11.26 - I finally notice a new version available, provide a different payload that is not mitigated.  
2018.11.29 - Version 18.11.1 is released. I provide another bypass.  
2018.12.14 - Aspose provides a release candidate for verification.  
2018.12.17 - Another bypass is found.  
2018.12.31 - New release candidate is provided. No bypasses were found.  
2019.01.03 - Version 19.1.0 is released.  
2019.01.04 - Post in Full Disclosure mailing group.  