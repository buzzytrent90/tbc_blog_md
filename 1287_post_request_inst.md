---
post_number: "1287"
title: "PostRequestForElementTypePlacement Sample"
slug: "post_request_inst"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'python', 'revit-api', 'transactions', 'vbnet', 'views', 'walls']
source_file: "1287_post_request_inst.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1287_post_request_inst.html"
---

### PostRequestForElementTypePlacement Sample

I am back from my vacation in the snow, full of new energy, and up to both ears in hot Revit API cases again instead of that frozen stuff :-)

The creation document NewFamilyInstance method provides the traditional way to programmatically create a new family instance within a project, or a nested instance within a family document.

It does not support any user interaction whatsoever.

A little bit of user interaction is provided by the PromptForFamilyInstancePlacement method introduced in the
[Revit 2011 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2011-api.html).

At least it shows a preview of the item to be placed and prompts the user for its location.

Numerous samples using the PromptForFamilyInstancePlacement method are available, most recently here, including sample code to execute it and
[terminate its family instance placement loop](http://thebuildingcoder.typepad.com/blog/2015/02/sending-escape-to-terminate-a-family-instance-placement-loop.html),
and in the Revit API discussion forum, e.g.:

- [PromptForFamilyInstancePlacement spacebar to rotate](http://forums.autodesk.com/t5/Revit-API/promptforfamilyinstanceplacement-spacebar-to-rotate-causes/m-p/4794701)
- [PromptForFamilyInstancePlacement in an enclosed space](http://forums.autodesk.com/t5/revit-api/promptforf-amilyinsta-nceplaceme-nt-to-allow-adding-in-an/m-p/5439737)

The PromptForFamilyInstancePlacement method does not offer all the possible user interaction functionality provided by the manual Revit user interface either, though.

The Revit 2015 API introduced the PostRequestForElementTypePlacement method to access the full Revit user interaction functionality, at the cost of giving up full programmatic control and passing the baton from the add-in code back to Revit, cf.
[What's New in the Revit 2015 API](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html) >
[Minor API additions](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#4) >
[Document API additions](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#4.02).

Another interesting Revit API forum discussion on
[changing the placement option for PromptForFamilyInstancePlacement](http://forums.autodesk.com/t5/Revit-API/Change-the-placement-option-for-PromptForFamilyInstancePlacement/m-p/3187174) also
includes a snippet of sample code in VB.NET by PhillipM for using PostRequestForElementTypePlacement.

Its usage is extremely simple, since the add-in is asking Revit to do all the work and passing over control as well.

All you need to do is supply an element type for the user to place:

```python
[Transaction( TransactionMode.Manual )]
class CmdPostRequestInstancePlacement : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    ElementType elementType
      = new FilteredElementCollector( doc )
        .OfCategory( BuiltInCategory.OST\_Walls )
        .OfClass( typeof( ElementType ) )
        .FirstElement() as ElementType;

    uidoc.PostRequestForElementTypePlacement(
      elementType );

    return Result.Succeeded;
  }
}
```

There is no need to encapsulate the call in a transaction.

In spite of this, posting the request cannot be triggered in a read-only command, or an invalid operation exception will be triggered saying:

> Revit encountered a Autodesk.Revit.Exceptions.InvalidOperationException: Starting a new transaction is not permitted. It could be because another transaction already started and has not been completed yet, or the document is in a state in which it cannot start a new transaction.

![PostRequestForElementTypePlacement requires a transaction](img/post_request_requires_transaction.png)

I implemented this code in a new external command named CmdPostRequestInstancePlacement in
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[version 2015.0.119.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.119.0).

#### The Fear of Fear

Before signing off for today, here are some sweet and wise thoughts by Mr Ramesh on
[fear and aliveness](https://www.youtube.com/watch?v=CDlILJoYA3E):

Mr Ramesh' invitation to become curious about experiencing your fears is so attractive and sensible that it could revolutionize the world! ... Becoming curious towards our fear...