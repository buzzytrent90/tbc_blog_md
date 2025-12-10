---
post_number: "1375"
title: "Flatten To Directshape"
slug: "flatten_to_directshape"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'geometry', 'python', 'references', 'revit-api', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1375_flatten_to_directshape.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1375_flatten_to_directshape.html"
---

### Flatten All Elements to DirectShape
I am still busy preparing my presentations for Autodesk University and making slow progress due to handling too many Revit API issues in between.
And blogging, as well.
Here is a new cool sample contributed by Nikolay Shulga, Senior Principal Engineer in the Revit development team.
In his own words:
- Name: Flatten
- Motivation: I wanted to see whether DirectShapes could be used to lock down a Revit design – remove most intelligence, make it read-only, perhaps improve performance.
- Spec: converts full Revit elements into DirectShapes that hold the same shape and have the same categories.
- Implementation: see below.
- Cool API aspects: copy element’s geometry and use it elsewhere.
- Cool ways to use it: lock down your project; make a copy of your element for presentation/export.
- How it can be enhanced: the sky is the limit.
- A suitable sample model: any Revit project. Note that the code changes the current project – make a backup copy.
This fits in well with the growing interest in direct shapes, as you can observe by the rapidly growing number of entries in
the [DirectShape topic group](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.50).
I implemented a new external
command [CmdFlatten](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdFlatten.cs)
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) to
test and demonstrate this functionality.
Nikolay's original code was for a future release of Revit, so some backwards adaptation was necessary.
You can see the changes by looking at the last few GitHub commits.
Here is the final result for running in Revit 2016:

```
  const string _direct_shape_appGUID = "Flatten";

  Result Flatten(
    Document doc,
    ElementId viewId )
  {
    FilteredElementCollector col
      = new FilteredElementCollector( doc, viewId )
        .WhereElementIsNotElementType();

    Options geometryOptions = new Options();

    using( Transaction tx = new Transaction( doc ) )
    {
      if( tx.Start( "Convert elements to DirectShapes" )
        == TransactionStatus.Started )
      {
        foreach( Element e in col )
        {
          GeometryElement gelt = e.get_Geometry(
            geometryOptions );

          if( null != gelt )
          {
            string appDataGUID = e.Id.ToString();

            // Currently create direct shape
            // replacement element in the original
            // document – no API to properly transfer
            // graphic styles to a new document.
            // A possible enhancement: make a copy
            // of the current project and operate
            // on the copy.

            DirectShape ds
              = DirectShape.CreateElement( doc,
                e.Category.Id, _direct_shape_appGUID,
                appDataGUID );

            try
            {
              ds.SetShape(
                new List<GeometryObject>( gelt ) );

              // Delete original element

              doc.Delete( e.Id );
            }
            catch( Autodesk.Revit.Exceptions
              .ArgumentException ex )
            {
              Debug.Print(
                "Failed to replace {0}; exception {1} {2}",
                Util.ElementDescription( e ),
                ex.GetType().FullName,
                ex.Message );
            }
          }
        }
        tx.Commit();
      }
    }
    return Result.Succeeded;
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    // At the moment we convert to DirectShapes
    // "in place" - that lets us preserve GStyles
    // referenced by element shape without doing
    // anything special.

    return Flatten( doc, uidoc.ActiveView.Id );
  }
```

Here is a trivial example of flattening a wall to a direct shape:
![Original wall element](img/flatten_01.png)
This is the automatically generated DirectShape replacement element, retaining the wall category and storing its original element id:
![DirectShape replacement element](img/flatten_02.png)
Let's try it out on a slightly more complex model:
![Walls with doors and windows](img/flatten_03.png)
The family instances get lost by the conversion in its current implementation:
![DirectShape replacement element](img/flatten_04.png)
If you wish to retain family instances, you should probably explore their geometry in a little bit more detail, e.g., to extract all the solids they contain and convert them individually.
The current version is provided in the
module [CmdFlatten.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdFlatten.cs)
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2016.0.123.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.123.0).
Have fun playing with this, and many thanks to Nikolay for implementing and sharing it!