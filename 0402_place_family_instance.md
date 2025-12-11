---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: code_example
optimization_date: '2025-12-11T11:44:13.889756'
original_url: https://thebuildingcoder.typepad.com/blog/0402_place_family_instance.html
post_number: '0402'
reading_time_minutes: 5
series: elements
slug: place_family_instance
source_file: 0402_place_family_instance.htm
tags:
- doors
- elements
- family
- filtering
- levels
- parameters
- python
- revit-api
- transactions
- views
title: Place Family Instance
word_count: 927
---

### Place Family Instance

The Revit SDK does not include any example of calling the new PromptForFamilyInstancePlacement method, which prompts the user to interactively and graphically place instances of a specified family symbol, so I am creating a new Building Coder sample command to demonstrate its use.

At the same time, I can answer an interesting follow-on question:

**Question:** I am starting to use PromptForFamilyInstancePlacement to get the user to place one or more instances of a family symbol.
This method returns void.
It would be nice if it returned the instances that were placed instead.

Anyway, how can I find the new instances after this call?
Does the Revit API provide anything like the (entlast) idea in good old AutoLISP?

For example, I would like to call the PromptForFamilyInstancePlacement method and then assign some parameter values to the new family instances immediately afterwards.

**Answer:** I already discussed
[retrieval of newly added elements](http://thebuildingcoder.typepad.com/blog/2010/04/retrieving-newly-created-elements-in-revit-2011.html#4) to
the database.
It presents one approach making use of the element ids, which grow monotonously as new elements are added.
However, the
[comment by Guy Robinson](http://thebuildingcoder.typepad.com/blog/2010/04/retrieving-newly-created-elements-in-revit-2011.html#comments)
to that suggestion points out a much more reliable and officially supported method to obtain the newly added elements using the
[DocumentChanged event](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html#1)
and the newly added element id collection provided to its event handler.
Before we get to that, though, let's look at the call to PromptForFamilyInstancePlacement itself.

#### PromptForFamilyInstancePlacement

The Revit API help file
RevitAPI.chm makes the following remark about this method:

This method opens its own transaction, so it's not permitted to be invoked in an active transaction. In a single invocation, the user can place multiple instances of the input family type until they finish the placement (with Cancel or ESC or a click elsewhere in the UI). The user will not be permitted to change the type to be placed. Users are not permitted to change the active view during this placement operation (the operation will be completed).

In case you wonder, if you do try to run it in a command with automatic transaction mode, i.e. with a transaction opened automatically by Revit for the external command, the following exception is thrown:

- Autodesk.Revit.Exceptions.InvalidOperationException: "It's not permitted to run this API method in an active transaction."

In other words, if you wish to make use of this method, you are forced to use the new and improved manual transaction mode.

The method takes a family symbol argument to define the component being placed in the model, and there is no other input required or possible.

#### Retrieving Newly Created Elements

As suggested above, to retrieve the newly created elements added to the model by the call to PromptForFamilyInstancePlacement, one can simply subscribe to the DocumentChanged event.
The event handler is called with a DocumentChangedEventArgs instance which includes a method GetAddedElementIds to return all newly added element ids.

One thing to be aware of is that this method is called each time a placement is made, i.e. repeated calls are made for repeated placements within one single call to PromptForFamilyInstancePlacement.
In the sample presented below, each call will report one single element added.
If you want to collect all the element ids added, you need to append them to some global collection.

As always, every single event handler or other reactor added to a system will cause additional overhead, so it makes sense to minimise the use of event handlers.
In this case, this can be easily achieved by adding the handler immediately before prompting for the instance placement, and removing it immediately afterwards.

#### CmdPlaceFamilyInstance

I implemented a new Building Coder sample command CmdPlaceFamilyInstance to demonstrate the issues outlined above:

- Use manual transaction mode.- Define a variable \_added\_element\_ids to collect the newly added element ids in the DocumentChanged call-back method.- Use a filtered element collector to retrieve a symbol defining the type of family instances to be placed, in this case doors.- Subscribe to the DocumentChanged event immediately before prompting for placement.- Call PromptForFamilyInstancePlacement to interactively place one or more instances.- Unsubscribe from the DocumentChanged event.- Report the number of instances placed in the model.

The OnDocumentChanged event handler at the end is trivial, because it just adds all newly created element ids to our local global collection:
```python
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
class CmdPlaceFamilyInstance : IExternalCommand
{
  List<ElementId> \_added\_element\_ids = new List<ElementId>();

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    FilteredElementCollector collector
      = new FilteredElementCollector( doc );

    collector.OfCategory( BuiltInCategory.OST\_Doors );
    collector.OfClass( typeof( FamilySymbol ) );

    FamilySymbol symbol = collector.FirstElement() as FamilySymbol;

    \_added\_element\_ids.Clear();

    app.DocumentChanged
      += new EventHandler<DocumentChangedEventArgs>(
        OnDocumentChanged );

    uidoc.PromptForFamilyInstancePlacement( symbol );

    app.DocumentChanged
      -= new EventHandler<DocumentChangedEventArgs>(
        OnDocumentChanged );

    int n = \_added\_element\_ids.Count;

    TaskDialog.Show(
      "Place Family Instance",
      string.Format(
        "{0} element{1} added.", n,
        ( ( 1 == n ) ? "" : "s" ) ) );

    return Result.Succeeded;
  }

  void OnDocumentChanged( object sender, DocumentChangedEventArgs e )
  {
    \_added\_element\_ids.AddRange( e.GetAddedElementIds() );
  }
}
```

Note that it is always important to unsubscribe from events as soon as possible to minimise the overhead that all your callbacks will cause.

Here is
[version 2011.0.74.0](zip/bc_11_74.zip)
of The Building Coder samples including the complete source code and Visual Studio solution and the new command.