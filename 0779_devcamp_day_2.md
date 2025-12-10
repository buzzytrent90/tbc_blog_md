---
post_number: "0779"
title: "DevCamp Day Two"
slug: "devcamp_day_2"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'python', 'revit-api', 'selection', 'transactions', 'views', 'windows']
source_file: "0779_devcamp_day_2.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0779_devcamp_day_2.html"
---

### DevCamp Day Two

This morning I presented my second session, on some of the Revit 2013 UI API enhancements, in the Revit API beginner track.
After that, I returned back to the Revit API expert one to attend the presentations on modeless interaction and IFC before presenting my final session on the Revit MEP API.
Here then are my sessions of today, in chronological order:

- [1-4 Revit 2013 UI API enhancements](#2)- [2-3 Asynchronous interactions](#3)- [2-2 IFC export open source](#4)- [2-7 Revit MEP API](#5)

#### Revit 2013 UI API Enhancements

[Enhanced add-in integration](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2) is
an important topic in the Revit 2013 API.
This session discusses four of the add-in integration topics in more depth, building on a preceding presentation by Saikat Bhattacharya presenting a snapshot of the entire Revit UI API functionality:

- [Revit progress bar notifications](#21)- [Options dialogue WPF custom extensions](#22)- [Embed and control a Revit view](#23)- [Drag and drop](#24)

All four of these are covered by existing SDK samples, the first by
[ProgressNotifier](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#2) (Events),
the others by the comprehensive
[UIAPI](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4) sample.

For someone like me, feeling a strong affinity to
[Winnie the Pooh](http://en.wikipedia.org/wiki/Winnie-the-Pooh), the
"[bear of very little brain](http://www.winnie-pooh.org/winnie-the-pooh-quotes.htm)",
the existing SDK samples are way beyond simple comprehension and reuse, so I ended up creating four own extremely simple and minimal ones exercising these features in just a few lines of code each instead, to reduce the code and required audience comprehension power to an absolute minimum.

#### Revit progress bar notifications

To receive information about the current Revit progress bar status, you can subscribe to the ProgressChanged event.

The event handler will receive a ProgressChangedEventArgs instance providing the following properties:

- Caption – progress bar caption describing operation in progress- Stage – current stage of the progress bar, one of Started, CaptionChanged, RangeChanged, PositionChanged, UserCancelled, Finished- LowerRange – lower limit of range, always zero- UpperRange – upper limit of range, any non-zero number- Position – Value between zero and UpperRange incremented with "PositionChanged" stage

The ProgressNotifier SDK sample displays a WPF-based dialogue box.
It provides a button to open and load a Revit model.
Loading the selected file will cause a number of progress bar actions to execute.
These actions are monitored and presented in a stack structure, since occasionally more than one progress bar event may be active simultaneously:

![ProgressNotifier SDK sample](img/ProgressNotifier.png)

My new ProgressWatcher sample in much simpler and tracks all progress bar events by just printing the event argument data to the Visual Studio debug output window.
This can be achieved in a very few lines of code, basically only one each to subscribe to the event and to access and print the progress bar event data:
```python
[Transaction( TransactionMode.ReadOnly )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    Application app = uiapp.Application;

    app.ProgressChanged
      += new EventHandler<ProgressChangedEventArgs>(
        OnProgressChanged );

    return Result.Succeeded;
  }

  void OnProgressChanged(
    object sender,
    ProgressChangedEventArgs e )
  {
    double percent = 100.0 \* e.Position / e.UpperRange;

    Debug.Print(
      "'{0}' stage {1} position {2} [{3}, {4}] ({5}%)",
      e.Caption, e.Stage, e.Position, e.LowerRange,
      e.UpperRange, percent.ToString( "0.##" ) );
  }
}
```

Saving the output to a text file will allow you to analyse in detail the progress bar event sequence and nesting.

#### Options dialogue WPF custom extensions

In Revit 2013, an add-in can add its own custom tabs to the standard Revit Options dialogue.

This is achieved by defining a WPF control to display and subscribing to the UIApplication DisplayingOptionsDialog event.

The event handler receives an DisplayingOptionsDialogEventArgs instance.
It has a PagesCount property providing the number of Options dialogue tabs including default Revit ones.
It also sports the AddTab method that can be used to add a tab, providing a name and a handler for it.
The handler information is encapsulated in the TabbedDialogExtension class.

The TabbedDialogExtension class provides a constructor taking two arguments, a WPF user control instance and an OK handler.
These cannot be changed later.
It provides methods to get and set contextual help, and properties to read the WPF control and OK handler.
It also provides properties to both get and set the cancel and 'restore defaults' handlers:

- Control – the control- OnOKAction – the ok handler- OnCancelAction – the cancel handler- OnRestoreDefaultsAction – the restore defaults handler

The
[UIAPI](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4) SDK
sample adds several custom tabs demonstrating most available features.

I created a minimal sample named AddOptionsTab which defines the simplest possible WPF control with one single button to click:

![AddOptionsTab custom Options tab](img/AddOptionsTab.png)

If you click the button, the corresponding event handler on the form is called, which simply displays a task dialog:

![AddOptionsTab button clicked](img/AddOptionsTab_clicked.png)

If this tab has ever been activated after the Options dialogue was opened, its OK, cancel and 'restore defaults' handlers are called, depending on the user actions.

Here is the entire user control source code defining its handlers:
```csharp
public partial class UserControl1 : UserControl
{
  string \_name;

  public UserControl1( string name )
  {
    \_name = name;

    InitializeComponent();
  }

  private void button1\_Click(
    object sender,
    RoutedEventArgs e )
  {
    TaskDialog.Show( \_name, "I was clicked..." );
  }

  public void OnOK()
  {
    TaskDialog.Show( \_name, "OK" );
  }

  public void OnCancel()
  {
    TaskDialog.Show( \_name, "OnCancel" );
  }

  public void OnRestoreDefaults()
  {
    TaskDialog.Show( \_name, "OnRestoreDefaults" );
  }
}
```

The event handler code reacting to the Options dialogue displaying simply instantiates the user control, calls the TabbedDialogExtension constructor with it, and feeds that to the event handler arguments AddTab method:
```csharp
void OnDisplayingOptionsDialog(
  object sender,
  DisplayingOptionsDialogEventArgs e )
{
  UserControl1 c = new UserControl1(
    "DevCamp User Control" );

  e.AddTab( "DevCamp Custom Tab",
    new TabbedDialogExtension(
      c, c.OnOK ) );
}
```

The event handler is called each time the Options dialogue is invoked once the event has been subscribed to, which is achieved in a single line of code like this in the AddOptionsTab external command:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;

  uiapp.DisplayingOptionsDialog
    += new EventHandler<DisplayingOptionsDialogEventArgs>(
      OnDisplayingOptionsDialog );

  return Result.Succeeded;
}
```

#### Embed and control a Revit view

Another of the add-in integration enhancements enables you to embed and control a Revit view in your own form, dialogue or window.

The new PreviewControl class presents a preview control to browse the Revit model.
It takes two input arguments, a document and a view id.
The hosting form or window must be modal.
The View can be any graphical view, i.e. must be printable.
Perspective views are also supported, and all 3D Views can be manipulated by view cube.
The visibility and graphical settings of the view are effective and respected by the control.

The
[UIAPI](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4) SDK
sample proves a demonstration of this as well, which allows you to interactively select any of the open documents to view, or even open a new one, and secondly presents a list of all the documents views for you to switch between.
Each time you change any of these two selections, the current preview control is discarded and a new one is instantiated with the updated input arguments.
These input arguments are the only way in which the add-in affects the view, all further interaction happens between the control and the user.

To use a Revit preview control, you simply create a standard .NET form and insert a WPF host in it.
A suitable host is the System.Windows.Forms.Integration.ElementHost.
You populate this with a new Revit preview control instance, e.g. like this
```csharp
  elementHost.Child = new PreviewControl(
    doc, view.Id );
```

The PreviewControl instance needs to be disposed of after use:
```csharp
PreviewControl vc = elementHost.Child
as PreviewControl;
if( vc != null ) { vc.Dispose(); }
```

Here is a code snippet showing how all the printable views can be retrieved using a filtered element collector:
```csharp
  IEnumerable<View> views
    = new FilteredElementCollector( doc )
      .OfClass( typeof( View ) )
      .Cast<View>()
      .Where<View>( v => v.CanBePrinted );
```

I created a minimal sample application PreviewControlSimple for this DevCamp class with much less code (and functionality than the UIAPI one, since it just displays the current view of a document with no further ado.

It does add one interesting twist, though: to show exactly how the hosting form is set up, I implemented that twice over, first using the Visual Studio designer as usual, and then converting the form to be completely programmatically generated.
Here is the DisplayRevitView method taking the document and view input arguments for the preview control.
It also takes a third argument for the main Revit window, so that the generated control can be hooked up to that as a parent window.
This defines an appropriate relationship between the two as far as the Windows OS is concerned, ensuring correct minimisation and restoration behaviour:
```csharp
const string \_caption\_prefix
  = "Simple Revit Preview - ";

void DisplayRevitView(
  Document doc,
  View view,
  IWin32Window owner )
{

  using( PreviewControl pc
    = new PreviewControl( doc, view.Id ) )
  {

#if CREATE\_FORM\_BY\_CODE

    using( System.Windows.Forms.Form form
      = new System.Windows.Forms.Form() )
    {
      ElementHost elementHost = new ElementHost();

      elementHost.Location
        = new System.Drawing.Point( 0, 0 );

      elementHost.Dock = DockStyle.Fill;
      elementHost.TabIndex = 0;
      elementHost.Parent = form;
      elementHost.Child = pc;

      form.Text = \_caption\_prefix + view.Name;
      form.Controls.Add( elementHost );
      form.Size = new Size( 400, 400 );
      form.ShowDialog( owner );
    }

#else // if not CREATE\_FORM\_BY\_CODE

    Form1 form = new Form1( pc );
    form.ShowDialog( owner );

#endif // CREATE\_FORM\_BY\_CODE

  }
}
```

Please note that correct disposal of the preview control is ensured by the 'using' statement.
This is similar to Arnošt's strong recommendation to always
[encapsulate Revit Transactions in a 'using' statement](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html).

Calling this method from the external command mainline with the current document view is trivial.
I add one line of code to access the Revit main window:
```csharp
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
  Document doc = uidoc.Document;

  View view = doc.ActiveView;

  DisplayRevitView( doc, view, revit\_window );

  return Result.Succeeded;
}
```

#### Drag and Drop API

I recently discussed the new
[drag and drop functionality](http://thebuildingcoder.typepad.com/blog/2012/04/drag-and-drop-api.html) and
the UIAPI sample command demonstrating it.

The UIApplication provides the new static DoDragDrop method, with two overloads:

- DoDragDrop( ICollection<string> ) –
  initiate a standard built-in Revit drag and drop operation taking a collection of file names.- DoDragDrop( object, IDropHandler ) –
    initiate a drag and drop operation with a custom drop implementation.

This method is designed for use in a modeless form.

Please note that this is currently the one and only Revit API method not requiring a valid Revit API context.

This method can be called from your modeless form at any time, even without Revit giving you explicit permission to interact with the API via some form of callback or notification, as required for all other Revit API calls.

The behaviour of drag and drop given a list of filenames depends on the type of files, as I already recently pointed out:

- One AutoCAD format or image file dragged onto Revit
  – A new import placement editor is started to import the file.- More than one AutoCAD format or image files dragged
    – A new import placement editor is started for the first file.- One family file dragged onto Revit
      – The family is loaded, and an editor started to place an instance.- More than one family file dragged onto Revit
        – All the families are loaded.- More than one family file including other format files dragged
          – Revit tries to open all the files.- Valid file or list of files is passed
            – Revit does its best to use them appropriately.- Any files are not usable
              – Failure is signalled to user, no exception is thrown, add-in is not notified.

Here again, I implemented a new DevCamp sample add-in DragDropApi to be able to show the necessary steps in a simpler context than the corresponding UIAPI example.

Here are my four new Revit 2013 UI API samples
[AddOptionsTab, DragDropApi, PreviewControlSimple, ProgressWatcher](file:///C:/a/j/adn/devcamp/2012/doc/1-4_revit_2013_ui_api.zip) and
the
[slides](file:///C:/a/j/adn/devcamp/2012/doc/1-4_revit_2013_ui_api.pptx) I
used.

#### Asynchronous Interactions

Arnošt Löbel already presented on this topic at Autodesk University 2011 in his class
[CP5381](http://au.autodesk.com/?nd=event_class&session_id=9879&jid=1725932) 'Asynchronous Interactions and Managing Modeless UI with the Autodesk Revit API', as mentioned in the overview of the AU 2011
[Revit and AEC API sessions](http://thebuildingcoder.typepad.com/blog/2011/09/revit-and-aec-api-classes-at-autodesk-university.html).

In this class, Arnošt also covered the new
[external events](http://thebuildingcoder.typepad.com/blog/2012/04/idling-enhancements-and-external-events.html) and
demonstrated his
[ModelessForm\_ExternalEvent, ModelessForm\_IdlingEvent](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#1) (ModelessDialog)
and
[WorkThread](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#5) (MultiThreading)
SDK samples.

The conclusion of the presentation is the following pledge that we are all strongly recommended to take:

- I shall not call the API unless invoked by it- I shall not call the API from anywhere but the main thread- I shall call the API only, not directly the Revits UI- I shall use the Idling event cautiously and considerably

Here is a snapshot of the
[entire slide deck](zip/devcamp_2012_2-3_Asynchronous_Interactions.pdf).
This is not the final version, which will be posted soon after the end of DevCamp.

#### IFC Export Open Source

Angel Velez is senior principal engineer in the Revit development team and presented on the IFC exporter open source project, covering the following main topics:

- Introduction
  - History of IFC in Revit- Why open source?- How the code Works
    - Overall structure of code- Key points of interest, e.g. element traversal, property sets- Look at alternate UI

Here is the snapshot of his
[slides](zip/devcamp_2012_2-2_IFC_Open_Source.pptx).

#### Revit MEP API

The Revit MEP API is a recurring topic of mine at Autodesk University, and I presented on this once again last year in my session
[CP4453](http://au.autodesk.com/?nd=event_class&session_id=9267&jid=1727895) 'Everything
in Place with Autodesk Revit MEP Programming', as mentioned in the overview of the AU 2011
[Revit and AEC API sessions](http://thebuildingcoder.typepad.com/blog/2011/09/revit-and-aec-api-classes-at-autodesk-university.html).
The AU materials and the sample code that I use there was already
[published and discussed](http://thebuildingcoder.typepad.com/blog/2011/08/mep-sample-code-for-revit-2012.html).

In preparation for the sessions here in Waltham, I also discussed the
[Revit 2013 MEP API](http://thebuildingcoder.typepad.com/blog/2012/05/the-revit-2013-mep-api-and-external-services.html) and completed the
[migration of the ADN MEP sample code](http://thebuildingcoder.typepad.com/blog/2012/05/the-adn-mep-sample-adnrme-for-revit-mep-2013.html) to Revit 2013.

Here is the
[slide deck](file:///C:/a/j/adn/devcamp/2012/doc/2-8_estorage.pptx) I
used today, based on the AU one and updated for Revit MEP 2013.

Please note again that all the materials provided above are just snapshots.
Soon after the conference completes, the complete and final materials will be published.

#### Winding Down and Winding Up Again

In the evening we went out for a DevTech dinner, Japanese and Chinese cuisine.
It was great to hang out with the guys for a while and wind down a bit.

Next stop is the DevLab tomorrow with a record number of almost fifty registered participants, so we will be way busy!
Time to wind up again.
No rest for the wicked!
These events are always great, and I'm looking forward to it a lot.
```csharp
```