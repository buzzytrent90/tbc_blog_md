---
post_number: "1006"
title: "Setting a Default 3D View Orientation"
slug: "3d_view_orientation"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'python', 'revit-api', 'sheets', 'views']
source_file: "1006_3d_view_orientation.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1006_3d_view_orientation.html"
---

### Setting a Default 3D View Orientation

Here is a nice little explanation by Alexander Ignatovich of
[Investicionnaya Venchurnaya Companiya](http://www.iv-com.ru) (that
sounds like a venture investment company to me) on how to export an image file for a family or project.

One issue that cropped up was how to set the default view orientation for a newly created 3D view:

**Question:** In versions prior to Revit 2014, I used this code to create a new 3D view with a default view direction:

```csharp
  var direction = new XYZ( -1, 1, -1 );
  var view3D = doc.IsFamilyDocument
    ? doc.FamilyCreate.NewView3D( direction )
    : doc.Create.NewView3D( direction );
```

I am having difficulty obtaining the same result in Revit 2014, though, using the View3D CreateIsometric and SetOrientation methods.

I tried the following, but with no success:

```csharp
  var direction = new XYZ(-1, 1, -1);
  var collector = new FilteredElementCollector(doc);
  var viewFamilyType = collector
    .OfClass<ViewFamilyType>()
    .Cast<ViewFamilyType>()
    .FirstOrDefault(x => x.ViewFamily
      == ViewFamily.ThreeDimensional);

  // . . .

  var view3D = View3D.CreateIsometric(
    doc, viewFamilyType.Id);

  // . . .

  view3D.SetOrientation( new ViewOrientation3D(
    direction, new XYZ(0, 1, 1), new XYZ(0, 1, -1)));
```

The result differs from the old obsolete code.

What parameters should I use to get the same result?

**Answer:** I solved my issue trying to generate pictures of families and project documents looking like the default 3D views in Revit.

I think I found the simplest way to do this, and maybe it will be useful not only for me.

When I initially tried to convert my code to the new way, I called the method

```csharp
  view3D.SetOrientation(
    new ViewOrientation3D(
      direction,
      new XYZ( 0, 1, 1 ),
      new XYZ( 0, 1, -1 ) ) );
```

I just removed the call to invoke the SetOrientation method and it now works perfectly.

It generates very nice pictures :-)
Here are two of them:

![Sample image](img/ai_3d_view_orient_1.png)

![Sample image](img/ai_3d_view_orient_2.jpg)

In my code, I make use of the following filtered element collector extension methods:

```python
public static class FilteredElementCollectorExtensions
{
  public static FilteredElementCollector OfClass<T>(
    this FilteredElementCollector collector )
      where T : Element
  {
    return collector.OfClass( typeof( T ) );
  }

  public static IEnumerable<T> OfType<T>(
    this FilteredElementCollector collector )
      where T : Element
  {
    return Enumerable.OfType<T>(
      collector.OfClass<T>() );
  }
}
```

Then I can generate the views using the following:

```csharp
static string ExportToImage( Document doc )
{
  var tempFileName = Path.ChangeExtension(
    Path.GetRandomFileName(), "png" );

  string tempImageFile;

  try
  {
    tempImageFile = Path.Combine(
      Path.GetTempPath(), tempFileName );
  }
  catch( IOException )
  {
    return null;
  }

  IList<ElementId> views = new List<ElementId>();

  try
  {

#if !VERSION2014
    var direction = new XYZ(-1, 1, -1);
    var view3D = doc.IsFamilyDocument
      ? doc.FamilyCreate.NewView3D(direction)
      : doc.Create.NewView3D(direction);
#else
    var collector = new FilteredElementCollector(
      doc );

    var viewFamilyType = collector
      .OfClass( typeof( ViewFamilyType ) )
      .OfType<ViewFamilyType>()
      .FirstOrDefault( x =>
        x.ViewFamily == ViewFamily.ThreeDimensional );

    var view3D = ( viewFamilyType != null )
      ? View3D.CreateIsometric( doc, viewFamilyType.Id )
      : null;

#endif // VERSION2014

    if( view3D != null )
    {
      views.Add( view3D.Id );

      var graphicDisplayOptions
        = view3D.get\_Parameter(
          BuiltInParameter.MODEL\_GRAPHICS\_STYLE );

      // Settings for best quality

      graphicDisplayOptions.Set( 6 );
    }
  }
  catch( Autodesk.Revit.Exceptions
    .InvalidOperationException )
  {
  }

  var ieo = new ImageExportOptions
  {
    FilePath = tempImageFile,
    FitDirection = FitDirectionType.Horizontal,
    HLRandWFViewsFileType = ImageFileType.PNG,
    ImageResolution = ImageResolution.DPI\_150,
    ShouldCreateWebSite = false
  };

  if( views.Count > 0 )
  {
    ieo.SetViewsAndSheets( views );
    ieo.ExportRange = ExportRange.SetOfViews;
  }
  else
  {
    ieo.ExportRange = ExportRange
      .VisibleRegionOfCurrentView;
  }

  ieo.ZoomType = ZoomFitType.FitToPage;
  ieo.ViewName = "tmp";

  if( ImageExportOptions.IsValidFileName(
    tempImageFile ) )
  {
    // If ExportRange = ExportRange.SetOfViews
    // and document is not active, then image
    // exports successfully, but throws
    // Autodesk.Revit.Exceptions.InternalException

    try
    {
      doc.ExportImage( ieo );
    }
    catch
    {
      return string.Empty;
    }
  }
  else
  {
    return string.Empty;
  }

  // File name has format like
  // "tempFileName - view type - view name", e.g.
  // "luccwjkz - 3D View - {3D}.png".
  // Get the first image (we only listed one view
  // in views).

  var files = Directory.GetFiles(
    Path.GetTempPath(),
    string.Format( "{0}\*.\*", Path
      .GetFileNameWithoutExtension(
        tempFileName ) ) );

  return files.Length > 0
    ? files[0]
    : string.Empty;
}
```

Many thanks to Alexander for sharing this!

#### Addendum – ImageExportOptions.GetFileName

Maxence points out in his [comment](#comment-4620342466) below:

> Since Revit 2015 you can use the static method `ImageExportOptions.GetFileName()` to get the file name (without path and without extension) that will be produced when exporting a view to an image.