---
post_number: "0678"
title: "Read-only Workset API"
slug: "workset_api"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'revit-api']
source_file: "0678_workset_api.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0678_workset_api.html"
---

### Read-only Workset API

Another one of the new
[Revit 2012 API features](http://thebuildingcoder.typepad.com/blog/2011/03/revit-2012-api-features.html) that
we still never had a detailed look at is the worksharing API, a comprehensive read-only API to worksharing features, access the list of worksets, find out what elements are in a workset, who borrowed what, whether an element went out of date with central, etc.
Here is a quick Q & A on some basic aspects of that API:

**Question:** I know that the built-in parameter ELEM\_PARTITION\_PARAM identifies the workset of an element in some manner.
Its value is an integer.
How can I retrieve the workset name from this value?

**Answer:** The integer value stored in the parameter is actually the workset id, which is used as a key in the workset table.

To access the workset, you therefore first have to
obtain the document workset table, for instance via the GetWorksetTable method:
```csharp
  WorksetTable worksetTable
    = doc.GetWorksetTable();
```

You can then create a workset id from the parameter value and access the workset of a given element 'e' using that:
```csharp
  Parameter p = e.get\_Parameter(
    BuiltInParameter.ELEM\_PARTITION\_PARAM );

  int paramValue = param.AsInteger();

  WorksetId wid = new WorksetId( paramValue );

  Workset workset = WorksetId.InvalidWorksetId == wid
    ? null
    : worksetTable.GetWorkset( wid );
```

Just like a standard Revit database element, the workset provides a property returning its name.

There is also a simpler direct way to access the workset of an element, however, using the document method GetWorksetId, which returns the workset id for a given element id:
```csharp
  WorksetId wid = doc.GetWorksetId( e.Id );
```

**Question:** How can I iterate the workset table to obtain a list of all the worksets defined in the project?

**Answer:** There are dedicated FilteredWorksetCollector and WorksetFilter classes for that purpose.
Here is some sample code demonstrating their use:
```csharp
  FilteredWorksetCollector coll
    = new FilteredWorksetCollector( doc );

  // You may want to filter them...

  //coll.OfKind(WorksetKind.UserWorkset);

  StringBuilder worksetNames = new StringBuilder();

  foreach( Workset workset in coll )
  {
    worksetNames.AppendFormat( "{0}: {1}\n",
      workset.Name, workset.Kind );
  }

  TaskDialog.Show( "Worksets",
    worksetNames.ToString() );
```

As mentioned in the comment, please note that the FilteredWorksetCollector class provides an OfKind method which can be used to limit the filters returned to a given workset kind, e.g. to UserWorkset only, i.e. worksets defined by users, including the two default Revit ones.