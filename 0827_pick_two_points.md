---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: code_example
optimization_date: '2025-12-11T11:44:14.682357'
original_url: https://thebuildingcoder.typepad.com/blog/0827_pick_two_points.html
post_number: 0827
reading_time_minutes: 4
series: geometry
slug: pick_two_points
source_file: 0827_pick_two_points.htm
tags:
- csharp
- elements
- python
- revit-api
- selection
- transactions
- windows
- geometry
title: Picking Two Points Consecutively
word_count: 720
---

﻿

### Picking Two Points Consecutively

Here is a question on picking points while interacting with a modeless dialogue:

**Question:** I want to pick two points consecutively using PickPoint.

I am using a modeless dialog, and have a section of code that is called when a button is pressed.

However, after picking the first point, an exception is thrown saying "The user aborted the pick operation".

Do you have a solution?

**Answer:** I implemented a sample application providing an external command PickTwoPoints for you to demonstrate one possible solution.

I use the following code to generate and display a modeless dialogue to the user before the picking is initiated:
```csharp
  Form DisplayModelessForm( IWin32Window owner )
  {
    System.Windows.Forms.Form form
      = new System.Windows.Forms.Form();
    form.Size = new Size( 300, 100 );
    form.Text = "Pick Two Points Modeless Form";
    form.Show( owner );
    return form;
  }
```

The form is initially displayed, then hidden again before calling PickPoint.

As always when displaying a modeless form, I define its owner window to be the Revit main frame.

I use my JtWindowHandle wrapper class to convert the IntPtr returned by the Autodesk.Windows.ComponentManager ApplicationWindow property to an IWin32Window, which can be passed in to the form Show method to set its owner:
```python
  /// <summary>
  /// Wrapper class for converting
  /// IntPtr to IWin32Window.
  /// </summary>
  public class JtWindowHandle : IWin32Window
  {
    IntPtr \_hwnd;

    public JtWindowHandle( IntPtr h )
    {
      Debug.Assert( IntPtr.Zero != h,
        "expected non-null window handle" );

      \_hwnd = h;
    }

    public IntPtr Handle
    {
      get
      {
        return \_hwnd;
      }
    }
  }
```

The external command Execute method performs the following steps:

- Determine the main Revit application window handle.- Display the dynamically generated modeless form.- Hide the form.- Select the two points.- Process the two resulting points, in this case by displaying them in the Visual Studio debug output window.- Redisplay the modeless form.- Close and terminate.

Here is the whole resulting Execute method mainline code implementing this:
```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  IWin32Window revit\_window
    = new JtWindowHandle(
      ComponentManager.ApplicationWindow );

  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Selection sel = uidoc.Selection;

  using( Form form = DisplayModelessForm(
    revit\_window ) )
  {
    form.Hide();

    try
    {
      XYZ p = sel.PickPoint( "Point 1" );

      XYZ q = sel.PickPoint( "Point 2" );

      Debug.Print( string.Format(
        "The two points you selected are {0} and {1}.",
        PointString( p ), PointString( q ) ) );
    }
    catch( Exception e )
    {
      Debug.Print( e.Message );
    }

    form.Show();
  }
  return Result.Succeeded;
}
```

As it now stands, this can obviously be packaged in an external command using read-only transaction mode, since the database is not affected in any way.

Here is
[PickTwoPoints.zip](zip/PickTwoPoints.zip) containing
the complete source code, entire Visual Studio solution and add-in manifest of the PickTwoPoints external command.

#### Automatically Store Revit Model as Blob in the Cloud

On the cloud and mobile front, Saikat Bhattacharya published an article describing his
[GbXmlCloudSync add-in](http://adndevblog.typepad.com/aec/2012/06/experiential-learning-gbxmlcloudsync-app.html) demonstrating
how to use the DocumentSaved event and the Windows Azure blob storage service to automatically upload each new version of a Revit model to the cloud, for instance for environmental analysis.

This is part of his
[Windows Azure experiential learning series](http://adndevblog.typepad.com/cloud_and_mobile/2012/06/experiential-learning-with-windows-azure-blogpost-series-on-the-aec-devblog.html).

#### Autodesk 360 Videos for MEP and Structure

Autodesk BIM 360 enables MEP and structural engineers to improve and extend their BIM workflows.

For MEP, this can provide insight into energy consumption, improve visualisation, photorealistic presentation, and CFD simulation.
Structural enhancement possibilities include access to simulation tools for more informed decisions, conducting computational heavy simulation earlier and more often, and optimising structural workflows with static and finite analysis tools.

This could be the first step of a move towards the next generation of BIM for anyone, anywhere, at any time, with access to intelligent, model-based workflows through a broad range of cloud-based services providing mobility, accessibility, and infinite computing power.

Here is a freshly published 1 minute and 20 seconds video on
[Autodesk BIM 360 for MEP engineers](http://www.youtube.com/embed/IrazkFnnWY0):

Here is an even more succinct 1 minute video on
[Autodesk BIM 360 for structural engineers](http://www.youtube.com/embed/Vlg7DlhITUQ):

According to some,
[start-ups rejoice](http://www.cloudtweaks.com/2012/09/autodesk-cloud-offering-to-slash-simulation-cost-startups-rejoice) at
the reduced simulation costs.