---
post_number: "1279"
title: "Sending Escape to Terminate a Family Instance Placement Loop"
slug: "terminate_inst_loop"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'python', 'references', 'revit-api', 'selection', 'transactions', 'views', 'windows']
source_file: "1279_terminate_inst_loop.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1279_terminate_inst_loop.html"
---

### Sending Escape to Terminate a Family Instance Placement Loop

I presented the
[PromptForFamilyInstancePlacement method](http://thebuildingcoder.typepad.com/blog/2010/06/place-family-instance.html) in
June 2010 when it was newly introduced in the Revit API, together with a nice little solution temporarily subscribing to the OnDocumentChanged to access the newly added elements after terminating the placement interaction.

Since then, the Revit 2015 API provided some
[Document API additions](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#4.02) such
as the method UIDocument.PostRequestForElementTypePlacement enabling an add-in to launch and to some extents control the full standard placement interaction between user and the Revit product.

To retain control over the operation and subsequent actions, however, the PromptForFamilyInstancePlacement method is still interesting as well.

One recent request was to limit it to place one single instance.

Luckily, a separate call to OnDocumentChanged is triggered by each family instance placement, enabling an add-in to terminate the placement loop programmatically in the middle of the user interaction loop.

This solution was suggested by Joachim Schmitz of
[liNear Gesellschaft für konstruktives Design mbH](http://www.linear.de) in
the following conversation:

**Question:**
In a special application case, we'd like to restrict the placement of family instances to one per call to PromptForFamilyInstancePlacement().

To do so, I have subscribed to the OnDocumentChanged event and am now trying to abort the placement operation after the addition of the first family instance to the model.

I can currently think of no better solution than to raise an Esc-Keydown event on the Revit MainWindow, but wasn't able to get that window object from its handle. Seems to be a dirty hack anyway.

Could you help me out with any suggestions here?

Any help highly appreciated!

**Answer:**
I can certainly help you determine a valid window handle for the Revit main window.

I have been using the JtWindowHandle helper class for years to determine the Revit window handle and reliably attach dialogue boxes and forms to it:

The most recent use of it was in the
[DirectObjLoader](https://github.com/jeremytammik/DirectObjLoader) add-in,
in the module
[JtWindowHandle.cs](https://github.com/jeremytammik/DirectObjLoader/blob/master/DirectObjLoader/JtWindowHandle.cs).

The code to make use of it looks like this at the beginning of the external command Execute method:

```csharp
IWin32Window revit\_window
= new JtWindowHandle(
ComponentManager.ApplicationWindow );
```

It requires a reference to AdWindows.dll from the Revit executable folder.

Yes, I agree that sending an Escape key hit sounds like a dirty hack.

Unfortunately, the Revit API currently provides no official method to achieve this.

**Response:**
After vainly trying to implement the escape key hack by raising a key down event on the Revit main window, I finally turned to a call to PostMessage.

It works like a charm now. I think that we can live with this solution until the next Revit API improvements to the PromptForFamilyInstancePlacement method are released.

Here is what I did:

```csharp
  private static void OnDocumentChanged(
    object sender,
    DocumentChangedEventArgs e )
  {
    addedInstances.AddRange( e.GetAddedElementIds() );
    if( addedInstances.Count >= 1 )
    {
      if( ComponentManager.ApplicationWindow != IntPtr.Zero )
      {
        WindowsMessaging.PostWindowsMessage(
          (int) ComponentManager.ApplicationWindow,
          WindowsMessaging.WM\_KEYDOWN,
          (int) Keys.Escape, 0 );

        WindowsMessaging.PostWindowsMessage(
          (int) ComponentManager.ApplicationWindow,
          WindowsMessaging.WM\_KEYDOWN,
          (int) Keys.Escape, 0 );
      }
    }
  }
  public static class WindowsMessaging
  {
    [DllImport( "User32.dll", EntryPoint = "SendMessage" )]
    public static extern int SendMessage(
      int hWnd, int Msg, int wParam, int lParam );

    [DllImport( "User32.dll", EntryPoint = "PostMessage" )]
    public static extern int PostMessage(
      int hWnd, int Msg, int wParam, int lParam );

    public const int WM\_KEYDOWN = 0x0100;

    public static int SendWindowsMessage(
      int hWnd, int Msg, int wParam, int lParam )
    {
      int result = 0;
      if( hWnd > 0 )
        result = SendMessage( hWnd, Msg, wParam, lParam );

      return result;
    }

    public static int PostWindowsMessage(
      int hWnd, int Msg, int wParam, int lParam )
    {
      int result = 0;
      if( hWnd > 0 )
        result = PostMessage( hWnd, Msg, wParam, lParam );

      return result;
    }
  }
```

**Answer:**
Congratulations on solving this!

Inspired by your solution, I implemented code to abort the PromptForFamilyInstancePlacement after placing the first instance in the existing external command implementation
[CmdPlaceFamilyInstance](http://thebuildingcoder.typepad.com/blog/2010/06/place-family-instance.html) in The Building Coder samples and posted the code to the
[The Building Coder sample GitHub repository](https://github.com/jeremytammik/the_building_coder_samples) in
[release 2015.0.117.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.117.1).

Here is my full implementation making use of the `Press` class implemented to
[launch a Revit command](http://thebuildingcoder.typepad.com/blog/2010/11/launching-a-revit-command.html) using WM\_KEYDOWN Windows messages similar to your sample code above:

```python
#region Namespaces
using System;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.UI;
using ComponentManager = Autodesk.Windows.ComponentManager;
using IWin32Window = System.Windows.Forms.IWin32Window;
using Keys = System.Windows.Forms.Keys;
#endregion // Namespaces
[Transaction( TransactionMode.Manual )]
class CmdPlaceFamilyInstance : IExternalCommand
{
  /// <summary>
  /// Set this flag to true to abort after
  /// placing the first instance.
  /// </summary>
  static bool \_place\_one\_single\_instance\_then\_abort
    = true;

  /// <summary>
  /// Send messages to main Revit application window.
  /// </summary>
  IWin32Window \_revit\_window;

  List<ElementId> \_added\_element\_ids
    = new List<ElementId>();

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    \_revit\_window
      = new JtWindowHandle(
        ComponentManager.ApplicationWindow );

    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    FilteredElementCollector collector
      = new FilteredElementCollector( doc );

    collector.OfCategory( BuiltInCategory.OST\_Doors );
    collector.OfClass( typeof( FamilySymbol ) );

    FamilySymbol symbol = collector.FirstElement()
      as FamilySymbol;

    \_added\_element\_ids.Clear();

    app.DocumentChanged
      += new EventHandler<DocumentChangedEventArgs>(
        OnDocumentChanged );

    uidoc.PromptForFamilyInstancePlacement( symbol );

    app.DocumentChanged
      -= new EventHandler<DocumentChangedEventArgs>(
        OnDocumentChanged );

    int n = \_added\_element\_ids.Count;

    TaskDialog.Show(
      "Place Family Instance",
      string.Format(
        "{0} element{1} added.", n,
        ( ( 1 == n ) ? "" : "s" ) ) );

    return Result.Succeeded;
  }

  void OnDocumentChanged(
    object sender,
    DocumentChangedEventArgs e )
  {
    ICollection<ElementId> idsAdded
      = e.GetAddedElementIds();

    int n = idsAdded.Count;

    Debug.Print( "{0} id{1} added.",
      n, Util.PluralSuffix( n ) );

    // this does not work, because the handler will
    // be called each time a new instance is added,
    // overwriting the previous ones recorded:

    //\_added\_element\_ids = e.GetAddedElementIds();

    \_added\_element\_ids.AddRange( idsAdded );

    if( \_place\_one\_single\_instance\_then\_abort
      && 0 < n )
    {
      // Why do we send the WM\_KEYDOWN message twice?
      // I tried sending it once only, and that does
      // not work. Maybe the proper thing to do would
      // be something like the Press.OneKey method...
      // nope, that did not work.

      //Press.OneKey( \_revit\_window.Handle,
      //  (char) Keys.Escape );

      Press.PostMessage( \_revit\_window.Handle,
        (uint) Press.KEYBOARD\_MSG.WM\_KEYDOWN,
        (uint) Keys.Escape, 0 );

      Press.PostMessage( \_revit\_window.Handle,
        (uint) Press.KEYBOARD\_MSG.WM\_KEYDOWN,
        (uint) Keys.Escape, 0 );
    }
  }
}
```

I don't know why one has to send the WM\_KEYDOWN message twice.

Whatever the reason, I can confirm that sending the WM\_KEYDOWN message twice works fine for me as well, and sending it once only does not. I also tested using the existing key-press method in The Building Coder samples instead, and that did not do the job either.

**Response:**
When you place instances with PromptForFamilyInstancePlacement, the previous one remains selected just until you drop the next one. The first esc key hit removes that selection while still allowing you to continue adding instances to the model. Only a second esc hit aborts the command. I was already used to this behavior from manually aborting the placement operation, even though I doubt that it serves any special purpose, since the user can’t do anything to that selected instance before the command is terminated. But it is probably perfectly normal for a newly added instance to automatically get selected in the first place, and obviously that selection has a higher priority in the message queue than the placement command. Hence the two esc hits.

Many thanks to Joachim for solving this task and sharing this approach!

#### MilanoJS Meetup

Today is the day of the
[Milano JS](http://milanojs.com) meetup and presentation of the
[View and Data API](https://developer-autodesk.github.io) at
Jobrapido, Via Pietro Paleocapa 7, 20121 Milano.

I will be using our new interactive online
[View and Data API slide deck](http://developer-autodesk.github.io/LmvQuickStartSlides),
implemented by my colleague Shiya Luo, who now also published the absolutely minimal
[View-and-Data-Barebone](https://github.com/Developer-Autodesk/View-and-Data-Barebone)
basic example of View and Data API client side scripting using the minimal JavaScript needed to get a viewer running, in only about 60 lines of code.

![Milano JS](img/milanojs.png)

Looking forward to seeing you at the meetup tonight, if you happen to be in the area!