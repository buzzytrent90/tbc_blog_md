---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:13.593450'
original_url: https://thebuildingcoder.typepad.com/blog/0232_namespaces.html
post_number: '0232'
reading_time_minutes: 2
series: general
slug: namespaces
source_file: 0232_namespaces.htm
tags:
- csharp
- elements
- levels
- references
- revit-api
title: Namespaces
word_count: 475
---

### Namespaces

Here is a
[question](http://thebuildingcoder.typepad.com/blog/2009/04/getting-started-with-the-revit-2009-api.html?cid=6a00e553e1689788330120a5e41815970b#comment-6a00e553e1689788330120a5e41815970b) by
[Vince](mailto:freelancecadd@gmail.com) that
many beginners may encounter, on namespaces and references and setting up a project.
Although it may seem confusing the first time through, it is very simple to handle once you know how.

**Question:** Can you point me to material, blog or forum that discusses how to insert Code Regions from the SDK material into existing code?

I'm learning to code so any basic level information that you can provide me would be a big help.

I figured out how to do the Hello World tutorial located at the beginning of the Revit 2010 API user Manual, but now I would like to expand on the information by utilizing the additional code regions supplied in the User Manual but I continue to get errors.

I'm sure it's basic in nature but because of my limited experience difficulty is around every corner.

Again, if you could supply me with a simple tutorial showing how to insert a code region into existing code would be very helpful.

**Answer:** I cannot really say anything special about copying source code snippets from one project to another. I just use copy and paste in any old editor, actually. In .NET, you just need to ensure that all required references and using statements are in place. Maybe that is causing the errors you see.

The 'using' statements at the head of each module specify namespaces that can be used without explicitly specifying the namespace each time, so that you can write 'Element' instead of the full class name 'Autodesk.Revit.Element':
```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit;
using Autodesk.Revit.Elements;
```

The references provide the definitions of the namespaces, and are actually .NET assemblies, i.e. DLL files that need to exist on you system and on the system executing your plug-in. They are loaded by the .NET framework when your plug-in is loaded:
![References](img/references.png)

Adding the required references to a project is demonstrated by the
[Revit DevTV recording](http://thebuildingcoder.typepad.com/blog/2009/04/getting-started-with-the-revit-2009-api.html).

The entire Revit API resides in one single assembly, RevitAPI.dll, which includes all the Revit namespaces, so that is simple. Which class resides in which namespace is documented by the Revit API help file RevitAPI.chm, which is included with the Revit SDK:
![Namespaces](img/namespaces.png)

In general, all you have to do when you copy code from one project to another is ensure that the required references are loaded, and then add appropriate using statements, unless the code you copied uses the full name of every class it references.