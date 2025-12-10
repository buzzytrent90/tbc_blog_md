---
post_number: "1010"
title: "Issue Using a Preview Control in a Macro"
slug: "preview_control_macro"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'transactions', 'views', 'windows']
source_file: "1010_preview_control_macro.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1010_preview_control_macro.html"
---

### Issue Using a Preview Control in a Macro

Here is a recent surprising little issue on using the Revit PreviewControl widget in the macro environment, brought up by Stephen Faust of
[Revolution Design](http://www.revolutiondesign.biz):

**Question:** I would like to use the preview control to display previews of parts of the model.

However, whenever I try to create a preview control it gives me the error "Revit does not support more than one preview control and there is already one active":

![Revit error](img/sf_revit_error.jpg)

I have searched online and I found the UI sample in the SDK as well as
[your class](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#2) on
this.

What I am trying to do to start off is just display the active view in a preview.
Looking at
[your sample](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#23) it
looks like it just feeds the active document and active view ID to the form and constructs from there.

I just have a WPF window and in the constructor it tries to create a preview control and add it to the children of the main grid with the following code:

```csharp
  gd\_Main.Children.Add(
    new PreviewControl(
      doc, doc.ActiveView.Id ) );
```

However this gives me the error above.
I'm not sure where mine is different in substance than yours, or where else there might be a preview control active...

What am I missing?

**Answer:** Congratulations on creating such a weird problem.

I have never heard of anything similar, nor would I have thought it possible.

You mention that you are aware of the UIAPI PreviewControl SDK and my simplified PreviewControlSimple samples.
Here are pointers to those from the discussion of using a
[preview control with linked document](http://thebuildingcoder.typepad.com/blog/2012/08/vacation-time-and-various-notes.html#4):

"One if the new
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html) features is the
[preview control](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2),
supporting enhanced integration between Revit and an add-in.
I presented a minimal sample named
[PreviewControlSimple](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#23) making
use of it in my DevCamp session on the
[Revit 2013 UI API enhancements](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#2),
and a more complete and complex one is given by the
[UIAPI Revit SDK sample](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4)."

I would suggest ensuring that you can successfully compile, execute and debug both of these, to ensure that your system is not causing the problem.

Are you sure you dispose properly of every preview control instance you created?

You might be creating one, not disposing it, and then trying to create another...

If the error persists in your add-in once that is done, I would suggest starting again from scratch, debugging regularly, and noting at what point the error occurs.

There is not much more to say.

After all, this is a one-liner, basically.

**Response:** I figured this out... ish.

I discovered that it seems to be a problem with creating it in a macro instead of an external command.

Some explanation:

I often test concepts with macros to see how things work or if something is going to work or not especially if it’s a pretty simple concept and doesn’t take a lot of setup. That way I don’t have to keep waiting for Revit to load when I need to debug something, etc. Anyway that is what I was doing here and no matter what I did I got that error.

I did compile the UIAPI command successfully and was able to run in properly so it’s not an issue with my machine or Revit install.

I was tried to figure out what differences in substance there are between the UIAPI example and what I had.
Just to test I created an external command in Visual Studio rather than in a macro.
The code is the same except that external commands have to return a result and apply attributes, etc., and the core is the same exact code.
When I run it in the external command DLL it works just as expected with no error...

So the culprit is the macro environment.

Here is a snapshot of the code in the macro:

```csharp
public void PreviewControl()
{
  using( Transaction t = new Transaction(
    ActiveUIDocument.Document ) )
  {
    t.Start( "temp" );

    Wnd\_Preview prv=new Wnd\_Preview(ActiveUIDocument);

    prv.ShowDialog();
    t.RollBack();
  }
}

public partial class Wnd\_Preview : Window
{
  public Wnd\_Preview( UIDocument Doc )
  {
    InitializeComponent();

    gd\_Main.Children.Add( new PreviewControl(
      Doc.Document, Doc.Document.ActiveView.Id ) );
  }
}
```

One more minor issue: what I ultimately want to do is create a new view so that I can play with the settings and display without changing the model around. I also want to be able to create the view so that if there is not already a 3D view in the project I can create it – I will then roll the transaction back so the view doesn’t stay in the users file.

I have a basic mock up of this working.
When I use an actual view already present in the project I can successfully use the view cube in the preview control.
However, when I create my own view, either by duplication or by View3D.CreateIsometric, it works fine, but the view cube is disabled.
I checked and the view is not showing as locked.
What could be causing this?

**Answer:** Hah!
I thought the problem might possibly be caused by something strange in the environment.
I am glad that the suggestion to start off by ensuring that the UIAPI sample works all right led in the right direction.

Regarding the disabled view cube:

You say that you are using the
[temporary transaction trick](http://thebuildingcoder.typepad.com/blog/2012/10/the-temporary-transaction-trick-for-gross-slab-data.html).

Sometimes, though, you need to either regenerate the document, or, even more, commit the transaction, to ensure that the current state actual comes into effect.

This led to Arnost's suggestion to
[encapsulate the transaction in a transaction group](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html).

That would enable you to commit your temporary transaction within a transaction group, ensuring that the changes take effect, and later discard the entire group, reverting the document back to its unchanged original state.

**Response:**

Thanks for the tip on transaction groups; I will give that a shot.

Many thanks to Steve for sharing this!