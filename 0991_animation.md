---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: code_example
optimization_date: '2025-12-11T11:44:15.071319'
original_url: https://thebuildingcoder.typepad.com/blog/0991_animation.html
post_number: 0991
reading_time_minutes: 4
series: general
slug: animation
source_file: 0991_animation.htm
tags:
- elements
- family
- levels
- python
- references
- revit-api
- rooms
- selection
- transactions
- views
- walls
title: Animation and the DisplacementElement Class
word_count: 825
---

### Animation and the DisplacementElement Class

A recent developer query asks about animating elements in the Revit model and updating the display within a loop in the external command to show the step-by-step displacements.

This provides a welcome opportunity to highlight another one of the
[Revit 2014 SDK Samples](http://thebuildingcoder.typepad.com/blog/2013/07/revit-2014-obj-exporter-and-new-sdk-samples.html):

#### The DisplacementElementAnimation SDK Sample

One of the very visually impressive Revit 2014 API samples is the DisplacementElementAnimation application.

It automatically animates groups of structural building members in any Revit BIM, proceeding sequentially from bottom level to top.

It creates and executes an animation of the model by disassembling and then reassembling it from the ground up.
The members are sorted into groups based on category and level and displaced from their actual position.
Each group is then animated using the Idling event until the model is reassembled.

It is an external application, so it is not automatically made available by installing RvtSamples, which just provides access to the external command SDK samples.

It makes use of the new DisplacementElement to visually create a view-specific temporary displacement of the elements.
The DisplacementElement class is a view-specific element that causes other elements to appear to be displaced from their actual locations.

The DisplacementElement does not actually change the location of any model elements; it merely causes them to be displayed in a different location.

The add-in subscribes to the Idling event to implement the step-by-step changes to the displacement offsets required to animate the entire model.

The project files include:

- Application.cs – implements the Revit add-in interface IExternalApplication to add the ribbon panel.
- DisplacementStructureModelAnimatorCommand.cs – the external command that initializes and starts the model animation.
- DisplacementStructureModelAnimator.cs – a class that executes an animation of structural model elements using DisplacementElements.

The displacement elements enable a visual animation to be presented without physically moving the elements, which would distort the model due to joins and geometric relationships between the elements.

An element may only be displaced by a single DisplacementElement in any view.
Assigning an element to more than one DisplacementElement is an error condition.

A DisplacementElement can declare another DisplacementElement as its parent.
In that case, its transform will be concatenated with that of the parent, and the displacement of its associated elements will be relative to the parent DisplacementElement.

The elements are grouped using parent and child displacements, which enables the elements to inherit a displacement in Z while varying in X and Y.

This sample is geared towards a structural model.
It is independent of the actual model it is run in, but only foundations, structural columns, walls, floors, and structural framing will be displaced and animated.

To run it, install the external application and open the sample model provided or any other structural model.
Select the Add-ins > Displacement > Displacement animation command.
The animation will begin and run automatically.

For a video showing a sample run, please refer to the recording included with the Revit 2014
[DevDays online material](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html#2).

#### Animating Movement of a Revit Family Instance or Other Element

With the overview of the DisplacementElementAnimation sample in hand, let's return to the developer query:

**Question:** I would like to implement an animated movement of a family instance in the model.
Here is my current code attempting this.
I tried to execute it using both manual and automatic transactions, but Revit does not refresh the view per transform although I included a call to Document.Regenerate in the loop:
```python
[Transaction( TransactionMode.Automatic )]
public class Command : IExternalCommand
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

    Reference r = uidoc.Selection.PickObject(
      ObjectType.Element, "Pick element" );

    Element e = doc.GetElement( r );

    using( Transaction tx = new Transaction( doc ) )
    {
      for( int i = 0; i < 50; i++ )
      {
        //tx.Start("modify");
        double dt = 10;
        XYZ v = new XYZ( 0.0, dt, 0.0 );
        ElementTransformUtils.MoveElement(
          e.Document, e.Id, v );
        //tx.Commit();
        System.Threading.Thread.Sleep( 1000 );
        doc.Regenerate();
      }
    }
    return Result.Succeeded;
  }
}
```

Is it possible to achieve this kind of simulation and animation in Revit?

Please let me know if I need to approach this differently or if you have other ideas.

**Answer:** While in an external command that has not yet been completed, it is not always enough to close a transaction, although closing a transaction is necessary for propagating all changes correctly throughout the mode.

In order for changes to appear correctly in the current view, an additional call to the UIDocument.RefreshActiveView method may also be needed.
Please keep in mind that it should not be called during the Idling event.

You can simply refer to the abovementioned DisplacementElementAnimation SDK sample to see how to implement a complete animation.