---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.3
content_type: code_example
optimization_date: '2025-12-11T11:44:14.232186'
original_url: https://thebuildingcoder.typepad.com/blog/0596_create_extrusion_fam.html
post_number: 0596
reading_time_minutes: 8
series: general
slug: create_extrusion_fam
source_file: 0596_create_extrusion_fam.htm
tags:
- csharp
- elements
- family
- filtering
- levels
- parameters
- python
- revit-api
- selection
- transactions
- views
title: Creating and Inserting an Extrusion Family
word_count: 1668
---

### Creating and Inserting an Extrusion Family

Today is the
[Pentecost](http://en.wikipedia.org/wiki/Pentecost) or
[Whit Monday](http://en.wikipedia.org/wiki/Whit_Monday) holiday
in Neuchâtel, which gave me some time to explore an interesting and important issue in a bit more depth.

Several developers recently asked about creating families programmatically, so here is a first simple example to demonstrate this.
It creates a new structural stiffener family as an extrusion of a profile based on a list of XYZ points and inserts an instance of it into the current document.
Here are the required steps one by one:

1. [Create an extrusion element from a profile based on a list of XYZ points](#1).- [Create a new structural stiffener family using the extrusion element](#2).- [Insert an instance of the new family into the current document](#3).

#### 1. Creating an Extrusion for a Profile

The extrusion profile is defined by a list of XYZ points, in millimetres.
In this example, we just use a very trivial rectangular shape.
The thickness is also given in millimetres:
```csharp
  static int n = 4;

  static List<XYZ> \_countour = new List<XYZ>( n )
  {
    new XYZ( 0 , -75 , 0 ),
    new XYZ( 508, -75 , 0 ),
    new XYZ( 508, 75 , 0 ),
    new XYZ( 0, 75 , 0 )
  };

  const double \_thicknessMm = 20.0;
```

Since all the input data is given in millimetres, and the extrusion has to be defined in the internal
[Revit database unit](http://thebuildingcoder.typepad.com/blog/2011/01/unit-conversion-and-new-blogs.html) for
length, which is
[feet](http://thebuildingcoder.typepad.com/blog/2011/03/internal-imperial-units.html),
we need a conversion method for that, e.g.:
```python
  /// <summary>
  /// Conversion factor from millimetre to foot
  /// </summary>
  const double \_mm\_to\_foot = 1 / 304.8;

  /// <summary>
  /// Convert a given length to feet
  /// </summary>
  double MmToFoot( double length\_in\_mm )
  {
    return \_mm\_to\_foot \* length\_in\_mm;
  }

  /// <summary>
  /// Convert a given point defined in millimetre units to feet
  /// </summary>
  XYZ MmToFootPoint( XYZ p )
  {
    return p.Multiply( \_mm\_to\_foot );
  }
```

In order to create an extrusion from the contour data, we need a sketch plane and a CurveArrArray instance.
The curve array array contains a list of loops, since the extrusion may require an outer loop and optional inner loops representing cavities.

Here is a helper method to create CurveArray from our XYZ point list:
```csharp
  /// <summary>
  /// Convert a given list of XYZ points
  /// to a CurveArray instance.
  /// The points are defined in millimetres,
  /// the returned CurveArray in feet.
  /// </summary>
  CurveArray CreateProfile(
    List<XYZ> pts,
    CreationApplication creapp )
  {
    CurveArray profile = new CurveArray();

    int n = \_countour.Count;

    for( int i = 0; i < n; ++i )
    {
      int j = (0 == i) ? n - 1 : i - 1;

      profile.Append( creapp.NewLineBound(
        MmToFootPoint( pts[j] ),
        MmToFootPoint( pts[i] ) ) );
    }
    return profile;
  }
```
In this case, the method is designed to work in a family based on the structural stiffener family template, which contains a sketch plane in the XY plane named "Ref. Level" that we can work with.
Actually, it redundantly contains three of them, and we can simply pick the first one.
One way to select the existing sketch plane if we know the exact structure of the document we are working with is to explicitly use a hard coded element id for it:
```csharp
  SketchPlane sketch = doc.get\_Element(
    new ElementId( 501 ) ) as SketchPlane;
```

That has the advantage of not being language dependent, but makes us completely dependent on the internal database structure of the family template file instead.
A slightly more flexible approach is to search for the element by name using a helper function such as this:
```csharp
  /// <summary>
  /// Return the first element found of the
  /// specific target type with the given name.
  /// </summary>
  Element FindElement(
    Document doc,
    Type targetType,
    string targetName )
  {
    return new FilteredElementCollector( doc )
      .OfClass( targetType )
      .First<Element>( e => e.Name.Equals( targetName ) );
  }
```

This method is making use of [LINQ](http://thebuildingcoder.typepad.com/blog/2009/07/language-integrated-query-linq.html).
A more efficient approach would use a filtered element collector based on a
[parameter](http://thebuildingcoder.typepad.com/blog/2010/06/parameter-filter.html)
[filter](http://thebuildingcoder.typepad.com/blog/2010/06/element-name-parameter-filter-correction.html) to
check the name instead.
The advantage of that approach is that the filtering then happens on the internal Revit side, before the retrieved element data is passed out from Revit to the managed .NET environment.
In our case, there are not many sketch planes in the document to examine, so the overhead is negligible.

Using this method to find the sketch plane, and with the curve array and thickness in hand, the call to create the extrusion element in a family document can be set up like this:
```python
  /// <summary>
  /// Create an extrusion from a given thickness
  /// and list of XYZ points defined in millimetres
  /// in the given family document, which  must
  /// contain a sketch plane named "Ref. Level".
  /// </summary>
  Extrusion CreateExtrusion(
    Document doc,
    List<XYZ> pts,
    double thickness )
  {
    FamilyItemFactory factory = doc.FamilyCreate;

    CreationApplication creapp = doc.Application.Create;

    SketchPlane sketch = FindElement( doc,
      typeof( SketchPlane ), "Ref. Level" )
        as SketchPlane;

    CurveArrArray curveArrArray = new CurveArrArray();

    curveArrArray.Append( CreateProfile( pts, creapp ) );

    double extrusionHeight = MmToFoot( thickness );

    return factory.NewExtrusion( true,
      curveArrArray, sketch, extrusionHeight );
  }
```

Next, let us see how to create a new family document in the background to make use of this method.

#### 2. Creating an Extrusion Family

Creating a new family to contain our extrusion is a simple process:

- Open the appropriate family template file.- Create the extrusion.- Save and close the file.

In theory, it should be possible to avoid the last step of saving the newly generated family document to a disk file, since it is possible to load the generated family document into a project environment straight from memory.
Unfortunately, I was unable to get this to work, because the call to 'fdoc.LoadFamily( doc )' caused a "serious error".
Saving the family to a disk file and using 'doc.LoadFamily( filename, out family )' instead works, though.

It would also not be necessary to require a different file to be open to start with, but if it is not, it is impossible to close the family document.
It has to be closed, though, in order to use the new Revit 2012 API OpenAndActivateDocument method on it, in case we want to see the results of our new family definition in the family environment instead of looking at an insertion in the project environment.
```python
  string templateFileName = Path.Combine( \_path,
    \_family\_template\_name + \_family\_template\_ext );

  Document fdoc = app.NewFamilyDocument(
    templateFileName );

  if( null == fdoc )
  {
    message = "Cannot create family document.";
    return Result.Failed;
  }

  Transaction t = new Transaction( fdoc,
    "Create structural stiffener family" );

  t.Start();

  CreateExtrusion( fdoc, \_countour, \_thicknessMm );

  t.Commit();

  // save our new family background document
  // and reopen it in the Revit user interface:

  string filename = Path.Combine(
    Path.GetTempPath(), \_family\_name + \_rfa\_ext );

  SaveAsOptions opt = new SaveAsOptions();
  opt.OverwriteExistingFile = true;

  fdoc.SaveAs( filename, opt );

  // cannot close the newly generated family file
  // if it is the only open document; that throws
  // an exception saying "The active document may
  // not be closed from the API."

  fdoc.Close( false );
```

Now that the family has been created and save to a file, we can either add a call to open it programmatically in the foreground to view it using 'uiapp.OpenAndActivateDocument( filename )', or, assuming that we have an open active project document in the Revit user interface, immediately insert an instance of it.
In this case, let's go for the latter.

#### 3. Inserting an Instance of the New Family

Inserting a new family instance is something that we already covered numerous times in the past.
The last contribution on
[placing a line based detail item instance](http://thebuildingcoder.typepad.com/blog/2011/06/placing-a-line-based-detail-item-instance.html) points
to some previous posts on this topic.

Here is an example of how we can place an instance of our newly created stiffener family into the current project at a selected point.
We make use of one of the simplest overloads of the NewFamilyInstance method taking just a point, the family symbol, and a structural type argument.

Before inserting the instance, we need to ensure that the family is in fact loaded at all.

Before loading the family, we need to check whether it has already been loaded.
This may happen, for instance, if the command is run multiple times.

Once loaded, we extract the family symbol from its container family, prompt the user for an insertion point, and create the new instance:

- Start a transaction.- Retrieve all families.- Check whether the one we need has been loaded already.- If not, load it.- Extract its one and only symbol.- Insert an instance of it.

Here is the code to achieve these steps:
```csharp
  t = new Transaction( doc,
    "Insert structural stiffener family instance" );

  t.Start();

  // load the family ... but check whether
  // it was already loaded first

  Family family = null;

  FilteredElementCollector a
    = new FilteredElementCollector( doc )
      .OfClass( typeof( Family ) );

  int n = a.Count<Element>(
    e => e.Name.Equals( \_family\_name ) );

  if( 0 < n )
  {
    family = a.First<Element>(
      e => e.Name.Equals( \_family\_name ) )
        as Family;
  }
  else
  {
    // calling this without a prior call
    // to SaveAs causes a "serious error":
    //family = fdoc.LoadFamily( doc );

    doc.LoadFamily( filename, out family );
  }

  FamilySymbol symbol = null;

  foreach( FamilySymbol s in family.Symbols )
  {
    symbol = s;

    // our family only contains one
    // symbol, so pick it and leave

    break;
  }

  XYZ p = uidoc.Selection.PickPoint(
    "Please pick a point for family instance insertion" );
  StructuralType st = StructuralType.UnknownFraming;
  doc.Create.NewFamilyInstance( p, symbol, st );
  t.Commit();
```

The steps presented above answer a whole bunch of questions and should provide a good starting point for your own further explorations.

Here is [Stifferener.zip](zip/Stiffener02.zip) containing the complete source code and Visual Studio solution of the external command implementing them.

Another very common question in this area is how to perform Boolean operations, both in the family and project context.
That is a topic that I would like to take a look at in the near future, expanding on the example presented here.