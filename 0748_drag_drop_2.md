---
post_number: "0748"
title: "Drag and Drop API"
slug: "drag_drop_2"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'revit-api', 'views', 'windows']
source_file: "0748_drag_drop_2.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0748_drag_drop_2.html"
---

### Drag and Drop API

We already looked at what can be done using
[drag and drop in Revit 2012](http://thebuildingcoder.typepad.com/blog/2012/01/drag-and-drop-to-revit.html).

At that point, an external Windows application could be used to initiate a standard Revit drag and drop sequence, but we had no control over Revit's behaviour on receiving the dropped files.

This has changed in the
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html),
which provides a drag and drop API as part of the
[add-in integration](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2) features
and includes the
[UIAPI](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4) sample
in the Revit SDK to demonstrate its use, among several other new API features such as discipline dependent application availability, preview control, and a custom WPF options dialogue.

By the way, the UIAPI sample differs from many other SDK samples in a couple of points:

- It is an external application, so you cannot use RvtSamples to load and run it.
  You have to install it separately.- It is still rather undocumented in the Revit 2013 RP version of the SDK, in that it lacks the customary readme file and is not listed in SamplesReadMe.htm like the others.

To make up for that, here is a pretty detailed description of its drag and drop functionality, at least.

#### UIAPI SDK Sample Drag and Drop

The UIAPI drag and drop command displays the following modeless dialogue:

![UIAPI DragAndDrop form](img/drag_drop_form.png)

It presents a list of loaded furniture family symbols on the left and all furniture family files found by recursively searching the content library on the right.

#### UIApplication DoDragDrop Method Overloads

The UIApplication now provides two overloads of the new static method DoDragDrop for interacting with drag and drop events:

