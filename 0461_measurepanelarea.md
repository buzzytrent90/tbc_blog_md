---
post_number: "0461"
title: "MeasurePanelArea Update"
slug: "measurepanelarea"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'revit-api', 'rooms', 'views', 'walls']
source_file: "0461_measurepanelarea.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0461_measurepanelarea.html"
---

### MeasurePanelArea Update

Iffat Mai of
[Perkins + Will](http://www.perkinswill.com) brought
up an issue with the MeasurePanelArea Revit SDK sample, handled by Joe Ye:

**Question:** As we reviewed the Massing > MeasurePanelArea SDK Sample, the following error message was displayed while trying to run it:

![MeasurePanelArea error message](img/MeasurePanelArea_error.jpg)

It says: "Input type is of an element type that exists in the API, but not in Revit's native object model.
Try using Autodesk.Revit.DB.FamilyInstance instead, and then post-processing the results to find the elements of interest."

**Answer:** This is due to the fact that the Panel class cannot be accepted by the FilteredElementCollector OfClass method, i.e. when calling
```csharp
  FilteredElementCollector
    .OfClass( typeof( Panel ) );
```

This call is made in the GetElements template method:
```csharp
  protected List<T>
    GetElements<T>() where T : Element
```

GetElements in turn is called by the method BuildPanelTypeList:
```csharp
  private void BuildPanelTypeList()
  {
    List<Panel> list = GetElements<Panel>();
    . . .
  }
```

We already discussed this issue of
[filtering for a non-native class](http://thebuildingcoder.typepad.com/blog/2010/08/filtering-for-a-nonnative-class.html) in
some depth.
It applies to AnnotationSymbol, Panel, Room and several other classes.

I changed the sample slightly to add another non-template method to get all panels in current document instead:
```csharp
protected IList<Element> GetPanels()
{
  FilteredElementCollector collector
    = new FilteredElementCollector(
      m\_uiDoc.Document );

  BuiltInCategory bic
    = BuiltInCategory.OST\_CurtainWallPanels;

  collector.OfClass( typeof( FamilyInstance ) )
    .OfCategory( bic );

  IList<Element> panels = collector.ToElements();

  return panels;
}
```

Here is my modified version of
[FormPanelArea.cs](zip/FormPanelArea.cs) with these changes.
Please replace the existing version of the file with it.

Many thanks to Joe for this fix!

The newly released
[updated SDK](http://thebuildingcoder.typepad.com/blog/2010/10/subscription-release-and-updated-sdk.html) for the
[web update 2](http://thebuildingcoder.typepad.com/blog/2010/09/revit-2011-web-update-2.html) and
[subscription releases](http://thebuildingcoder.typepad.com/blog/2010/10/subscription-release-and-updated-sdk.html) includes
several other
[SDK sample updates](http://thebuildingcoder.typepad.com/blog/2010/10/subscription-release-and-updated-sdk.html#7).
Unfortunately the MeasurePanelArea fix was not made in time to be included in the SDK update.
We also recently discussed a fix to the
[DoorSwing SDK sample](http://thebuildingcoder.typepad.com/blog/2010/09/doorswing-fix.html) which
does not seem to have been included either.