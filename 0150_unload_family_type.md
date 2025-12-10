---
post_number: "0150"
title: "Unload Family Type"
slug: "unload_family_type"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'revit-api']
source_file: "0150_unload_family_type.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0150_unload_family_type.html"
---

### Unload Family Type

We discussed the
[loading of a single family type](http://thebuildingcoder.typepad.com/blog/2009/06/addin-ribbon-panel-and-loading-one-single-type.html)
just two days ago.
Here is a related question that came up and was handled by Saikat Bhattacharya:

**Question:**
How can I use the API to remove unused family types from a project?
I know there is a command within Revit to do this, but we would like to automate the process.

**Answer:**
You can unload a specific loaded family symbol from your project using the Document.Delete() method.
I tested this as follows:

- Load a specific column symbol "457 x 610mm".- Run an external command with the code below to filter out and iterate over all column elements, including symbols.- Apply the Delete method to remove and thus unload the specific symbol.

After running the code, the list of loaded family symbols displayed in the user interface confirmed that the specific family symbol was no longer present in the project.
Here is the code for the external command Execute method:

```csharp
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  Filter filter = app.Create.Filter.NewCategoryFilter(
    BuiltInCategory.OST\_Columns );

  ElementIterator it = doc.get\_Elements( filter );

  while ( it.MoveNext() )
  {
    Element e = it.Current as Element;
    if ( e.Name.Equals( "457 x 610mm" ) )
    {
      doc.Delete( e );
    }
  }
  return IExternalCommand.Result.Succeeded;
}
```

Thank you Saikat for this solution!