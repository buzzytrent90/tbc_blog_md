---
post_number: "1360"
title: "Revitlookup"
slug: "revitlookup"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'python', 'references', 'revit-api', 'rooms', 'selection', 'sheets', 'transactions', 'views']
source_file: "1360_revitlookup.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1360_revitlookup.html"
---

### SVG, In-Memory Family, RevitLookup BoundingBox
I have been blogging less here and doing more work elsewhere in the past couple of days.
The [cloud accelerator](http://autodeskcloudaccelerator.com/prague/) kept
me busy last week, I am working hard on
the [CompHound project](https://github.com/CompHound/CompHound.github.io) and documenting that work
on [The 3D Web Coder](http://the3dwebcoder.typepad.com).
Besides that, I submitted several enhancements to various Revit add-in projects.
Here are some of my recent activities:
- [CompHound component tracker project development](#2)
- [RevitLookup displays element bounding box](#3)
- [SvgExport for Revit 2016](#4)
- [In-memory family creation and structural stiffener migration](#5) from 2012 to 2016
#### CompHound Component Tracker Project Development
If you are interested in connecting Revit or any other desktop application with the cloud, you may be interested in
my [CompHound](https://github.com/CompHound/CompHound.github.io)
and [FireRating in the Cloud](https://github.com/jeremytammik/firerating) projects.
CompHound is a cloud-based universal component and asset usage analysis, report, bill of materials and visualisation project.
I am working on it to provide sample code for two presentations on connecting the desktop with the cloud at
[RTC Europe](http://www.rtcevents.com/rtc2015eu) in Budapest end of October and
[Autodesk University](http://au.autodesk.com) in Las Vegas in December.
It currently consists of two parts:
- [CompHoundWeb](https://github.com/CompHound/CompHoundWeb),
a Node.js web server, mongo database and
[Autodesk View and Data API](https://developer.autodesk.com) viewer.
- [CompHoundRvt](https://github.com/CompHound/CompHoundRvt),
a Revit add-in to populate the CompHound web database with ±BIM family instances.
CompHound is based on and derived from the \*\*FireRating in the Cloud\*\* sample, consisting of the
[FireRatingCloud](https://github.com/jeremytammik/FireRatingCloud) C# .NET REST API client Revit add-in and the
[fireratingdb](https://github.com/jeremytammik/firerating) Node.js mongoDB web server.
I hope you are aware of the standard Revit SDK FireRating sample... FireRatingCloud is a rewrite of it to support multiple projects and collect the relevant data from all of them in one single scalable cloud database. It is really minimal and efficient, well worth checking out.
The FireRating in the Cloud sample has no user interface, whereas CompHound does. It is still under development, and will soon include reporting, bill of materials, [View and Data API](https://developer.autodesk.com) viewing and navigation functionality.
For more information, please refer to the project home pages on GitHub
and [The 3D Web Coder](http://the3dwebcoder.typepad.com) detailed articles describing its implementation and evolution so far:
- [Project definition and christening](http://the3dwebcoder.typepad.com/blog/2015/09/comphound-jsfiddle-and-my-first-react-component.html).
- [First implementation](http://the3dwebcoder.typepad.com/blog/2015/09/comphound-restsharp-mongoose-put-and-post.html#2) based on
[fireratingdb](https://github.com/jeremytammik/firerating) and
[FireRatingCloud](https://github.com/jeremytammik/FireRatingCloud).
- [Steps towards a database table view](http://the3dwebcoder.typepad.com/blog/2015/09/towards-a-comphound-mongo-database-table-view.html).
- [The CompHound Mongoose DataTable](http://the3dwebcoder.typepad.com/blog/2015/09/the-comphound-mongoose-datatable.html)
- [CompHound Heroku Deployment](http://the3dwebcoder.typepad.com/blog/2015/09/comphound-heroku-deployment-and-urban-farming.html)
Back from the cloud to the pure desktop Revit API...
#### RevitLookup Displays Element Bounding Box
Participants at the cloud accelerator were interested in the bounding box of various elements, to enable zooming to rooms and floors in the
[View and Data API](https://developer.autodesk.com/) viewer.
I was surprised that RevitLookup did not list that information.
Not that surprising, actually: in general, it just lists property values, and mostly does not evaluate and display results from explicit method calls.
I added just four lines of code:

```
  BoundingBoxXYZ bb = elem.get_BoundingBox( null );
  if( null != bb )
  {
    data.Add( new Snoop.Data.Object( "Bounding box", bb ) );
  }
```

The update is provided in
[RevitLookup release 2016.0.0.11](https://github.com/jeremytammik/RevitLookup/releases/tag/2016.0.0.11).
Please always download and install the latest version to ensure you are up to date.
#### SvgExport for Revit 2016
I also migrated SvgExport, my Revit add-in to export element geometry to SVG, discussed in the posts on
[displaying 2D graphics via a node server](http://the3dwebcoder.typepad.com/blog/2015/04/displaying-2d-graphics-via-a-node-server.html) and
[sending a room boundary to an SVG node web server](http://thebuildingcoder.typepad.com/blog/2015/04/sending-a-room-boundary-to-an-svg-node-web-server.html).
The current version including some small clean-up is now
[SvgExport release 2016.0.0.1](https://github.com/jeremytammik/SvgExport/releases/tag/2016.0.0.1).
#### In-memory Family Creation and Structural Stiffener Migration
Finally, to complete the series of updates and migrations, I looked at implementing in-memory family creation, loading, and instance placement.
This issue was prompted by a developer query:
\*\*Question:\*\*
When creating a family document the title is an automatically assigned name.
Is it possible to change the title and family name without saving the document?
I don't want to save the file.
I want to load it directly into another model that is in memory.
If I have to access the hard disk change the name each time. I have to potentially create thousands of very small families.
Theoretically the hard disk is about 80000 times slower then memory.
So I want to avoid setting the family name by saving the document.
\*\*Answer:\*\*
As you know and are already doing, you can create a family definition in memory and load it into the project without ever saving to disk.
I tried to implement that back in 2011, for Revit 2012, as explained in the discussion
on [creating and inserting an extrusion family](http://thebuildingcoder.typepad.com/blog/2011/06/creating-and-inserting-an-extrusion-family.html) for
a structural stiffener.
At that time, the pure in-memory loading did not work as expected.
That has been fixed in the meantime, though, probably in Revit 2013 or so.
In order to answer your question on setting the family and symbol name, I created a
new [Stiffener GitHub repository](https://github.com/jeremytammik/Stiffener) for
it and migrated it to Revit 2014.
The first Revit 2014 version is just a flat migration that reproduces the Revit 2012 behaviour, saving the family definition to disk before loading it:
![Revit 2014 stiffener family saved to disk](img/stiffener_2014.png)
I then implemented some significant changes to enable loading the family directly from memory.
That works, and presumably displays the behaviour that you are currently observing:
![Revit 2014 stiffener family loaded from memory](img/stiffener_2014_in_memory.png)
Next, I set the family name. That is possibly a way to achieve part of what you want:
![Revit 2014 stiffener family loaded from memory with family name](img/stiffener_2014_in_memory_with_name.png)
Finally, I migrated to Revit 2015 and 2016 as well, and added a line of code to set the symbol name as well as the family name:
![Revit 2014 stiffener family loaded from memory with family and symbol name](img/stiffener_2016_in_memory_with_both_names.png)
Does that fulfil your needs?
Here is the complete list of releases and modifications:
- [2012.0.0.0](https://github.com/jeremytammik/Stiffener/releases/tag/2012.0.0.0) initial commit for Revit 2012, copied from the June 13 2011 blog post
- [2014.0.0.0](https://github.com/jeremytammik/Stiffener/releases/tag/2014.0.0.0) migration to Revit 2014 including code update, .NET framework target 4.0, removal of architecture mismatch warning and obsolete API usage
- [2014.0.0.1](https://github.com/jeremytammik/Stiffener/releases/tag/2014.0.0.1) updated to place instance from in-memory family definition with no file save, restructured logic, added using statements around transactions
- [2014.0.0.2](https://github.com/jeremytammik/Stiffener/releases/tag/2014.0.0.2) set family name after loading into project
- [2014.0.0.3](https://github.com/jeremytammik/Stiffener/releases/tag/2014.0.0.3) set 'copy local' to false on the Revit API assemblies
- [2015.0.0.0](https://github.com/jeremytammik/Stiffener/releases/tag/2015.0.0.0) flat migration to Revit 2015
- [2016.0.0.0](https://github.com/jeremytammik/Stiffener/releases/tag/2016.0.0.0) migration to Revit 2016, need to activate symbol, set symbol name as well as family
- [2016.0.0.1](https://github.com/jeremytammik/Stiffener/releases/tag/2016.0.0.1) fixed deprecated API usage
Please use the GitHub diff tool to compare the differences between each of these.
For instance, to see how I set the family name, you can compare version 2014.0.0.1 and 2014.0.0.2 using the
URL [github.com/jeremytammik/Stiffener/compare/2014.0.0.1...2014.0.0.2](https://github.com/jeremytammik/Stiffener/compare/2014.0.0.1...2014.0.0.2).
It was fun to migrate all the way from release 2012 all the way to 2016 in one single sitting.
The final implementation now looks like this and includes a couple of comments that illustrate further implementation details:

```
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  #region Constants
  ///
  /// Family template filename extension
  ///
  const string _family_template_ext = ".rft";

  ///
  /// Revit family filename extension
  ///
  const string _rfa_ext = ".rfa";

  ///
  /// Family template library path
  ///
  //const string _path = "C:/ProgramData/Autodesk/RST 2012/Family Templates/English";
  const string _path = "C:/Users/All Users/Autodesk/RVT 2014/Family Templates/English";

  ///
  /// Family template filename stem
  ///
  const string _family_template_name = "Metric Structural Stiffener";

  // Family template path and filename for imperial units

  //const string _path = "C:/ProgramData/Autodesk/RST 2012/Family Templates/English_I";
  //const string _family_name = "Structural Stiffener";

  ///
  /// Name of the generated stiffener family
  ///
  const string _family_name = "Stiffener";

  ///
  /// Conversion factor from millimetre to foot
  ///
  const double _mm_to_foot = 1 / 304.8;
  #endregion // Constants

  ///
  /// Convert a given length to feet
  ///
  double MmToFoot( double length_in_mm )
  {
    return _mm_to_foot * length_in_mm;
  }

  ///
  /// Convert a given point defined in millimetre units to feet
  ///
  XYZ MmToFootPoint( XYZ p )
  {
    return p.Multiply( _mm_to_foot );
  }

  static int n = 4;

  ///
  /// Extrusion profile points defined in millimetres.
  /// Here is just a very trivial rectangular shape.
  ///
  static List<XYZ> _countour = new List<XYZ>( n )
  {
    new XYZ( 0 , -75 , 0 ),
    new XYZ( 508, -75 , 0 ),
    new XYZ( 508, 75 , 0 ),
    new XYZ( 0, 75 , 0 )
  };

  ///
  /// Extrusion thickness for stiffener plate
  ///
  const double _thicknessMm = 20.0;

  ///
  /// Return the first element found of the
  /// specific target type with the given name.
  ///
  Element FindElement(
    Document doc,
    Type targetType,
    string targetName )
  {
    return new FilteredElementCollector( doc )
      .OfClass( targetType )
      .First<Element>( e => e.Name.Equals( targetName ) );

    // Obsolete code parsing the collection for the
    // given name using a LINQ query.

    //var targetElems
    //  = from element in collector
    //    where element.Name.Equals( targetName )
    //    select element;

    //return targetElems.First
    //List elems = targetElems.ToList();

    //if( elems.Count > 0 )
    //{  // we should have only one with the given name.
    //  return elems[0];
    //}

    // cannot find it.
    //return null;

    /*
    // most efficient way to find a named
    // family symbol: use a parameter filter.

    ParameterValueProvider provider
      = new ParameterValueProvider(
        new ElementId( BuiltInParameter.DATUM_TEXT ) ); // VIEW_NAME for a view

    FilterStringRuleEvaluator evaluator
      = new FilterStringEquals();

    FilterRule rule = new FilterStringRule(
      provider, evaluator, targetName, true );

    ElementParameterFilter filter
      = new ElementParameterFilter( rule );

    return new FilteredElementCollector( doc )
      .OfClass( targetType )
      .WherePasses( filter )
      .FirstElement();
    */
  }

  ///
  /// Convert a given list of XYZ points
  /// to a CurveArray instance.
  /// The points are defined in millimetres,
  /// the returned CurveArray in feet.
  ///
  CurveArray CreateProfile( List<XYZ> pts )
  {
    CurveArray profile = new CurveArray();

    int n = _countour.Count;

    for( int i = 0; i < n; ++i )
    {
      int j = ( 0 == i ) ? n - 1 : i - 1;

      profile.Append( Line.CreateBound(
        MmToFootPoint( pts[j] ),
        MmToFootPoint( pts[i] ) ) );
    }
    return profile;
  }

  ///
  /// Create an extrusion from a given thickness
  /// and list of XYZ points defined in millimetres
  /// in the given family document, which  must
  /// contain a sketch plane named "Ref. Level".
  ///
  Extrusion CreateExtrusion(
    Document doc,
    List<XYZ> pts,
    double thickness )
  {
    Autodesk.Revit.Creation.FamilyItemFactory factory
      = doc.FamilyCreate;

    SketchPlane sketch = FindElement( doc,
      typeof( SketchPlane ), "Ref. Level" )
        as SketchPlane;

    CurveArrArray curveArrArray = new CurveArrArray();

    curveArrArray.Append( CreateProfile( pts ) );

    double extrusionHeight = MmToFoot( thickness );

    return factory.NewExtrusion( true,
      curveArrArray, sketch, extrusionHeight );
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;
    Document fdoc = null;
    Transaction t = null;

    if( null == doc )
    {
      message = "Please run this command in an open document.";
      return Result.Failed;
    }

    #region Create a new structural stiffener family

    // Check whether the family has already
    // been created or loaded previously.

    Family family
      = new FilteredElementCollector( doc )
        .OfClass( typeof( Family ) )
        .Cast<Family>()
        .FirstOrDefault<Family>( e
          => e.Name.Equals( _family_name ) );

    if( null != family )
    {
      fdoc = family.Document;
    }
    else
    {
      string templateFileName = Path.Combine( _path,
        _family_template_name + _family_template_ext );

      fdoc = app.NewFamilyDocument(
        templateFileName );

      if( null == fdoc )
      {
        message = "Cannot create family document.";
        return Result.Failed;
      }

      using( t = new Transaction( fdoc ) )
      {
        t.Start( "Create structural stiffener family" );

        CreateExtrusion( fdoc, _countour, _thicknessMm );

        t.Commit();
      }

      //fdoc.Title = _family_name; // read-only property

      bool needToSaveBeforeLoad = false;

      if( needToSaveBeforeLoad )
      {
        // Save our new family background document
        // and reopen it in the Revit user interface.

        string filename = Path.Combine(
          Path.GetTempPath(), _family_name + _rfa_ext );

        SaveAsOptions opt = new SaveAsOptions();
        opt.OverwriteExistingFile = true;

        fdoc.SaveAs( filename, opt );

        bool closeAndOpen = true;

        if( closeAndOpen )
        {
          // Cannot close the newly generated family file
          // if it is the only open document; that throws
          // an exception saying "The active document may
          // not be closed from the API."

          fdoc.Close( false );

          // This obviously invalidates the uidoc
          // instance on the previously open document.
          //uiapp.OpenAndActivateDocument( filename );
        }
      }
    }
    #endregion // Create a new structural stiffener family

    #region Load the structural stiffener family

    // Must be outside transaction; otherwise Revit
    // throws InvalidOperationException: The document
    // must not be modifiable before calling LoadFamily.
    // Any open transaction must be closed prior the call.

    // Calling this without a prior call to SaveAs
    // caused a "serious error" in Revit 2012:

    family = fdoc.LoadFamily( doc );

    // Workaround for Revit 2012,
    // no longer needed in Revit 2014:

    //doc.LoadFamily( filename, out family );

    // Setting the name requires an open
    // transaction, of course.

    //family.Name = _family_name;

    FamilySymbol symbol = null;

    foreach( ElementId id
      in family.GetFamilySymbolIds() )
    {
      // Our family only contains one
      // symbol, so pick it and leave.

      symbol = doc.GetElement( id ) as FamilySymbol;

      break;
    }
    #endregion // Load the structural stiffener family

    #region Insert stiffener family instance

    using( t = new Transaction( doc ) )
    {
      t.Start( "Insert structural stiffener family instance" );

      // Setting the name requires an open
      // transaction, of course.

      family.Name = _family_name;
      symbol.Name = _family_name;

      // Need to activate symbol before
      // using it in Revit 2016.

      symbol.Activate();

      bool useSimpleInsertionPoint = true;

      if( useSimpleInsertionPoint )
      {
        XYZ p = uidoc.Selection.PickPoint(
          "Please pick a point for family instance insertion" );

        StructuralType st = StructuralType.UnknownFraming;

        doc.Create.NewFamilyInstance( p, symbol, st );
      }

      bool useFaceReference = false;

      if( useFaceReference )
      {
        Reference r = uidoc.Selection.PickObject(
          ObjectType.Face,
          "Please pick a point on a face for family instance insertion" );

        Element e = doc.GetElement( r.ElementId );
        GeometryObject obj = e.GetGeometryObjectFromReference( r );
        PlanarFace face = obj as PlanarFace;

        if( null == face )
        {
          message = "Please select a point on a planar face.";
          t.RollBack();
          return Result.Failed;
        }
        else
        {
          XYZ p = r.GlobalPoint;
          //XYZ v = face.Normal.CrossProduct( XYZ.BasisZ ); // 2015
          XYZ v = face.FaceNormal.CrossProduct( XYZ.BasisZ ); // 2016
          if( v.IsZeroLength() )
          {
            v = face.FaceNormal.CrossProduct( XYZ.BasisX );
          }
          doc.Create.NewFamilyInstance( r, p, v, symbol );

          // This throws an exception saying that the face has no reference on it:
          //doc.Create.NewFamilyInstance( face, p, v, symbol );
        }
      }
      t.Commit();
    }
    #endregion // Insert stiffener family instance

    return Result.Succeeded;
  }
}
```

I hope this helps.