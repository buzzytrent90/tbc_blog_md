---
post_number: "0565"
title: "Nested Lighting Fixture Instances"
slug: "nested_instances"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'revit-api', 'selection', 'views']
source_file: "0565_nested_instances.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0565_nested_instances.html"
---

### Nested Lighting Fixture Instances

We already looked at a couple of aspects of nested family instances, such as access to
[nested instance geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html),
[creating nested families](http://thebuildingcoder.typepad.com/blog/2009/11/nested-family.html)
and
[determining whether a family instance is nested](http://thebuildingcoder.typepad.com/blog/2010/02/nested-family-instance.html).

Now Joel Spahn of [Lighting Analysts, Inc.](http://www.agi32.com) submitted a
[comment](http://thebuildingcoder.typepad.com/blog/2010/03/nested-family-utility-methods.html?cid=6a00e553e168978833014e86ea5f26970d#comment-6a00e553e168978833014e86ea5f26970d) on the
[nested family utility methods](http://thebuildingcoder.typepad.com/blog/2010/03/nested-family-utility-methods.html) raising
a new question related to nested lighting fixtures, efficiently resolved by Harry Mattison of the Revit development team:

**Question:** I have adapted this code to extract nested family instances and it works beautifully.
However, I have a new problem.

I need to extract the location and position of all the light source elements in a lighting fixture family which contains nested lighting fixture family instances.

Based on my testing, the filtered element collector does not return nested family instances.
If this is true, how can I query the nested family instances?

My initial solution to this problem was to open the family document to collect and iterate the nested family instances.
There are two problems to this approach:

1. It is performance intensive and dramatically slows the operation.- It cannot handle "dynamic" families (most important).

That is, how can I extract the nested family instances if the quantity and position of the nested family instances is different for each instance of the parent family in the project?

Example: I have a lighting fixture family (track light) that contains nested lighting fixture family instances (each can-light on the track). The nested instances are part of an array which is defined by the parent instance in the project.
Each parent instance can have a different number of nested instances, as described in the documentation on
[creating lighting fixtures with multiple light sources](http://docs.autodesk.com/REVIT/2010/ENU/Revit%20Architecture%202010%20Users%20Guide/RAC/index.html?url=WS73099cc142f487553b93539f117a2602bd5-62b4.htm,topicNumber=d0e97197).

Initially I thought that the FamilyInstance.SubComponents property would work for this, however it is always null! When does the SubComponents property return anything? The API documentation is very sparse.

Any ideas on how to access nested family instances given the parent instance?

**Answer:** I need more information about the SubComponents property that is not working for you.
I tried the following code with the "Table-Dining Round w Chairs.rfa" from the Revit Furniture library:
```csharp
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  FamilyInstance fi = doc.get\_Element(
    uidoc.Selection.PickObject(
      ObjectType.Element ).ElementId )
    as FamilyInstance;

  ElementSet subElemSet = fi.SubComponents;

  if( subElemSet != null )
  {
    string subElems = "";

    foreach( Element ee in subElemSet )
    {
      FamilyInstance f = ee as FamilyInstance;
      subElems = subElems + f.Name + "\n";
    }

    TaskDialog.Show( "Revit",
      "Subcomponent count = " + subElemSet.Size
      + "\n" + subElems );
  }

  FamilyInstance super = fi.SuperComponent
    as FamilyInstance;

  if( super != null )
  {
    TaskDialog.Show( "Revit",
      "SUPER component: " + super.Name );
  }
```

It provides the expected output:

![Nested furniture instances](img/nested_instances_1.png)

**Response:** The sample code works for me as well when I select an instance of Table-Dining Round w Chairs.

However, it does not work when I select an instance Nested Table Lamp, which is a very simple Lighting Fixture family with four nested Lighting Fixture Families.
Both family instances are visible from the 3D view.

**Answer:** The mesh geometry of the light sources can be found as follows:
```csharp
  FamilyInstance fi;
  GeometryElement geomElem = fi.get\_Geometry(
    app.Create.NewGeometryOptions() );

  foreach( GeometryObject geomObj
    in geomElem.Objects )
  {
    GeometryInstance geomInst
      = geomObj as GeometryInstance;

    if( geomInst != null )
    {
      foreach( GeometryObject geomObject
        in geomInst.SymbolGeometry.Objects )
      {
        GraphicsStyle gStyle = doc.get\_Element(
          geomObject.GraphicsStyleId )
          as GraphicsStyle;

        if( gStyle != null )
        {
          if( gStyle.GraphicsStyleCategory.Name
            == "Light Source" )
          {
            Mesh mesh = geomObject as Mesh;
          }
        }
      }
    }
  }
```

Is that sufficient for you to get the location & position information that you mentioned?

The lamps are not found by the SubComponents property because the "Shared" property is false:

![Shared property is false](img/nested_instances_2.png)

We will update the API documentation to explain this requirement.

**Response:** I thought of using the mesh geometry, but this approach doesn't allow access to important parameter data such as the photometric data that the mesh is built from.

However, I forgot about the behaviour of "shared" light fixtures.
This seems to solve the issue and after sharing all the nested light fixtures, the subcomponents property works as expected.

Many thanks to Joel and Harry for this fruitful exploration, and for kindly sharing the results with us!