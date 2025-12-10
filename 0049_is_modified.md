---
post_number: "0049"
title: "Document IsModified Property"
slug: "is_modified"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'transactions']
source_file: "0049_is_modified.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0049_is_modified.html"
---

### Document IsModified Property

Looking at the Document.IsModified property, one quickly notices that it is frequently set, even if no real modifications have been applied to a document. One reason for this is that every execution of an external command returning IExternalCommand.Result.Succeeded will cause this property to return True. To explore this in detail, we can use the following minimal command implementation:

```csharp
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;
  Debug.WriteLine( string.Format(
    "Document.IsModified {0}", doc.IsModified ) );
  return IExternalCommand.Result.Succeeded;
}
```

The result of running this command twice in a newly opened document is:

```
Document.IsModified False
Document.IsModified True
```

According to the Revit API help, the Document.IsModified property reports whether the document has been modified since it was last saved.
Since the command returned Succeeded, Revit assumes that the execution did modify it, and the result is as expected.
If you do not want this property to be modified, for instance by a simple external command displaying a Help > About... dialogue which has no interaction at all with the Revit model, it should return Cancelled or Failed.
To test this, I modified my little test command as follows:

```csharp
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;
  Debug.WriteLine( string.Format(
    "Document.IsModified {0}", doc.IsModified ) );

  IExternalCommand.Result rc;
  rc = IExternalCommand.Result.Failed;
  return rc;
}
```

The result of running the modified code twice in a newly opened document is:

```
Document.IsModified False
Document.IsModified False
```

Many thanks to
[Matt Mason](http://cadappdev.blogspot.com)
of
[Avatech Solutions](http://www.avatech.com)
for raising this issue!

Matt performed some additional analysis and suggests noting the following implications of the results above:

- Returning Succeeded commits a transaction to the Revit model, even if that transaction is empty.
- If it is important to the add-in to not "modify" the model inadvertently, you need to keep track of whether you have made modifications, return Succeeded if you did, and Cancelled or Failed if you did not.

Most sample applications always return Succeeded.
Matt also discovered that simply opening a transaction will immediately set this flag.
Because
[RvtMgdDbg](http://download.autodesk.com/media/adn/RvtMgdDbg2009_0429_2008.zip)
immediately opens a transaction, the flag is always set when debugging into its code.