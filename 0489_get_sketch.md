---
post_number: "0489"
title: "Access to Sketch and Sketch Plane"
slug: "get_sketch"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'levels', 'parameters', 'python', 'references', 'revit-api', 'schedules', 'selection', 'transactions', 'walls']
source_file: "0489_get_sketch.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0489_get_sketch.html"
---

### Access to Sketch and Sketch Plane

I just presented my first Revit API class here at AU 2010,
[CP228-2](http://au.autodesk.com/?nd=event_class&session_id=6943&jid=614479), on the optimal use of the Revit 2011 programming features and the Idling event.
I already posted my AU
[class descriptions](http://thebuildingcoder.typepad.com/blog/2010/09/autodesk-university-2010-classes.html) and
[materials](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-university-2010-class-materials.html).
The presentation went well and was good fun.
The next one,
[CP234-2](http://au.autodesk.com/?nd=event_class&session_id=7246&jid=614479),
on Revit 2011 programming optimization and filtered element collectors, is scheduled in an hour's time.
Right now I am sitting in on
[Matt Mason's presentation on Revit parameters](http://au.autodesk.com/?nd=event_class&session_id=7107&jid=613470).

Meanwhile, here is another interesting application of the doc.Delete method to access related elements that we already explored several times in the past for determining
[all kinds of useful object relationships](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html).
Now Joe Ye picked it up again to solve the following questions:

**Question:** When creating a floor, roof, filled region, etc., a sketch and a sketch plane are required.
I want to retrieve the sketch and sketch plane from an existing element.
However there is no API to do that.
Could you please tell me how to find these objects?

**Answer:** When creating a floor, slab and filled region etc., you draw a profile to define the element shape.
Revit automatically creates a sketch and sketch plane for the profile.
If the host element is deleted, the sketch and sketch plane are deleted accordingly.
The Document.Delete method returns all deleted element ids.
This mechanism can be used as follows to access the desired elements:

- Create a temporary transaction.- Delete the target element inside the transaction.- The sketch and sketch plane ids are included in the returned element id list.- Filter out the target elements.- Roll back the transaction to restore the target and all dependent elements and the model to its original state.

Using this approach, all elements that are closely related with a specified element can be retrieved.
For example, the detail lines that form the profile of a wall can be found.

Here is another similar question that can be answered using the same approach:

**Question:** I need to find out whether a given wall has an elevation profile and if so which.
I tried using a filtered element collector to retrieve all sketch objects in the model, but they have no properties that I can use to associate them with the wall.

Is there a way for a given wall element to find if there is a SketchPlane associated with it that defines its elevation profile?

Or, alternatively, is there a way for a given SketchPlane to find which object in the database it is attached to?

**Answer:** Just like above, one possible way is to delete the wall in a transaction using Document.Delete, which returns the deleted element ids. The wall and the sketch will be returned in the ElementId list. Then roll back the transaction, so the model remains unchanged.

Here is the resulting list of elements that were deleted due to the deletion of a specific sample wall element hosting them.
As you can see, the sketch element is included in the list:

![Wall with sketch](img/wall_with_sketch.png)

I used Joe's suggestion to create a new Building Coder sample external command CmdGetSketchElements to retrieve and list all the sketch elements associated with a selected host.

I added a variable 'showOnlySketchElements' to choose whether to list only the sketch elements, or all related elements.

I first implemented the string building code explicitly, then commented it out and rewrote it using generic Linq methods.
You can judge for yourself which variation you find clearer (and see whether you spot the obvious flaw in the StringBuilder implementation (the final trailing comma)):
```python
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
class CmdGetSketchElements : IExternalCommand
{
  const string \_caption = "Retrieve Sketch Elements";

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;
    Selection sel = uidoc.Selection;

    Reference r = sel.PickObject( ObjectType.Element,
      "Please pick an element" );

    Element elem = r.Element;

    Transaction trans = new Transaction( doc );

    trans.Start( \_caption );

    ICollection<ElementId> ids = doc.Delete( elem );

    trans.RollBack();

    bool showOnlySketchElements = true;

    /\*
    StringBuilder s = new StringBuilder(
      \_caption
      + " for host element "
      + Util.ElementDescription( elem )
      + ": " );

    foreach( ElementId id in ids )
    {
      Element e = doc.get\_Element( id );

      if( !showOnlySketchElements
        || e is Sketch
        || e is SketchPlane )
      {
        s.Append( Util.ElementDescription( e ) + ", " );
      }
    }
    \*/

    List<Element> a = new List<Element>(
      ids.Select( id => doc.get\_Element( id ) ) );

    string s = \_caption
      + " for host element "
      + Util.ElementDescription( elem )
      + ": ";

    s += string.Join( ", ",
      a.Where( e => !showOnlySketchElements
        || e is Sketch
        || e is SketchPlane )
      .Select( e => Util.ElementDescription( e ) )
      .ToArray() );

    Util.InfoMsg( s );

    return Result.Succeeded;
  }
}
```

Here is a simple sample model with a single floor element:
![Floor with sketch](img/floor_with_sketch.png)

Running the command on this returns the following list of all related elements:

Retrieve Sketch Elements for host element Floors <130589 Generic Floor - 400mm>: SketchPlane <130587 Level 1>, Sketch <130588 Sketch>, Floors <130589 Generic Floor - 400mm>, <Sketch> <130592 Model Lines>, <Sketch> <130595 Model Lines>, <Sketch> <130598 Model Lines>, <Sketch> <130601 Model Lines>, <Sketch> <130604 Model Lines>, <Sketch> <130607 Model Lines>, <Sketch> <130610 Model Lines>, SketchPlane <130621 Level 1>, Sketch <130622 Sketch>, Floor opening cut <130623 Opening Cut>, <Sketch> <130643 Model Lines>, <Sketch> <130667 Model Lines>, <Sketch> <130686 Model Lines>, <Sketch> <130702 Model Lines>, <Sketch> <130721 Model Lines>, <Sketch> <130752 Model Lines>, Automatic Sketch Dimensions <130775 Linear Dimension Style>, Automatic Sketch Dimensions <130776 Linear Dimension Style>, Automatic Sketch Dimensions <130777 Linear Dimension Style>, Automatic Sketch Dimensions <130778 Linear Dimension Style>, Automatic Sketch Dimensions <130779 Linear Dimension Style>, SketchPlane <130832 Level 1>, Sketch <130833 Sketch>, Floor opening cut <130834 Opening Cut>, <Sketch> <130848 Model Lines>, <Sketch> <130861 Model Lines>, <Sketch> <130878 Model Lines>, <Sketch> <130894 Model Lines>, Automatic Sketch Dimensions <130909 Linear Dimension Style>, Automatic Sketch Dimensions <130910 Linear Dimension Style>, SketchPlane <131013 Level 1>, Sketch <131014 Sketch>, Floor opening cut <131015 Opening Cut>, <Sketch> <131062 Model Lines>, Automatic Sketch Dimensions <131065 Linear Dimension Style>

As you can see, there are a large number of elements associated with this floor, including the sketch elements that we are interested in.

Setting 'showOnlySketchElements' to true returns the sketch elements only:

Retrieve Sketch Elements for host element Floors <130589 Generic Floor - 400mm>: SketchPlane <130587 Level 1>, Sketch <130588 Sketch>, SketchPlane <130621 Level 1>, Sketch <130622 Sketch>, SketchPlane <130832 Level 1>, Sketch <130833 Sketch>, SketchPlane <131013 Level 1>, Sketch <131014 Sketch>

Here is
[version 2011.0.81.0](zip/bc_11_81.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.