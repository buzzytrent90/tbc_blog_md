---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.288902'
original_url: https://thebuildingcoder.typepad.com/blog/0051_driving_revit.html
post_number: '0051'
reading_time_minutes: 7
series: general
slug: driving_revit
source_file: 0051_driving_revit.htm
tags:
- csharp
- elements
- revit-api
- selection
- transactions
- views
- windows
title: Driving Revit from Outside
word_count: 1307
---

### Driving Revit from Outside

We are winding up here at Autodesk University, with a final panel session to
[meet the AEC API experts](http://au.autodesk.com/sessions/?speaker=Jeremy+Tammik&year=2008)
yesterday afternoon, and one on
[advanced use of the Revit API](http://au.autodesk.com/sessions/detail/2866)
this morning.

One topic that keeps cropping up in infinite variations and was mentioned several times in my sessions here at AU is how to drive Revit reliably from an application.
The Revit API assumes that all interaction with the API functionality happens from within an external command.
In general, this requires user interaction to select a menu entry or click a toolbar button to trigger the external command execution.
In many cases, an application would like to make use of the API without requiring this explicit manual user interaction to initiate it.
One example of such a situation is a modeless dialogue displayed side by side with the Revit user interface.

It is actually possible to at least query the model from a modeless dialogue without explicitly starting a command.
It does help if you open a transaction before accessing the document.
Making modifications to a document definitely does not work, however, without being within in the context of an external command execution.
The good news is that it is possible to trigger the execution of an external command programmatically using Win32 API functions to simulate the required user input, such as a menu entry selection.
Here are some of the topics that are of interest in this context, related topics for me to remember and you to optionally ponder until we get around to discussing them in depth:

- Determining the Revit window handle.
- Initiating a Revit external command from an external application.
- Driving Revit from a modeless dialogue.
- Synchronisation issues, such as determining when a command has begun and terminated.
- Executing a command that creates new elements and determining which ones they are.
- Opening and activating a document in Revit.

Simply opening a new document in the background is not an issue, since the API provides the Application OpenDocumentFile method, but there is no Revit API call to activate it. There is, however, a known workaround. What do you think it is? Don't worry; we will get to it one of these days.

This post starts off the exploration of these issues by addressing the first two points, determining the Revit window handle and triggering some Revit menu picks from an external command line application. It is so simple that I can list the code right here and now:

```csharp
[DllImport( "USER32.DLL" )]
public static extern IntPtr FindWindow(
  string lpClassName, string lpWindowName );
[DllImport( "USER32.DLL" )]
public static extern bool SetForegroundWindow(
  IntPtr hWnd );

const string \_window\_class\_name\_zero
  = "Afx:00400000:8:00010011:00000000:007B05A1";
const string \_window\_class\_name\_project\_open
  = "Afx:00400000:8:00010011:00000000:007B05A1";

static int Main( string[] args )
{
  IntPtr revitHandle
    = FindWindow( \_window\_class\_name\_project\_open, null );
  if( IntPtr.Zero == revitHandle )
  {
    revitHandle = FindWindow( \_window\_class\_name\_zero, null );
  }
  if( IntPtr.Zero == revitHandle )
  {
    Console.WriteLine( "Unable to find Revit window."
      + " Is Revit Architecture up and running yet?" );
    return 1;
  }
  SetForegroundWindow( revitHandle );
  SendKeys.SendWait( "{F1}" );
  SetForegroundWindow( revitHandle );
  SendKeys.SendWait( "^{F10}{LEFT}{LEFT}{DOWN}{UP}{ENTER}" );
  return 0;
}
```

I am providing the complete Visual Studio solution implementing this
[console application named SendCmd](zip/SendCmd_1000.zip) here.

We use the two Win32 API functions FindWindow() to get a handle to an application window and SetForegroundWindow() to activate it.
In the call to the former, we need to specify either a window caption or a window class name.
Both of these can be obtained using the Visual Studio Spy++ tool, which is available under Start > Programs > Microsoft Visual Studio 2005 > Visual Studio Tools > Spy++.
The window class name varies from one Revit version to the next, and also between different flavours of Revit.
In addition, the class name is different depending on whether Revit has a project document open or not.
We would call the latter a zero document state.
Another difference between these two states is that no external tools can be activated in zero document state, which is another reason why it is useful to be able to differentiate between the two states.
The two class names for the open project and zero documents states specified in the code are valid for Revit Architecture 2009 Build 20080915\_2100.

The code in the Main() function searches for a window belonging to a running instance of Revit, first with an open project, then, if that fails, with no project open.
If that fails as well, a message is printed, and the program terminates.
Otherwise, it ensures that the Revit window is brought to the foreground and sends two keystroke sequences to it:

- `"{F1}"`  to open the help file.
- `"^{F10}{LEFT}{LEFT}{DOWN}{UP}{ENTER}"` to enter the menu system, navigate to a particular menu entry, and activate it.

The first sequence is a simple hit of the F1 function key to open the Revit help file.
The second sequence represents "Ctrl + F10, Left Arrow, Left Arrow, Down Arrow, Up Arrow, Enter".
This activates the Revit menu and then navigates to and selects the menu entry Help > About... to display the about box.
Thus, running this program with an open instance of Revit will pop up the Revit help file and about dialogue.

We will probably revisit this question in more depth and from other points of view in future discussions.
To start it off, we have looked at the simple scenario of running a standalone command line executable and triggering a menu selection in a running instance of Revit.
A related possibility would be to send language dependent strings to activate menu items.
For instance, to execute the line command on a Spanish version of Revit Architecture, one could use the string "(%M)":

- **(%M)** opens the "Modelling Menu".
- is the Spanish menu shortcut for the Line command.

In Revit 2008, it was also possible to send the simple keystroke sequence "LI" for the shortcut that appears in the keyboardShortcuts.txt file, but that no longer seems to work in Revit 2009.

An issue in this scenario is that the SendWait() method does not wait for the user to finalise the command.
It is asynchronous. Once you have sent the keystroke sequence, you need to wait until the command has actually been activated by Revit before you can start interacting with it.
How can we determine when the command is active, and also when it has ended?

For more information on the use of SendKeys and the preparation in setting the active foreground window, you can look at the MSDN documentation on how to
[simulate mouse and keyboard events](http://msdn2.microsoft.com/en-us/library/ms171548.aspx) in code.
The escape codes used with SendWait() are explained in the section on
[SendKeys.Send()](http://msdn2.microsoft.com/en-us/library/system.windows.forms.sendkeys.send.aspx).
Using Ctrl + F10 to activate the menu bar, followed by arrow keys to navigate to the proper menu entry, make the key sequence language independent,
so it does not need to be modified to handle different language dependencies, e.g. for the English or Spanish menu shortcuts.

There are
[other ways](http://forums.augi.com/showpost.php?p=765178&postcount=6)
of doing this as well. One developer mentioned that using SendMessage() requires more work to implement, but is considerably more reliable.
Also, in the minimal sample above, we are using a simple call to FindWindow() to determine the Revit handle.
This will not deal reliably with multiple sessions. In such a case, EnumWindow is sometimes a better way to go, or you can make use of the
[process name](http://www.mycsharpcorner.com/Post.aspx?postID=32)
to identify the required instance and retrieve its window handle.