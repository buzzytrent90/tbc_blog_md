---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.3
content_type: code_example
optimization_date: '2025-12-11T11:44:13.340601'
original_url: https://thebuildingcoder.typepad.com/blog/0088_revit_window_handle.html
post_number: 0088
reading_time_minutes: 8
series: general
slug: revit_window_handle
source_file: 0088_revit_window_handle.htm
tags:
- csharp
- elements
- levels
- python
- revit-api
- rooms
- selection
- walls
- windows
title: Revit Window Handle and Modeless Dialogues
word_count: 1581
---

### Revit Window Handle and Modeless Dialogues

Even though we cannot make use of the Revit API unless we are inside a Revit external command context, which is a modal state, there is still a lot of very useful functionality that can be provided with the help of modeless dialogue boxes.
Modeless dialogues require knowledge of the Revit main window handle.

As an example of a modeless dialogue, we will implement a form which is displayed in parallel with the Revit user interaction for selecting elements on the graphic screen.
Our implementation of this dialogue simply reports the current contents of the Revit selection set.
Obviously, it could be adapted to prompt the user to select only specific elements, or a very specific combination of them.

In order to host a modeless dialogue box, one needs to supply a parent window handle.
This ensures that the modeless form is displayed on top of the parent window.

In .NET, a modeless dialogue is displayed to the user through the Form.Show method.
It has two overloads.
One of them takes an IWin32Window argument, which is defined by the System.Windows.Forms namespace to encapsulate a windows handle.
In that case, the specified window becomes the top-level owner of the form being displayed.
The other overload takes no arguments, in which case the owner is unspecified.
If so, Windows has no knowledge that the modeless dialogue belongs to Revit, thus there is no guarantee that it will appear on top of the Revit window.
In fact, to the contrary, as soon as the Revit window is activated, it will hide the modeless form.
This will sabotage our intention of using the modeless form to display information to the Revit user.

The Revit API does not provide the main application window handle directly.
The Windows API offers several possibilities for determining a window handle using various criteria.
Two of the most common approaches are based on the Win32 FindWindow and EnumWindows calls.

Our first discussion on
[driving Revit from outside](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html)
discussed and demonstrated the use of FindWindow.
It requires either the window class name or the full window caption.
The window class name depends on the exact version of Revit, and also changes depending on the state of the application.
For instance, the Revit window class name is different depending on whether a document is currently opened or not.
The window caption is also variable, again depending on the flavour and state of Revit and the name of the current document.

Another way to obtain a window handle makes use of the .NET framework Process class.
This has two advantages:

- This is pure .NET, no need make use of Win32 API calls.
- The process name is invariant, always "Revit".

This means we do not need to understand or import any methods from Win32 DLLs, and the code will work regardless of the Revit version number or flavour, i.e. Architecture, MEP, or Structure.

To retrieve all processes named "Revit", we can simply call Process.GetProcessesByName( "Revit" ).
That returns a list with zero or more entries of instances of the Process class.
The Process class provides a property MainWindowHandle to return its top level window handle.

As explained above, the Form.Show method takes an IWin32Window argument.
The value returned by the MainWindowHandle is an IntPtr, so we have to somehow convert it.
One way of doing so is to implement our own class implementing the IWin32Window interface.
The only required method is Handle, so our minimal window handle wrapper class implementation can look like this:

```python
public class WindowHandle : IWin32Window
{
  IntPtr \_hwnd;

  public WindowHandle( IntPtr h )
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

The rest is quite easy and straightforward, actually.
We have implemented an external command CmdWindowHandle which encapsulates the following steps:

- Determine the Revit top level window handle.
- Display a modeless form listing the current contents of the Revit selection set and prompting the user to continue the selection process.
- Drive the continued selection using the Revit API PickOne method.

Since we do not expect the Revit main window handle to change from one call of our external command to the next, we can determine it once and for all and store it in a static class variable \_hWndRevit.
We initialise \_hWndRevit to null, and set it on the first call to the command using the .NET GetProcessesByName and MainWindowHandle methods.

The form CmdWindowHandleForm is used to display the current contents of the Revit selection set.
It is simply a resizable form hosting one single label element, whose text can be set through a property LabelText:

```python
public partial class CmdWindowHandleForm : Form
{
  public CmdWindowHandleForm()
  {
    InitializeComponent();
  }

  public string LabelText
  {
    get
    {
      return label1.Text;
    }
    set
    {
      label1.Text = value;
    }
  }
}
```

We define two class variables for the default prompt and the Revit window handle:

```csharp
const string \_prompt
  = "Please select some elements.";

