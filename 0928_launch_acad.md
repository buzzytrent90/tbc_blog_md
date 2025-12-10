---
post_number: "0928"
title: "Launching AutoCAD within a Revit Add-in"
slug: "launch_acad"
author: "Jeremy Tammik"
tags: ['elements', 'python', 'references', 'revit-api', 'transactions', 'vbnet', 'views', 'windows']
source_file: "0928_launch_acad.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0928_launch_acad.html"
---

### Launching AutoCAD within a Revit Add-in

We looked at the topic of
[running AutoCAD inside a Revit add-in](http://thebuildingcoder.typepad.com/blog/2008/11/running-autocad-inside-revit.html) way
back in 2008, in the grey and dusty beginnings of this blog.

Many developers have made use of that since then, and the topic is still important for all those who have significant functionality running inside an AutoCAD application and wish to make use of or enhance it by connecting it with their Revit add-in.

Therefore, it is about time to dig that out and have a fresh look at it.
Actually, it is rather overdue.
I have some more up-to-date information on this that has been sitting around on my to-do list for quite a while.
In fact, it is high time to get it out there before it becomes obsolete, and should still prove useful to anyone interested in this topic.

Actually, here are two separate solutions,
[testole.zip](#2) and
[Acctrl\_Interops.zip + EmbedAutoCAD.zip](#3).

#### Testole.zip

I will simply provide the sample code as it is with no further comment:
```python
// This could have been read-only, but then it
// cannot be started in zero document state:

[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    Autodesk.AutoCAD.Interop.AcadApplication a
      = new Autodesk.AutoCAD.Interop.AcadApplication();

    a.Visible = true;

    Autodesk.AutoCAD.Interop.AcadDocument doc
      = a.Documents.Application.ActiveDocument;

    double[] stpoint = new double[3];
    double[] enpoint = new double[3];
    stpoint[0] = 5;
    stpoint[1] = 5;
    stpoint[2] = 0;
    enpoint[0] = 12;
    enpoint[1] = 3;
    enpoint[2] = 0;

    doc.ModelSpace.AddLine( stpoint, enpoint );

    doc.SaveAs( "C:/tmp/testline.dwg" );

    a.Quit();

    return Result.Succeeded;
  }
}
```

That shows how simple this can be.

Here is [testole.zip](zip/launch_acad_testole.zip) containing the complete Visual Studio solution.

This is up to you to explore further on your own, if you like :-)

#### 2. Acctrl\_Interops.zip + EmbedAutoCAD\_2012.zip

The second solution is not directly related to the Revit API, but should work in an add-in as well.
It was prompted by the following query:

**Question:** How can I view and manipulate DWG files in a Windows form?

I want to create an AutoCAD add-in that shows a preview of another DWG file (not the current one) in a VB.NET form and then switch some layers on and off.

I am using Visual Studio 2010.

**Answer:** You can use the AutoCAD control AcCtrl for this.

Kean's article on
[embedding AutoCAD in a standalone dialog](http://through-the-interface.typepad.com/through_the_interface/2008/03/embedding-autoc.html) explains
everything in full detail.

**Response:** This control does not seem to work with VS 2010 on the .NET framework 4.

**Answer:** I have attached the sample project that uses the Acctrl.dll from AutoCAD 2012 and Visual Studio 2010 with the .NET 4 framework.

The code is identical to the one that you download from the blog and works fine.

Here are the only things I changed:

- I used aximp to generate the interop DLLs from Acctrl.dll and attached them below in Acctrl\_Interops.zip.
  These interop DLLs need to be referenced in the project.
- Set the 'Embed Interop types' toggle to False in the properties of these references.

Here are the two attachments,
[Acctrl\_Interops.zip](zip/Acctrl_Interops.zip) containing the interop DLLs I generated and
[EmbedAutoCAD.zip](zip/EmbedAutoCAD.zip) containing the complete Visual Studio solution using them.

I hope you find this useful!