- [DoDragDrop( ICollection<string> )](#3) to initiate a standard Revit drag and drop operation of a collection of file names on the Revit user interface.- [DoDragDrop( object, IDropHandler )](#5) to initiate a drag and drop operation with a custom drop implementation.

This method and its same two overloads are also provided on the macro-specific ApplicationEntryPoint class, for Revit macro use only.

#### Drag and Drop a List of Files

In the first overload, the method argument holds a list of paths and names of files to be dropped on the Revit user interface, which causes the following default behaviour:

- Only one AutoCAD format or image file dragged onto Revit: a new import placement editor will be started to import the file.- More than one AutoCAD format or image files dragged onto Revit: a new import placement editor will be started only for the first AutoCAD format or image file.- Only one family file dragged onto Revit: the family will be loaded, and an editor will be started to place the family.- More than one family file dragged onto Revit: all the families will be loaded.- More than one family file including other format files dragged onto Revit: Revit will try to open all the files.- If a valid file or list of files is passed, Revit will do its best to use them appropriately. If any files are not usable, failure will be signalled to the interactive Revit user, but **no** exception is thrown and the add-in will not be notified.

Here again, this behaviour is built into Revit and cannot be modified.

The code initiating the drag and drop from the modeless dialogue is as simple as this:
```csharp
  // Drag action from list box
  private void listBox1\_MouseMove(
    object sender,
    MouseEventArgs e )
  {
    if( System.Windows.Forms.Control.MouseButtons
      == MouseButtons.Left )
    {
      FamilyListBoxMember member
        = (FamilyListBoxMember) listBox1.SelectedItem;

      // Use standard Revit drag and drop behavior

      List<String> data = new List<String>();
      data.Add( member.FullPath );
      UIApplication.DoDragDrop( data );
    }
  }
```

In other words, dragging a file name from the list on the right onto Revit will trigger the same built-in default behaviour as dragging the same file from the Windows explorer.

#### Yet another Caveat against Invoking Revit API Methods Modelessly

Note that both this method and the one discussed below call the static UIApplication DoDragDrop method from a modeless dialogue, i.e. outside a valid Revit API context, which is unusual for the Revit API.
This is most likely going to be changed in the sample, as it will almost certainly cause serious problems such as unexpected results, corrupted documents, crashes, etc. if utilised in a commercial application.
A better approach would be to use the Idling or an external event to trigger the DragDrop activity at a later time.

The only safe option is to call DoDragDrop from an Idling or external event call-back, just like all other Revit API calls.

Actually, on further discussion, we realised that the situation is even worse: invoking the drag-and-drop method during both Idling and external events is never recommendable, because it is likely to cause
[deadlock](http://en.wikipedia.org/wiki/Deadlock).

The problem is that the user interface is not expecting to be invoked during idling; the application is idling for the exact reason that there is no UI activity.
If UI operations are invoked in this situation, two things may happen:

- The same message pump is re-entered with a new request, which cannot be processed, since the pump is still waiting for the idling event to be ended.- Another, secondary message pump is entered, which may eventually invoke another idling event, which may dead-lock the client application.

Unfortunately, this means that there really is no totally safe way to utilise this wonderful new API from a modeless dialogue.
We will obviously be taking a new look at this in future versions, but for now it is important to realise the risks of invoking the Revit UI during the Idling event.

Here are some ideas which may help relativate these worries:

- The problem of dead-lock or re-entering idling will only be apparent if invoking Revit UI, for instance the interactive placement of a family instance
  If the method only loads the family or does some other non-UI activity, it will be OK.- The tool works perfectly well if there is no other Revit tool active.
    Of course, the best way to tell this is from Idling, which it seems cannot successfully call the call-back.

So once again: make sure that you test the scenario in which you plan to use this very carefully, and avoid all risks in a commercial implementation.

#### Drag and Drop with a Custom Handler

The second overload is much more exciting, because it allows us to define our own drop behaviour inside Revit.

This can only be used to define behaviour within the context of the add-in's own UI, though.
For example, it does not allow you to define new drag and drop behaviour for files which are not supported by the method above when dropped onto Revit from explorer, or from an unconnected application like Excel, AutoCAD, etc.
The drag operation must be initiated from a point where it can call DoDragDrop to Revit to allow it to complete.

It requires us to set up a handler for the drop event, which is derived from the IDropHandler interface, requiring then implementation of one single method, Execute.

In this sample, the drop handler expects the element id of a family symbol to be passed in and simply calls the PromptForFamilyInstancePlacement method on the symbol.

You are obviously completely free to pass in any data you like when invoking the handler, and can choose to do something completely different with it inside Revit on receipt:
```csharp
  /// <summary>
  /// Custom handler for placement of loaded family types
  /// </summary>
  public class LoadedFamilyDropHandler : IDropHandler
  {
    public void Execute(
      UIDocument doc,
      object data )
    {
      ElementId familySymbolId = (ElementId) data;

      FamilySymbol symbol = doc.Document.GetElement(
        familySymbolId ) as FamilySymbol;

      if( symbol != null )
      {
        doc.PromptForFamilyInstancePlacement(
          symbol );
      }
    }
  }
```

With the drop handler in place, we can initiate a custom drag and drop like this:
```csharp
  // Drag action from list view
  private void listView\_MouseMove(
    object sender,
    MouseEventArgs e )
  {
    if( System.Windows.Forms.Control.MouseButtons
      == MouseButtons.Left )
    {
      ListViewItem selectedItem = this.listView1
        .SelectedItems.Cast<ListViewItem>()
        .FirstOrDefault<ListViewItem>();

      if( selectedItem != null )
      {
        // Use custom Revit drag and drop behavior

        LoadedFamilyDropHandler myhandler
          = new LoadedFamilyDropHandler();

        UIApplication.DoDragDrop(
          selectedItem.Tag, myhandler );
      }
    }
  }
```

Be sure to take a look at the other commands defined by the UIAPI SDK sample as well, because they are all pretty exciting and address a number of long-standing wish list items.