static WindowHandle \_hWndRevit = null;
```

The rest is the implementation of the main command line, which realises the three steps listed above:

```csharp
if( null == \_hWndRevit )
{
  Process[] processes
    = Process.GetProcessesByName( "Revit" );

  if( 0 < processes.Length )
  {
    IntPtr h = processes[0].MainWindowHandle;
    \_hWndRevit = new WindowHandle( h );
  }
}

Application app = commandData.Application;
Document doc = app.ActiveDocument;
Selection sel = doc.Selection;

using( CmdWindowHandleForm f
  = new CmdWindowHandleForm() )
{
  f.Show( \_hWndRevit );
  bool go = true;
  while( go )
  {
    SelElementSet ss = sel.Elements;
    int n = ss.Size;

    string s = string.Format(
      "{0} element{1} selected{2}",
      n, Util.PluralSuffix( n ),
      ((0 == n)
        ? ";\n" + \_prompt
        : ":" ) );

    foreach( Element e in ss )
    {
      s += "\n";
      s += Util.ElementDescription( e );
    }
    f.LabelText = s;
    sel.StatusbarTip = \_prompt;
    go = sel.PickOne();
  }
}
return CmdResult.Failed;
```

Note that we can initialise the variable 'sel' for the document selection outside the loop.
However, we cannot do the same for the currently selected elements stored in 'ss', because the sel.Elements property just returns a snapshot of the current state.
Therefore, we need to reinitialise that variable on each loop iteration.

In real life, the application looks like this; if the Revit selection set is initially empty, it prompts us to select some elements:

![Modeless form displaying prompt and empty selection set](img/window_handle_none.png)

Every time a new pick is made, the contents of the dialogue update to reflect the new state:

![Modeless form displaying some selected elements](img/window_handle_some.png)

At any point, the application can determine that a valid selection has been made and terminate the interaction to process the elements, or the user can make an empty pick to cause PickOne to return false and terminate the loop.

Here is
[version 1.0.0.21](http://thebuildingcoder.typepad.com/blog/files/bc10021.zip)
of the complete Visual Studio solution with the new command CmdWindowHandle discussed here, as well as the
[room and wall adjacency](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html)
command that we presented a few days ago, and an additional secret command that we have not presented in detail yet.
By the way, we are still working on the room and wall adjacency and will have something more to say about that and the Boolean operations for 2D polygons sometime soon.

As suggested below by Guy, I have updated the code to use GetCurrentProcess instead of GetProcessesByName.
This has several advantages:

- It completely removes any dependency whatsoever on the Revit executable name.
- When running multiple Revit sessions, it ensures you get the one with no additional logic.
- It siplifies the code above, and removes the need for an array of processes.

Here is the simplified code snippet for the first step using GetCurrentProcess:

```csharp
if( null == \_hWndRevit )
{
  Process process
    = Process.GetCurrentProcess();

  IntPtr h = process.MainWindowHandle;
  \_hWndRevit = new WindowHandle( h );
}
```

#### Various Places

I originally started working on this topic during the Revit API training in Barcelona, then started writing this specific post in Terni in Italy.
Asking for the single most important local sight of Terni, I was told there are none, this is an industrial town.
On second thoughts, the waterfall of Marmore was mentioned,
[Cascata delle Marmore](http://www.marmore.it),
which unfortunately closed before I got there.
Apparently, it is the highest waterfall in Europe.
Hard to believe, coming from Switzerland.
From Terni I continued to the beautiful towns and cities of Perugia, Firenze, and Bologna.
In Perugia, I met Gaetano, with whom I spent a wonderful evening together practicing amateur philosophy and very basic Italian.
In Firenze and Bologna, I was impressed with the size of the cathedrals.
I had originally thought of staying longer in Firenze, but my tourist allergy forced me to leave quickly.
Massive tourism polarises the entire population to a degree that I simply cannot stand.
I arrived in Verona in the north of Italy, which is much colder and where it has been raining incessantly.
I learned the word settiornio, for the north part of somthing, etymologically derived from seven, for the seven stars of the Ursa minor or major or something.

By the time I finally get to post this, much more has happened.
The world's first Revit API training in Italian in Verona is complete.
I met several extremely nice people and discovered that I like Italy very much indeed.
I performed some research on tiramisu and discovered a simple dessert of mascarpone con caffee which I liked even better.
I had two nice dinners with Stefano and Giovanna, and in the last one we had some pizza by the meter.