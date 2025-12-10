---
post_number: "1140"
title: "WPF Fill Pattern Viewer Control Benchmark"
slug: "fill_pattern_benchmark"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'revit-api', 'rooms', 'transactions', 'views']
source_file: "1140_fill_pattern_benchmark.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1140_fill_pattern_benchmark.html"
---

### WPF Fill Pattern Viewer Control Benchmark

I recently presented
[Victor Chekalin](http://www.facebook.com/profile.php?id=100003616852588)'s
[WPF control for viewing Revit fill patterns](http://thebuildingcoder.typepad.com/blog/2014/04/wpf-fill-pattern-viewer-control.html).

In his
[comment](http://thebuildingcoder.typepad.com/blog/2014/04/wpf-fill-pattern-viewer-control.html#comment-6a00e553e16897883301a5119e101d970c),
Victor already mentioned some worries about performance of this control and his intention to create a simple benchmark to monitor it.

Well, here it is.

At the same time, I also took the opportunity to migrate the fill pattern control to Revit 2015.

#### Migrate the Fill Pattern Viewer Control to Revit 2015

The migration to the Revit 2015 API was extremely straightforward, just like the previous add-ins I performed it for, e.g.:

- [RevitLookup for Revit 2015](http://thebuildingcoder.typepad.com/blog/2014/04/revitlookup-for-revit-2015.html)
- [Compiling the Revit 2015 SDK and Migrating The Building Coder Samples](http://thebuildingcoder.typepad.com/blog/2014/04/compiling-the-revit-2015-sdk-and-migrating-bc-samples.html)
- [Migrating RoomEditorApp to Revit 2015](http://thebuildingcoder.typepad.com/blog/2014/04/migrating-roomeditorapp-to-revit-2015.html)
- [RevitLookup for Revit 2015 UR1](http://thebuildingcoder.typepad.com/blog/2014/04/revitlookup-for-ur1-adn-aec-and-au-news.html)

The source code, Visual Studio solution and add-in manifest of the complete AddMaterials add-in including the fill pattern viewer WPF control is provided in the
[AddMaterials GitHub repository](https://github.com/jeremytammik/AddMaterials),
and the initial migration to the Revit 2015 API is captured in
[release 2015.0.0.2](https://github.com/jeremytammik/AddMaterials/releases/tag/2015.0.0.2).

#### Fill Pattern Viewer Control Benchmark

In Victor's own words:

> I was interested about benchmark and created it:)
>
> I created the new command 'AddMaterials - Fill Pattern benchmark' and added it to the AddMaterials Add-In.
>
> As it turned out, the most complex pattern is 'Concrete'. To create the graphics for it took more that one second.

Displaying any other fill pattern in this control requires only about 10 milliseconds.

The benchmark implementation is accomplished in a new external command, FillPatternBenchmarkCommand, which retrieves all fill patterns from the model and displays them in a form:

```csharp
  [Transaction( TransactionMode.ReadOnly )]
  public class FillPatternBenchmarkCommand
    : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      var doc = commandData.Application
        .ActiveUIDocument.Document;

      var fillPatternElements
        = new FilteredElementCollector( doc )
          .OfClass( typeof( FillPatternElement ) )
          .OfType<FillPatternElement>()
          .OrderBy( fp => fp.Name )
          .ToList();

      var fillPatterns
        = fillPatternElements.Select(
          fpe => fpe.GetFillPattern() );

      FillPatternsViewModel fillPatternsViewModel
        = new FillPatternsViewModel( fillPatterns
          .Select( x => new FillPatternViewModel(
            x ) ) );

      FillPatternsView fillPatternsView
        = new FillPatternsView()
      {
        DataContext = fillPatternsViewModel
      };

      fillPatternsView.ShowDialog();

      return Result.Succeeded;
    }
  }
```

The form requires two new ViewModel modules to view a simple fill pattern, a list of all fill patterns, and present that list in a WPF form.

The actual benchmarking is performed by adding a couple of lines to the existing FillPatternViewerControlWpf.xaml.cs module.

The new code instantiates a stopwatch.

In debug mode only, the stopwatch elapsed time in milliseconds is displayed in red on top of the fill pattern:

```csharp
  private void DrawFillPattern( Graphics g )
  {
    Stopwatch sw = Stopwatch.StartNew();

    float matrixScale;

    var fillPattern = FillPattern;

    if( fillPattern == null )
      return;

    if( fillPattern.Target == FillPatternTarget.Model )
      matrixScale = Scale;
    else
      matrixScale = Scale \* 10;

    try
    {
      var width =
      ( ActualWidth == 0 ? Width : ActualWidth ) == 0
        ? 100
        : ( ActualWidth == 0 ? Width : ActualWidth );

      if( double.IsNaN( width ) )
        width = 100;

      var height =
          ( ActualHeight == 0 ? Height : ActualHeight ) == 0
            ? 30
            : ( ActualHeight == 0 ? Height : ActualHeight );

      if( double.IsNaN( height ) )
        height = 30;

      var rect = new Rectangle( 0, 0,
        (int) width, (int) height );

      var centerX = ( rect.Left + rect.Left
        + rect.Width ) / 2;

      var centerY = ( rect.Top + rect.Top
        + rect.Height ) / 2;

      g.TranslateTransform( centerX, centerY );

      var rectF = new Rectangle( -1, -1, 2, 2 );

      g.FillRectangle( Brushes.Blue, rectF );

      g.ResetTransform();

      var fillGrids = fillPattern.GetFillGrids();

      Debug.Print( "FilPattern name: {0}",
        fillPattern.Name );

      Debug.Print( "Grids count: {0}",
        fillGrids.Count );

      foreach( var fillGrid in fillGrids )
      {
        var degreeAngle = (float) RadianToGradus(
          fillGrid.Angle );

        Debug.Print( new string( '-', 100 ) );

        Debug.Print( "\tOrigin: U: {0} V:{1}",
          fillGrid.Origin.U, fillGrid.Origin.V );

        Debug.Print( "\tOffset: {0}", fillGrid.Offset );
        Debug.Print( "\tAngle: {0}", degreeAngle );
        Debug.Print( "\tShift: {0}", fillGrid.Shift );

        var pen = new Pen( System.Drawing.Color.Black )
        {
          Width = 1f / matrixScale
        };

        float dashLength = 1;

        var segments = fillGrid.GetSegments();

        if( segments.Count > 0 )
        {
          pen.DashPattern = segments
              .Select( Convert.ToSingle )
              .ToArray();

          Debug.Print( "\tSegments:" );

          foreach( var segment in segments )
          {
            Debug.Print( "\t\t{0}", segment );
          }

          dashLength = pen.DashPattern.Sum();
        }

        g.ResetTransform();

        var rotateMatrix = new Matrix();
        rotateMatrix.Rotate( degreeAngle );

        var matrix = new Matrix( 1, 0, 0, -1,
          centerX, centerY );

        matrix.Scale( matrixScale, matrixScale );

        matrix.Translate( (float) fillGrid.Origin.U,
          (float) fillGrid.Origin.V );

        var backMatrix = matrix.Clone();
        backMatrix.Multiply( rotateMatrix );
        matrix.Multiply( rotateMatrix );

        bool first = true;
        for( int i = 20; i > 0; i-- )
        {
          if( !first )
          {
            matrix.Translate( (float) fillGrid.Shift,
              (float) fillGrid.Offset );

            backMatrix.Translate( (float) fillGrid.Shift,
              -(float) fillGrid.Offset );
          }
          else
          {
            first = false;
          }

          var offset = ( -10 ) \* dashLength;
          matrix.Translate( offset, 0 );
          backMatrix.Translate( offset, 0 );

          g.Transform = matrix;

          g.DrawLine( pen, new PointF( 0, 0 ),
            new PointF( 200, 0 ) );

          g.Transform = backMatrix;

          g.DrawLine( pen, new PointF( 0, 0 ),
            new PointF( 200, 0 ) );
        }
      }

      sw.Stop();
      g.ResetTransform();

#if DEBUG
      g.DrawString( string.Format( "{0} ms",
        sw.ElapsedMilliseconds),
        System.Drawing.SystemFonts.DefaultFont,
        Brushes.Red, 0, 0);
#endif

    }
    catch( Exception ex )
    {
      Debug.Print( ex.Message );
    }
  }
```

The result of running the benchmark command in Revit 2015 looks like this:

![AddMaterial fill pattern viewer benchmark](img/fill_pattern_viewer_benchmark.png)

The benchmarking code is added to the
[AddMaterials GitHub repository](https://github.com/jeremytammik/AddMaterials) and
captured in
[release 2015.0.0.3](https://github.com/jeremytammik/AddMaterials/releases/tag/2015.0.0.3).

Many thanks to Victor for the initial implementation and again for this benchmark implementation!