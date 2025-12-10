---
post_number: "0625"
title: "PickObject Requires Valid View"
slug: "pickobject_view"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'references', 'revit-api', 'selection', 'views']
source_file: "0625_pickobject_view.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0625_pickobject_view.html"
---

### PickObject Requires Valid View

The Revit 2011 API introduced the PickObject and PickObjects methods, and we gave a simple sample of using the latter for
[interactive filtered element selection](http://thebuildingcoder.typepad.com/blog/2010/05/pipe-to-conduit-converter.html#8) in the
[pipe to conduit converter](http://thebuildingcoder.typepad.com/blog/2010/05/pipe-to-conduit-converter.html).

One thing to be aware of is that these methods throw an exception if the user cancels the selection, so the add-in needs to implement a handler for that.

They can also throw an exception of called in an invalid view, for example in the Revit MEP system browser:

**Question:** If I launch an external command calling PickObject after opening the system browser and selecting an MEP system, Revit throws a System.Runtime.InteropServices.SEHException.

How can I avoid this from happening?

**Answer:** Here are two simple and self-explanatory workarounds for this:

1. Always check the active view before calling PickObject, and avoid calling it at all if the current view type is ViewType.Internal.

In this case, the active view is 'System Browse', which is an internal view:
```csharp
  if( ViewType.Internal == doc.ActiveView.ViewType )
  {
    TaskDialog.Show( "Error",
      "Cannot pick element in this view: "
      + doc.ActiveView.Name );

    return null;
  }
```

2. Activate a valid view before the PickObject action.
You will obviously need to know which is the currently valid 'active' view.
```csharp
  uidoc.ActiveView = view; // non internal view

  Reference selectedRef = uidoc.Selection
    .PickObject( ObjectType.Element );
```

The first workaround is very easy to implement.
The second one needs more effort to keep track of the currently valid 'active' view.
The ViewActivating and ViewActivated events might help you implement something to achieve this.

**Reponse:** Thank you for these tips.
I ended up using alternative no. 2 and it seems to be working fine.