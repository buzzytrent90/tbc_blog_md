---
post_number: "0028"
title: "Converting between VB and C#, and .NET Decompilation"
slug: "reflector"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'vbnet']
source_file: "0028_reflector.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0028_reflector.html"
---

### Converting between VB and C#, and .NET Decompilation

Here is post to follow up on the
[comment](http://thebuildingcoder.typepad.com/blog/2008/10/application-events-in-vb.html#comments)
on converting between C# and VB code by
[Rod Howarth](http://roddotnet.blogspot.com)
on the post on
[Application Events in VB](http://thebuildingcoder.typepad.com/blog/2008/10/application-events-in-vb.html).

First of all, converting code between C# and VB is possible and can be done automatically. C# and VB.NET both produce the identical intermediate language IL code, which is what is actually stored in the .NET assembly and evaluated by the .NET framework. The IL code contains all information required to regenerate the source code. So, on one hand, you can translate between C# and VB. Furthermore, you can even recreate source code in any language of your choice by decompiling the IL code in the assembly.

To translate between C# and VB, just google for "c# vb translator". Rod mentioned a couple of such translators; he says:

> Also useful for anyone who is a VB coder and wants to convert samples are the many VB to C# code converters available. Using a Google query such as
> [this one](http://www.google.com.au/search?source=ig&hl=en&rlz=1G1GGLQ_ENAU244&=&q=C%23+to+VB+code+converter&btnG=Google+Search&meta=)
> will turn up tools such as
> <http://www.developerfusion.com/tools/convert/csharp-to-vb>
> and
> <http://www.carlosag.net/Tools/CodeTranslator>.
> They may not do whole projects but they can help you decipher the samples!

To analyse or decompile any .NET assembly, even without access to the source code, a tool that I use is
[Reflector](http://www.aisto.com/roeder/dotnet)
by
[Lutz Roeder](http://www.lutzroeder.com).

It includes reverse engineering functionality, for instance a decompilation and source code generation feature, so it can read your compiled DLL and decompile the IL intermediate language it contains into various languages including C#, VB, managed C++, and others. It can obviously also be used to list all namespaces and classes defined in an assembly, and their methods and properties.

Here is an example of decompiling a Revit external command assembly I built using C# and requesting VB code for it:

![](C:/a/j/adn/case/bsd/1242691/attach/_reflector.png)

See the VB code? I wrote this application in C#! My version of Reflector will also decompile into Delphi, Chrome and managed C++, besides C# and VB.

Although ... please note that even Reflector is not always 100% correct ... if you look carefully at the screen snapshot, note that it seems to have come up with something strange; in the line following

```
If (Not Nothing Is PlanarFace) Then
```

it says

```
Dim <>8__locals2 As <>c__DisplayClass1
```

That will probably not compile correctly. Thanks to Adam Nagy, a DevTech colleague of mine, for noticing this error.

I find this an invaluable tool, and often use it to explore my own .NET assemblies, just to ensure that they really do contain what I am expecting them to.