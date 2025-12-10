---
post_number: "0448"
title: "Extract Part Atom Revisited"
slug: "extract_part_atom"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'python', 'revit-api', 'transactions', 'windows']
source_file: "0448_extract_part_atom.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0448_extract_part_atom.html"
---

### Extract Part Atom Revisited

We briefly discussed the use of the
[ExtractPartAtomFromFamilyFile](http://thebuildingcoder.typepad.com/blog/2009/11/extract-part-atoms.html) method in Revit 2010.
Now the question on using this method arose again in the context of the Revit 2011 API:

**Question:** I invoked the Autodesk.Revit.ApplicationServices.Application ExtractPartAtomFromFamilyFile method of on an RFA file, using the following code:
```csharp
private void createPartAtomFile(
  Application app,
  string rfaFilePath,
  string partAtomFilePath )
{
  app.ExtractPartAtomFromFamilyFile(
    rfaFilePath,
    partAtomFilePath );
}
```
This threw an AccessViolationException with the following stack trace:

```
AccessViolationException: Attempted to read or write protected memory. This is often an indication that other memory is corrupt.
at extractPartAtomFromFamilyFile(AString* , AString* )
at Autodesk.Revit.ApplicationServices.Application.ExtractPartAtomFromFamilyFile(String familyFilePath, String xmlFilePath)
...
...
```

How can I avoid this exception and successfully generate the part atom XML for the RFA file?

**Answer:** Based on an answer to this question by my colleague Saikat Bhattacharya, I wrote a new Building Coder sample command CmdPartAtom to test run this method.
The command implementation is achieved in the following few lines of code and successfully generates the part atom XML file:
```python
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
class CmdPartAtom : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    Transaction trans = new Transaction( doc,
      "Extract Part Atom" );

    trans.Start();

    string familyFilePath
      = "C:/Documents and Settings/All Users"
      + "/Application Data/Autodesk/RAC 2011"
      + "/Metric Library/Doors/M\_Double-Flush.rfa";

    string xmlPath = "C:/tmp/ExtractPartAtom.xml";

    app.ExtractPartAtomFromFamilyFile(
      familyFilePath, xmlPath );

    trans.Commit();

    return Result.Succeeded;
  }
}
```

Here is a screen snapshot of the resulting XML file displayed in the browser, with all except one of the A:part nodes collapsed:

![ExtractPartAtom XML output](img/ExtractPartAtomXml.png)

In Saikat's words:
Looking at the exception you have received, I am not sure if it is related to the TransactionMode and RegenerationOption attributes used in your plug-in.
In my case, I used the Manual mode for both the attributes.
Please try using my code snippet and steps to see if it works well at your end too.

Many thanks to Saikat for this exploration!

Here is
[version 2011.0.75.0](zip/bc_11_75.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.

More recently, we have heard that there is currently a problem with this API call on 64 bit systems, so right now this will only work in the 32 bit version.
As Steve points out below, the issue might be occurring only on certain platforms, e.g. on 64 bit Windows 7 OS running 64-bit Revit Architecture 2011.
It has been resolved internally, though, so the next update should have the issue fixed.