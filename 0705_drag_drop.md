---
post_number: "0705"
title: "Drag and Drop to Revit"
slug: "drag_drop"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'revit-api', 'views', 'windows']
source_file: "0705_drag_drop.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0705_drag_drop.html"
---

### Drag and Drop to Revit

Tim Hoffeller of
[CAD-Development Tim Hoffeller](http://www.cad-development.de)
presents DragDropRevitExample, a nice little stand-alone sample application which prompted us to document how to populate the suitable drag and drop data to cause Revit to load a family and prompt the user to place instances of it.

It has one limitation, unfortunately, or rather the Revit behaviour is currently limited in one respect: it only loads and prompts for instance placement if the family has not previously been loaded.
Not reloading an already loaded family obviously makes sense, but not repeating the prompt for instance placement when the same family is dragged and dropped repeatedly is a problem for actual production use.

In that case, of course, a Revit add-in could simply call the
[PromptForFamilyInstancePlacement](http://thebuildingcoder.typepad.com/blog/2010/06/place-family-instance.html) instead,
but this stand-alone drag and drop sample is an external Windows application which makes no use of the Revit API at all; it just populates the drag and drop data appropriately and initiates the drag and drop sequence.

Let's start with the good news, though, the initial question and solution:

**Question:** I'm trying to mimic the drag & drop from Explorer to load families inside of my custom control.

My application is a stand-alone Windows app, running outside of Revit.
Now I want to drag an item from a list view into Revit and load the family represented by this list view item.

In general the drag & drop seems to work fine, but Revit doesn't load the file.
What kind of data needs to be passed by the DoDragDrop method of my list view to force Revit to load the RFA file?

**Answer:** Revit expects a standard Explorer shell file drop, specifically CF\_HDROP, and processes it through CWnd::OnDropFiles() and DragQueryFile().
Therefore, you just need to construct a CF\_HDROP data object with a DROPFILES structure.

Here are some useful pointers for this:

- [Shell Clipboard Formats](http://msdn.microsoft.com/en-us/library/windows/desktop/bb776902(v=vs.85).aspx#CF_HDROP)- [DROPFILES](http://msdn.microsoft.com/en-us/library/windows/desktop/bb773269(v=vs.85).aspx)- [CWnd::OnDropFiles](http://msdn.microsoft.com/en-us/library/62zys01d(v=vs.80).aspx)- [DragQueryFile](http://msdn.microsoft.com/en-us/library/windows/desktop/bb776408(v=vs.85).aspx)

In addition, here is an untested CodeProject article and sample on
[how to implement Drag and Drop between your program and Explorer](http://www.codeproject.com/KB/shell/explorerdragdrop.aspx)
which might also be of interest.

**Question:** Thank you for the feedback.
Now I have a working version.
I have used the DataObject and passed it to my Listview DoDragDrop method.
Here is
<DragDropRevitExample.zip> containing
the entire source code and Visual Studio solution, but without the families.
You can place your own families into the Families subfolder provided in the project files.

This is its minimalistic user interface:

![DragDropRevitExample user interface](img/DragDropRevitExample_ui.png)

The application automatically populates the list view with all families found in a specific subdirectory.
They are listed in the ListView user interface and can be dragged and dropped into Revit to intiate the family load and family instance placement.

As said above, only the first drag and drop operation has any effect.
All following repetitions simply do nothing, since Revit determines that the family has already been loaded and decides that it has nothing further to do.

You possibly work around this limitation by implementing an additional Revit add-in that does catches this situation somehow and does something with the PromptForFamilyInstancePlacement method to achieve the same effect, if you like.

Here is the code of the MouseMove event handler which initiates the drag and drop operation and populates the DataObject appropriately:
```csharp
void \_listViewTestFamilies\_MouseMove(
  object sender,
  MouseEventArgs e )
{
  if( ( e.Button & MouseButtons.Left ) == MouseButtons.Left )
  {
    // Proceed with the drag-and-drop,
    // passing in the list item.

    FileInfo finf
      = (FileInfo) \_listViewTestFamilies.Items[
        indexOfItemUnderMouseToDrag].Tag;

    if( finf.Exists )
    {
      DataObject myDataObject = new DataObject();

      // You have to set up the FileDropList,
      // otherwise nothing will happen ;-)

      StringCollection coll
        = new StringCollection();

      coll.Add( finf.FullName );

      myDataObject.SetFileDropList( coll );

      DragDropEffects dropEffect
        = \_listViewTestFamilies.DoDragDrop(
          myDataObject, DragDropEffects.Copy );
    }
  }
}
```

Many thanks to Tim for exploring this issue and providing this nice little sample!