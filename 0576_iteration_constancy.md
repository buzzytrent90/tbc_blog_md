---
post_number: "0576"
title: "Iteration and Springtime – Change is the Only Constant"
slug: "iteration_constancy"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'python', 'revit-api', 'selection', 'transactions', 'walls']
source_file: "0576_iteration_constancy.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0576_iteration_constancy.html"
---

### Iteration and Springtime – Change is the Only Constant

I am back again from a wonderful relaxing Easter holiday with beautiful weather and an almost obscene wealth of blossoming trees and flowers in the Swiss
[Emmental](http://en.wikipedia.org/wiki/Emmental) region:

![Emmental in bloom](img/emmental_in_bloom.jpg)

Spring time really gives us an overwhelming impression of nature's capacity for rapid change!
And change is the only constant...
And, as we see below, change is a constant challenge in programming as well...

Meanwhile, here are quick pointers to two of the many noteworthy items that cropped up during the past week:

- Jose Guia'S bimKICKS [Revit command prompt](http://blog.bimkicks.com/post/2011/04/01/REVIT-COMMAND-PROMPT-aww-yeah!!!!!!!.aspx).- [Revit AppStore](http://www.revitappstore.com) updated for Revit 2012.

For my own contribution on my first official day back at work and in the blogosphere, here is a summary of an issue I was dealing with just before Easter:

I heard of several different developers struggling with iterations over collections returned by properties of the Revit API classes.
The last one we looked into was related to the
[iteration over an unordered set](http://thebuildingcoder.typepad.com/blog/2011/02/iterating-over-an-unordered-set-property.html) returned
by the CurtainGrid Cells property.

Now the question was encountered again, in a seemingly different form, yet with a similar answer.
In this case, both the question and the solution to it were found by Winston Yaw of
[RISA Technologies](http://www.risatech.com):

**Question:** I ran the following code on a simple Revit Structure model that only has one structural wall and I got an index of -1 for the first layer in the wall's compound structure.
That makes no sense.
I probably overlooked something simply, but what?
```csharp
  FilteredElementIterator ^elementsIter;

  FilteredElementCollector ^filteredCollection
    = gcnew FilteredElementCollector(
      pApplication->ActiveUIDocument->Document );

  filteredCollection->OfClass( Wall::typeid );

  elementsIter = filteredCollection
    ->GetElementIterator();

  Wall ^testWall = nullptr;

  // Get a wall

  while( elementsIter->MoveNext() )
  {
    testWall = dynamic\_cast<Wall^>(
      elementsIter->Current );

    if( testWall )
    {
      // Look at compound structure

      CompoundStructure^ pCompound
        = testWall->WallType->GetCompoundStructure();

      if( pCompound )
      {
        // Look at first layer

        System::Collections::IEnumerator ^layerIter
          = pCompound->GetLayers()->GetEnumerator();

        while( layerIter->MoveNext() )
        {
          CompoundStructureLayer^ pLayer
            = dynamic\_cast<CompoundStructureLayer^>(
              layerIter->Current );

          // Get index of first layer
          Int32 index = pCompound->GetLayers()->IndexOf(
            pLayer );

          // Display in MessageBox

          MessageBox::Show( index.ToString() );
        }
      }
    }
  }
```

In this case, I was expecting a zero-valued index to be displayed.
Can you tell me what's wrong, please?
Thanks.

**Answer:** I found the problem myself.
The code above is calling the GetLayers method twice: once to get the Enumerator, and again to use IndexOf.

In summary, the code is doing something like this:

- GetLayers()->GetEnumerator();- GetLayers()->IndexOf(layer);

The problem doesn't occur when an intermediate variable to store the collection returned is introduced as follows:

- IList<> ^layers = GetLayers();- layers->GetEnumerator();- layers->IndexOf();

Jeremy responds: Yes, exactly.
Congratulations on finding the root of the problem!
As said, we have seen similar issues in the past, e.g. related to the
[iteration over an unordered set property](http://thebuildingcoder.typepad.com/blog/2011/02/iterating-over-an-unordered-set-property.html).

#### Accessing the Compound Layer Structure in Revit 2012

For completeness sake, I tested accessing the compound layer structure and the IndexOf method using C# on both an architectural wall in Revit Architecture 2012 and a structural wall in Revit Structure 2012.
Since I store the layer collection returned by the GetLayers method in an intermediate variable, just as suggested in your solution, I obviously don't see the problem you describe.
Here is the code that I used, which includes comments to highlight the differences between the Revit 2011 and 2012 APIs, excerpted from the external command CmdWallLayers in
[The Building Coder samples migrated to Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/02/iterating-over-an-unordered-set-property.html) and
with a call added to exercise the IndexOf method instead of using an additional variable 'i' for the index:
```python
[Transaction( TransactionMode.Automatic )]
class CmdWallLayers : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    UIDocument uidoc = app.ActiveUIDocument;
    Document doc = app.ActiveUIDocument.Document;

    // retrieve selected walls, or all walls,
    // if nothing is selected:

    List<Element> walls = new List<Element>();
    if( !Util.GetSelectedElementsOrAll(
      walls, uidoc, typeof( Wall ) ) )
    {
      Selection sel = uidoc.Selection;
      message = ( 0 < sel.Elements.Size )
        ? "Please select some wall elements."
        : "No wall elements found.";
      return Result.Failed;
    }

    //int i; // 2011
    int n;
    double halfThickness, layerOffset;
    Creator creator = new Creator( doc );
    XYZ lcstart, lcend, v, w, p, q;

    foreach( Wall wall in walls )
    {
      string desc = Util.ElementDescription( wall );

      LocationCurve curve
        = wall.Location as LocationCurve;

      if( null == curve )
      {
        message = desc + ": No wall curve found.";
        return Result.Failed;
      }

      // wall centre line and thickness:

      lcstart = curve.Curve.get\_EndPoint( 0 );
      lcend = curve.Curve.get\_EndPoint( 1 );
      halfThickness = 0.5 \* wall.WallType.Width;
      v = lcend - lcstart;
      v = v.Normalize(); // one foot long
      w = XYZ.BasisZ.CrossProduct( v ).Normalize();
      if( wall.Flipped ) { w = -w; }

      p = lcstart - 2 \* v;
      q = lcend + 2 \* v;
      creator.CreateModelLine( p, q );

      q = p + halfThickness \* w;
      creator.CreateModelLine( p, q );

      // exterior edge

      p = lcstart - v + halfThickness \* w;
      q = lcend + v + halfThickness \* w;
      creator.CreateModelLine( p, q );

      //CompoundStructure structure = wall.WallType.CompoundStructure; // 2011
      CompoundStructure structure = wall.WallType.GetCompoundStructure(); // 2012

      //CompoundStructureLayerArray layers = structure.Layers; // 2011
      IList<CompoundStructureLayer> layers = structure.GetLayers(); // 2012

      //i = 0; // 2011
      //n = layers.Size; // 2011
      n = layers.Count; // 2012

      Debug.Print(
        "{0} with thickness {1}"
        + " has {2} layer{3}{4}",
        desc,
        Util.MmString( 2 \* halfThickness ),
        n, Util.PluralSuffix( n ),
        Util.DotOrColon( n ) );

      if( 0 == n )
      {
        // interior edge
        p = lcstart - v - halfThickness \* w;
        q = lcend + v - halfThickness \* w;
        creator.CreateModelLine( p, q );
      }
      else
      {
        layerOffset = halfThickness;
        foreach( CompoundStructureLayer layer
          in layers )
        {
          Debug.Print(
            "  Layer {0}: function {1}, "
            + "thickness {2}",
            //++i, // 2011
            layers.IndexOf( layer ), // 2012
            layer.Function,
            Util.MmString( layer.Width ) );

          //layerOffset -= layer.Thickness; // 2011
          layerOffset -= layer.Width; // 2012

          p = lcstart - v + layerOffset \* w;
          q = lcend + v + layerOffset \* w;
          creator.CreateModelLine( p, q );
        }
      }
    }
    return Result.Succeeded;
  }
}
```

Here is the result of running this on a structural wall:

```
Walls <246407 Generic - 8"> with thickness 203.2 mm has 1 layer:
  Layer 0: function Structure, thickness 203.2 mm
```

As said, there is no problem with the IndexOf method itself.
As you noted yourself, the issue you observed was due to the repeated call to GetLayers to retrieve the collection multiple times while iterating over it at the same time.