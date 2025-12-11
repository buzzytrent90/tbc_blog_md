---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.905029'
original_url: https://thebuildingcoder.typepad.com/blog/0411_visual_studio_2010.html
post_number: '0411'
reading_time_minutes: 2
series: general
slug: visual_studio_2010
source_file: 0411_visual_studio_2010.htm
tags:
- csharp
- elements
- references
- revit-api
- vbnet
title: Visual Studio 2010 and the .NET Framework
word_count: 476
---

### Visual Studio 2010 and the .NET Framework

In section 1.6 'Supported Programming Languages', the developer guide "Revit 2011 API Developer Guide.pdf" in the Revit SDK clearly states that "The Revit Platform API is fully accessible by any language compatible with the Microsoft .NET Framework 3.5, such as Visual Basic .NET or Visual C#."

In section 1.3 'Requirements', it also mentions that the programming environment recommended for going through the examples is Visual Studio 2008.
That is also the environment used to create and compile all the SDK samples.

We already once mentioned the possibility of using and debugging with
[Visual Studio 2010](http://thebuildingcoder.typepad.com/blog/2010/04/debugging-with-visual-studio-2010-and-rvtsamples.html).
Questions on this have been coming up more frequently recently.
Here is one fundamental issue in connection with use of this version, with a clear and succinct answer by Joe Ye:

**Question:** I am having difficulty getting my VB Revit plug-in application to run with the Revit 2011 API when using Visual Studio 2010.
Just to keep things simple, I went back through the HelloWorld tutorial just in case I was missing something.
The HelloWorld add-in shows up correctly in the external tools drop down menu during debugging, but nothing happens when it is selected.
The code does not appear to be executing.

I added the references and namespaces, and set the 'copy local' flag to false for the Revit API assemblies.
Still no luck.

Maybe there is some problem with the add-in manifest file, but I cannot figure it out.
I also substituted a new GUID just in case there was a duplicate, but with no luck.

Here is the VB code that I am using to test with, in HelloWorld.vb:
```vbnet
Imports System
Imports Autodesk.Revit.UI
Imports Autodesk.Revit.DB

Public Class HelloWorld
    Implements IExternalCommand
    Public Function Execute( \_
      ByVal revit As ExternalCommandData, \_
      ByRef message As String, \_
      ByVal elements As ElementSet) \_
      As Autodesk.Revit.UI.Result \_
      Implements IExternalCommand.Execute

        TaskDialog.Show("Revit", "Hello World")
        Return Autodesk.Revit.UI.Result.Succeeded

    End Function

End Class
```

This is my add-in manifest file HelloWorld.addin:
```csharp
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<RevitAddIns>
  <AddIn Type="Command">
    <Assembly>C:\...\HelloWorld\bin\Debug\HelloWorld.dll</Assembly>
    <AddInId>8983a66d-3a78-4db8-b88b-e5c92bf04644</AddInId>
    <FullClassName>HelloWorld.HelloWorld</FullClassName>
    <Text>HelloWorld</Text>
  </AddIn>
</RevitAddIns>
```

What can be wrong, please?
Please help.

**Answer:** The recommended version of Visual Studio is 2008.
Revit 2011 does not support the .NET framework 4.0, which is used by default in Visual studio 2010 to compile the plug-in.

This might be causing an issue.
Please change the compilation settings to use the .NET framework 3.5 or an earlier version and see if that works.

**Response:** That solved the problem.
Thanks!