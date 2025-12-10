---
post_number: "0535"
title: "Set Elbow Fitting Type"
slug: "set_elbow_fitting_type"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'revit-api']
source_file: "0535_set_elbow_fitting_type.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0535_set_elbow_fitting_type.html"
---

### Set Elbow Fitting Type

Here is a simple question with a simple solution on setting the type of a family instance:

**Question:** I read your post on
[cable tray orientation and fittings](http://thebuildingcoder.typepad.com/blog/2010/05/cable-tray-orientation-and-fittings.html) and
also looked at the
[AutoRoute](http://thebuildingcoder.typepad.com/blog/2009/09/the-revit-mep-api.html#6) SDK sample.
I am using NewElbowFitting too.

The problem is that the fittings created automatically are of the family 'Rectangle Elbow - Mitered: Standard'.
If I want to create instances of another family, say 'Rectangle Elbow - Radius: 1W', how could I achieve that?

**Answer:** How about creating them first, in whatever way you can, and then accessing the new family instance and setting its type as you like after it has been created.
Does that work?

**Response:** Thanks for your suggestion.
It works!

I implemented the idea as follows:

1. Find the target type element id using the following code:
```csharp
  ICollection<ElementId> iElementIds
    = fi.GetValidTypes();

  if( iElementIds != null && iElementIds.Count > 0 )
  {
    string typeInfo = "";
    foreach( ElementId eid in iElementIds )
    {
      Element elem = doc.get\_Element( eid );
      if( elem != null )
      {
        FamilySymbol fs = elem as FamilySymbol;
        typeInfo += fs.Family.Name + "  "
          + elem.Name + "  "
          + eid.IntegerValue.ToString() + "\n";
      }
    }
    TaskDialog.Show( "revit", typeInfo );
  }
```

2. Use the ChangeTypeId method to change the FamilyInstance type:
```csharp
  fi.ChangeTypeId( new ElementId( 133158 ) );
```