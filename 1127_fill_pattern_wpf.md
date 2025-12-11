---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.354869'
original_url: https://thebuildingcoder.typepad.com/blog/1127_fill_pattern_wpf.html
post_number: '1127'
reading_time_minutes: 6
series: general
slug: fill_pattern_wpf
source_file: 1127_fill_pattern_wpf.htm
tags:
- csharp
- elements
- references
- revit-api
- sheets
- views
- windows
title: WPF Fill Pattern Viewer Control
word_count: 1110
---

### WPF Fill Pattern Viewer Control

Today, I present a useful stand-alone WPF control for viewing Revit fill patterns, originally implemented by Victor Checkalin, Виктор Чекалин, shared with us by Alexander Ignatovich, Александр Игнатович, of
[Investicionnaya Venchurnaya Companiya](http://www.iv-com.ru),
as part of further enhancements for the
[AddMaterial add-in](http://thebuildingcoder.typepad.com/blog/2014/03/adding-new-materials-from-list-updated-again.html).
The rest of his new enhancements will be discussed as soon as possible in a future post.

The AddMaterials add-in reads a list of material properties from an Excel spreadsheet and generates Revit material elements accordingly.

The input data includes columns defining the material surface and cut patterns.

These patterns are displayed graphically like this in the third and fourth columns of Alex' WPF control:

![AddMaterial WPF form](img/add_materials_ai_one.png)

Making use of the WPF fill pattern control in your own add-in is achieved in two simple steps:

- Add a copy of the XAML module FillPatternViewerControlWpf.xaml to your Visual Studio add-in project.
- Add references to the FillPatternViewerControlWpf control in your own XAML form source.

The fill pattern viewer XAML module appears like this in the solution explorer:

![FillPatternViewerControl XAML module](img/fill_pattern_wpf.png)

Here is the XAML code to define the surface and cut pattern columns in the form shown above:

```csharp
  <GridViewColumn Header="Surface"
                  Width="80">

    <GridViewColumn.CellTemplate>
      <DataTemplate DataType
          ="{x:Type ViewModel:MaterialViewModel}">

        <Border CornerRadius="2"
                BorderThickness="1"
                BorderBrush="Gray">

          <Controls:FillPatternViewerControlWpf
            HorizontalAlignment="Stretch"
            Height="30"
            Margin="1,3"
            FillPattern="{Binding SurfacePattern}"/>

        </Border>
      </DataTemplate>
    </GridViewColumn.CellTemplate>
  </GridViewColumn>

  <GridViewColumn Header="Cut"
                  Width="80">

    <GridViewColumn.CellTemplate>
      <DataTemplate DataType
          ="{x:Type ViewModel:MaterialViewModel}">

        <Border CornerRadius="2"
                BorderThickness="1"
                BorderBrush="Gray">

          <Controls:FillPatternViewerControlWpf
            HorizontalAlignment="Stretch"
            Height="30"
            Margin="1,3"
            FillPattern="{Binding CutPattern}"/>

        </Border>
      </DataTemplate>
    </GridViewColumn.CellTemplate>
  </GridViewColumn>
```

The fill pattern viewer control is defined by its XAML description plus the C# code defining its associated behaviour.

The XAML description looks like this:

```csharp
<UserControl
  x:Class="AddMaterials.View.Controls.FillPatternViewerControlWpf"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
  xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
  xmlns:Converters="clr-namespace:AddMaterials.View.Converters"
  mc:Ignorable="d"
  d:DesignHeight="300"
  d:DesignWidth="300">

  <Grid Name="\_grid"
        ToolTip="{Binding Path=FillPattern.Name,
          RelativeSource={RelativeSource
            AncestorType={x:Type UserControl}}}">
    <Grid.Resources>
      <Converters:BitmapToImageSourceConverter
        x:Key="BitmapToImageSourceConverter"/>
    </Grid.Resources>
    <Image Source="{Binding Path=FillPatternImage,
             RelativeSource={RelativeSource
             AncestorType={x:Type UserControl}},
             Converter={StaticResource BitmapToImageSourceConverter}}"
           Stretch="None"/>

  </Grid>
</UserControl>
```

The C# code defining its behaviour is where the real action takes place, in the DrawFillPattern method, which retrieves and displays the graphical data defined by the Revit FillGrid instances managed by the FillPattern object:

```csharp
using System;
using System.ComponentModel;
using System.Diagnostics;
using System.Drawing;
using System.Linq;
using System.Windows;
using Autodesk.Revit.DB;
using Image = System.Drawing.Image;
using Matrix = System.Drawing.Drawing2D.Matrix;
using Rectangle = System.Drawing.Rectangle;

namespace AddMaterials.View.Controls
{
  /// <summary>
  /// Interaction logic for FillPatternViewerControlWpf.xaml
  /// </summary>
  public partial class FillPatternViewerControlWpf
    : INotifyPropertyChanged
  {
    private const float Scale = 50;
    private Bitmap fillPatternImg;

    public static readonly DependencyProperty
      FillPatternProperty = DependencyProperty
        .RegisterAttached( "FillPattern",
          typeof( FillPattern ),
          typeof( FillPatternViewerControlWpf ),
          new UIPropertyMetadata( null,
            OnFillPatternChanged ) );

    private static void OnFillPatternChanged(
      DependencyObject d,
      DependencyPropertyChangedEventArgs e )
    {
      var fillPatternViewerControl
        = d as FillPatternViewerControlWpf;

      if( fillPatternViewerControl == null ) return;

      fillPatternViewerControl.OnPropertyChanged(
        "FillPattern" );

      fillPatternViewerControl.CreateFillPatternImage();
    }

    public FillPattern FillPattern
    {
      get
      {
        return (FillPattern) GetValue(
          FillPatternProperty );
      }
      set
      {
        SetValue( FillPatternProperty, value );
      }
    }

    public FillPattern GetFillPattern(
      DependencyObject obj )
    {
      return (FillPattern) obj.GetValue(
        FillPatternProperty );
    }

    public void SetFillPattern(
      DependencyObject obj,
      FillPattern value )
    {
      obj.SetValue( FillPatternProperty, value );
    }

    public FillPatternViewerControlWpf()
    {
      InitializeComponent();
    }

    public event PropertyChangedEventHandler
      PropertyChanged;

    //[NotifyPropertyChangedInvocator]
    protected virtual void OnPropertyChanged(
      string propertyName )
    {
      var handler = PropertyChanged;
      if( handler != null ) handler( this,
        new PropertyChangedEventArgs( propertyName ) );
    }

    public Image FillPatternImage
    {
      get
      {
        if( fillPatternImg == null )
          CreateFillPatternImage();
        return fillPatternImg;
      }
    }

    private void CreateFillPatternImage()
    {
      if( FillPattern == null )
        return;

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

      fillPatternImg = new Bitmap(
        (int) width, (int) height );

      using( var g = Graphics.FromImage(
        fillPatternImg ) )
      {
        DrawFillPattern( g );
      }

      OnPropertyChanged( "FillPatternImage" );
    }

    private void DrawFillPattern( Graphics g )
    {
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
      }
      catch( Exception ex )
      {
        Debug.Print( ex.Message );
      }
    }

    private double RadianToGradus( double radian )
    {
      return radian \* 180 / Math.PI;
    }
  }
}
```

The source code, Visual Studio solution and add-in manifest of the complete AddMaterials add-in including the fill pattern viewer WPF control is provided in the
[AddMaterials GitHub repository](https://github.com/jeremytammik/AddMaterials).

The version discussed above is
[release 2014.0.0.2](https://github.com/jeremytammik/AddMaterials/releases/tag/2014.0.0.2).

Many thanks to Victor for the implementation and to Alex for sharing this valuable tool!