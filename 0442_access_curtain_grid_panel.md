---
post_number: "0442"
title: "Access to Curtain Grid Panels"
slug: "access_curtain_grid_panel"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'python', 'references', 'revit-api', 'selection', 'transactions', 'walls']
source_file: "0442_access_curtain_grid_panel.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0442_access_curtain_grid_panel.html"
---

### Access to Curtain Grid Panels

Here is some strange behaviour noted by Daren Thomas of the
[Professur für Gebäudetechnik, Institut für Hochbautechnik](http://www.gt.arch.ethz.ch) at
the technical university ETH Zürich and author of the
[Revit Python shell](http://thebuildingcoder.typepad.com/blog/2009/12/revit-python-shell.html) which
led to the discovery and quick fix of a problem in the curtain grid API.
Since Daren also provides a workaround for the problem for immediate use, I think it worthwhile to note it here.

**Question:** I just came by some strange behaviour in Autodesk Revit Architecture 2011: It seems that reading a Wall.CurtainGrid.Panels requires a transaction.

Here is a repro (using RevitPythonShell to avoid having to create a new project etc.):

- Create a new Revit project.- Add a wall.- Change the wall type to CurtainGrid.- Remember the element ID of wall (Manage > Inquiry > IDs of Selection), in my case 127699.- Go to Add-Ins > Open Python Shell.- Type following sequence of commands (this can be translated to C# pretty easily, as it is mainly about the API); copy to a text editor to see the truncated lines:

```python
>>>import clr
>>>clr.AddReference('RevitAPI')
>>>from Autodesk.Revit.DB import \*
>>>doc = \_\_revit\_\_.ActiveUIDocument.Document
>>>cw = doc.get\_Element(ElementId(127699)) # substitute actual ID here
>>>cw
<Autodesk.Revit.DB.Wall object at
0x000000000000002B [Autodesk.Revit.DB.Wall]>
>>>cw.CurtainGrid
<Autodesk.Revit.DB.CurtainGrid object at
0x000000000000002C [Autodesk.Revit.DB.CurtainGrid]>
>>>cw.CurtainGrid.Panels
IronTextBoxControl error: Attempt to modify the model outside of transaction.
>>>
```

By the way, this little script also amply demonstrates how extremely easy and powerfully a Revit model can be accessed programmatically and interactively through the Python shell.

This dialog with RevitPythonShell shows that accessing the wall and its CurtainGrid property is ok, but accessing the Panels property triggers the exception.
The RevitPythonShell runs with TransactionMode.Manual.
This behaviour is not restricted to RevitPythonShell scripts – this is only for the repro.
I came across this behaviour with a C# plug-in that also uses TransactionMode.Manual.

What is the rationale behind this behaviour?

**Answer:** Several of the CurtainGrid APIs were ensuring that they had write access to the underlying objects, even when only read access was required.

This behaviour has now been reported and corrected.

**Response:** I'm not sure how fixes are deployed with Revit, but assume they are just included in the annual release – so for your readers, you might want to mention a workaround:

Starting a transaction and then calling the method Transaction.Rollback for tasks that don't need transactions but do read curtain wall panels.

Ideally, you want to place the transaction code as close to the curtain wall code as possible, but not inside an inner loop as that can slow things down considerably (been there, done that, getting t-shirts printed to prove).

**Answer:** Yes, I would love to mention this in the blog and provide a workaround for those who run into this issue.

Do you happen to have some sample code ready to demonstrate exactly what you mean?

Or even (miraculously?) a reproducible sample project with one external command showing the problem and another one showing the solution?

**Response:** I went through the trouble to build such a project.

You will find attached a C# solution (Visual Studio 2010) that creates the two external commands requested.
Inside the solution is also a sample Revit project file SingleCurtainWall.rvt containing four walls, one of which is a CurtainWall with one panel.
A sample add-in file can also be found inside the solution.

Also, here are some screenshots of the output of said external commands; first the exception being thrown when the transaction workaround is not implemented:

![CurtainGrid Panels transaction bug](img/CurtainGridPanelsTransactionBug.png)

With the workaround in place, the panels become accessible:

![CurtainGrid Panels transaction workaround](img/CurtainGridPanelsWorkAround.png)

For completeness sake, here is the source code of the initial command causing the problem for comparison with the workaround listed below:
```csharp
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
public class TriggerCurtainGridPanelsTransactionBug
  : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    var doc = commandData.Application
      .ActiveUIDocument.Document;

    var walls = new FilteredElementCollector( doc )
      .OfClass( typeof( Wall ) )
      .Cast<Wall>();

    foreach( var wall in walls )
    {
      if( wall.WallType.Kind == WallKind.Curtain )
      {
        TaskDialog.Show(
          "CurtainGridPanelsTransactionBug",
          string.Format(
            "CurtainWall has {0} panels.",
            wall.CurtainGrid.Panels.Size ) );
      }
    }
    return Result.Succeeded;
  }
}
```

This is the updated code with the added transaction required to access the curtain grid panel data, which can be discarded after use:
```csharp
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
public class WorkAroundCurtainGridPanelsTransactionBug
  : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    var doc = commandData.Application
      .ActiveUIDocument.Document;

    var walls = new FilteredElementCollector( doc )
      .OfClass( typeof( Wall ) )
      .Cast<Wall>();

    // workaround: create a transaction

    var transaction = new Transaction( doc,
      "CurtainGridPanelsTransactionBug" );

    transaction.Start();

    try
    {
      foreach( var wall in walls )
      {
        if( wall.WallType.Kind == WallKind.Curtain )
        {
          TaskDialog.Show(
            "CurtainGridPanelsTransactionBug",
            string.Format(
              "CurtainWall has {0} panels.",
              wall.CurtainGrid.Panels.Size ) );
        }
      }
    }
    finally
    {
      // we don't really want to change the document!

      transaction.RollBack();
    }
    return Result.Succeeded;
  }
}
```

For yet more completeness sake, here is
[CurtainGridPanelsTransaction.zip](zip/CurtainGridPanelsTransaction.zip) containing
the entire source code, Visual Studio 2008 and 2010 solution, and a sample file SingleCurtainWall.rvt to test it on.

I guess I got carried away on this task...

Many thanks to Daren for all his research, workaround, and complete documentation of this issue!