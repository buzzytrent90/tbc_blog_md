---
post_number: "0229"
title: "Dismiss Dialogue using Windows API"
slug: "dialogue_clicker"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'transactions', 'vbnet', 'windows']
source_file: "0229_dialogue_clicker.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0229_dialogue_clicker.html"
---

### Dismiss Dialogue using Windows API

I am often confronted with unwanted dialogues popping up when I am trying to drive some application programmatically.
In Revit, many dialogues raise an event when they are displayed and allow us to implement an add-in application to handle them automatically.
Unfortunately, as noted in the discussion on
[using the DialogBoxShowing event](http://thebuildingcoder.typepad.com/blog/2009/06/autoconfirm-save-using-dialogboxshowing-event.html),
some of the Revit dialogues do not raise such an event when displayed, so we cannot use the standard Revit API to handle or dismiss them.

Also, there are many other applications besides Revit that I am trying to drive programmatically, and they often do not provide any API at all, let alone a possibility to disable or automatically handle any of their user interface message boxes.

Furthermore, there may be situations within Revit where the DialogBoxShowing event is raised and the dialogue can be handled, but this causes problems within Revit.
For instance, one developer reports that the DialogBoxShowing event handler crashes when trying to hide the dialog using an override result after the dialogue has been triggered due to closing a transaction using EndTransaction.
In that case, Revit says that the document was modified outside the transaction and generates an exception.

**Question:** What can I do to automatically handle or dismiss dialogues in these situations?

**Answer:** As said, most Revit messages can be caught and responded to programmatically by a plug-in application by handling the Revit dialogue box DialogBoxShowing event, and
source code and a description for exploring this kind of situation is provided in the
[discussion](http://thebuildingcoder.typepad.com/blog/2009/06/autoconfirm-save-using-dialogboxshowing-event.html) mentioned above.

The easiest way to find out whether you can use this event to handle your specific dialogue is to:

- Implement and install the dialogue box handler.- Set a breakpoint in the event handler.- Reproduce the situation in which the dialogue is displayed by Revit.- Check whether the event was fired or not.

Inside the event handler, the debugger will show you what data is being passed in and thus how you can identify this dialogue in your code for automatic processing.

On the other hand, if the specific dialogue you are confronted with is not accessible in the Revit call-back, or you are dealing with some completely different application lacking the DialogBoxShowing event, I will explain below how you can use the
[.NET framework in C#](#1) and
[VB](#2) or the
[native Windows API](#3) to handle it automatically instead.
In short:

- In the .NET framework, you can use methods from the WinApi.User32 namespace like FindWindow, EnumWindows, and GetWindowText from inside a System.Timers.Timer instance to determine that a certain window is being displayed by the Windows OS and close it using WinApi.User32.SendMessage.- In the native Windows API, similar functionality can be achieved by creating a hook in the main Revit window with SetWindowsHookEx and sending a WM\_CLOSE or some other message to a dialogue when it is displayed.

#### .NET Dialogue Clicker in C#

Here is the C# source code and entire Visual Studio solution for a little utility named
[JtClicker](zip/JtClicker.zip) that
I have used many times in the past and is very near and dear to me.
It was especially useful during my time in Autodesk Consulting, when we were always automating a large number of different applications which unfortunately were equipped with user interfaces as well as APIs.
The user interfaces would often insist on popping up various irritating dialogue boxes which could not be suppressed programmatically.

The solution that I developed for this is a Windows API based stand-alone command-line utility that simply sits and waits for a dialogue with certain characteristics to appear.
If and when such a dialogue is displayed, the utility immediately dismisses it in a certain predefined way.

This solution I implemented is thus a generic dialogue clicker in C# using the .NET framework to access the underlying Windows API.
In fact, it has nothing at all to do with the Revit API.

Since every dialogue is different, we need a lot of flexibility in the details of identifying and handling various specific dialogues.
The two main requirements for a high degree of flexibility are:

- How to identify that the dialogue we are interested in has been displayed.- How the dialogue should be dismissed, once it has been identified.

Therefore, this utility will always require some tweaking before use and is only useful in source code format for that reason.
Check out the source code and the following discussion I had on that topic with my colleague Greg Wesner of Autodesk Consulting for details:

#### .NET Dialogue Clicker in VB

I was prompted to dive into this subject again recently by Greg Wesner of Autodesk Consulting.
Here is an example of my interaction with him when he made use of **JtClicker**, in which Greg also provides a VB version of the original C# implementation:

**[Q]** I heard you might have a sample of code for how to 'spoof' a click of the OK button in a windows dialog.
Specifically, I need to launch the options dialog in AutoCAD and then close it immediately with an OK click, i.e. I cannot cancel it.
Closing the dialog with OK results in AutoCAD refreshing all the paths.
Launching is obviously no problem but the OK click is tougher.
I'm working in VB.NET, so a .NET sample is really what I'm after.

**[A]** Yes, I am appending my little dialogue clicker to this mail.
There are comments inside form1 explaining its function and usage.
It searches for a dialogue using certain characteristics.
Once the dialogue appears, it sends windows messages to it in EnumChildProc.
You will need to tweak those messages to hit the button you need.

**[Q]** Thanks Jeremy. That was exactly what I needed. Hooked it all up and it is working great.

**[A]** Very glad to hear that it works for you!
Have you changed or improved anything in it?
If so, could you show me what you fixed or improved?

**[Q]** Well, I wish I could say that I improved on it, but you pretty much had all the parts I needed.
I was actually able to 'dumb' it down some because I could hard code the dialog name "Options" and the button "OK".
I also tested that "Options" and "OK" would dismiss the Options dialog in AutoCAD as expected with the original project files you sent.

I was hoping that I wouldn't need to use the timer and would be able to just make the call straight to EnumWindows.
But that didn't work because the Options dialog doesn't actually come up until my function returns.
So I had to use the timer as well to delay the call to EnumWindows.
I found dropping the interval to 300ms was better though, because you barely see the dialog come up.
I call Timer.Stop() as soon as I find the button I'm looking for, because I will only need it to happen once every time the dialog comes up.
I am thinking about adding some defensive code that counts how many times we call EnumWindows in the Timer, and if we go above a certain number of times (maybe 500), we stop trying.
That way if anything happens and the Options dialog never comes up, we don't chew up CPU cycles forever.

One thing I did do was translate WinApi.vb and the EnumWindowsProc() and EnumChildProc() methods to VB.
So I could pass that on to you in case you get requests for this in VB.
First, here is the module WinApi.vb providing access to the Windows API calls:
```vbnet
Imports System
Imports System.Runtime.InteropServices
Imports System.Text

Module User32
  Delegate Function EnumWindowsProc(ByVal hWnd As Integer, ByVal lParam As Integer) As Boolean
  <DllImport("user32.dll", CharSet:=CharSet.Unicode)> \_
  Function FindWindow(ByVal className As String, ByVal windowName As String) As Integer
  End Function
  <DllImport("user32.dll", CharSet:=CharSet.Unicode)> \_
  Function EnumWindows(ByVal callbackFunc As EnumWindowsProc, ByVal lParam As Integer) As Integer
  End Function
  <DllImport("user32.dll", CharSet:=CharSet.Unicode)> \_
  Function EnumChildWindows(ByVal hwnd As Integer, ByVal callbackFunc As EnumWindowsProc, ByVal lParam As Integer) As Integer
  End Function
  <DllImport("user32.dll", CharSet:=CharSet.Unicode)> \_
  Function GetWindowText(ByVal hwnd As Integer, ByVal buff As StringBuilder, ByVal maxCount As Integer) As Integer
  End Function
  <DllImport("user32.dll", CharSet:=CharSet.Unicode)> \_
  Function GetLastActivePopup(ByVal hwnd As Integer) As Integer
  End Function
  <DllImport("user32.dll", CharSet:=CharSet.Unicode)> \_
  Function SendMessage(ByVal hwnd As Integer, ByVal Msg As Integer, ByVal wParam As Integer, ByVal lParam As Integer) As Integer
  End Function
  Public BM\_SETSTATE As Integer = 243
  Public WM\_LBUTTONDOWN As Integer = 513
  Public WM\_LBUTTONUP As Integer = 514
End Module
```

Here is the main application:
```vbnet
Private timer1 As Timer
Private timer\_interval As Integer = 300  ' in milliseconds so 1000 = 1 second
Private timer\_attempts As Integer
Public Function EnumWindowsProc(ByVal hwnd As Integer, ByVal lParam As Integer) As Boolean
  Dim sbTitle As New StringBuilder(256)
  Dim test As Integer = User32.GetWindowText(hwnd, sbTitle, sbTitle.Capacity)
  Dim title As String = sbTitle.ToString()
  If title.Length = 0 Then
    Return True
  End If
  If title = "Options" Then
  Else
    Return True
  End If
  User32.EnumChildWindows(hwnd, New User32.EnumWindowsProc(AddressOf EnumChildProc), 0)
  Return False
End Function
Public Function EnumChildProc(ByVal hwnd As Integer, ByVal lParam As Integer) As Boolean
  Dim sbTitle As New StringBuilder(256)
  User32.GetWindowText(hwnd, sbTitle, sbTitle.Capacity)
  Dim title As String = sbTitle.ToString()
  If title.Length = 0 Then
    Return True
  End If
  If title = "OK" Then
  Else
    Return True
  End If
  User32.SendMessage(hwnd, User32.BM\_SETSTATE, 1, 0)
  User32.SendMessage(hwnd, User32.WM\_LBUTTONDOWN, 0, 0)
  User32.SendMessage(hwnd, User32.WM\_LBUTTONUP, 0, 0)
  User32.SendMessage(hwnd, User32.BM\_SETSTATE, 1, 0)
  timer1.Stop()
  timer1 = Nothing
  Return False
End Function
Public Sub timer1\_Tick(ByVal sender As Object, ByVal e As EventArgs)
  'timer\_attempts is a failsafe to make sure that if for some reason the Options dialog
  ' never comes up, we don't keep looking for it forever. We usually find the dialog on the
  ' first few tries, so if it doesn't come up in about the first minute, we give up
  If timer\_attempts < 300 Then
    User32.EnumWindows(New User32.EnumWindowsProc(AddressOf EnumWindowsProc), 0)
  Else
    timer1.Stop()
  End If
  timer\_attempts += 1
  Debug.Print(timer\_attempts.ToString())
End Sub
' The options dialog will not come up until our function returns, so we setup a timer
' that will check every 300 milliseconds to see if it can find the dialog. Once it finds
' the dialog and clicks OK, it stops.
Public Sub closeOptionsDialog()
  timer\_attempts = 0
  If timer1 Is Nothing Then
    timer1 = New Timer()
  End If
  timer1.Interval = timer\_interval
  AddHandler timer1.Tick, New EventHandler(AddressOf timer1\_Tick)
  timer1.Start()
End Sub
```

If the source code lines are truncated by your browser, you can copy and paste the text to a text editor.

#### Using a native Windows API Hook

The JtClicker dialogue clicker application described above is using the .NET framework to identify the dialogue and send messages to dismiss it.
Since the Win32 portion of the .NET framework is just a wrapper around native Windows API calls, the same functionality can also be implemented more directly using the native API.

Here is a report on using a Windows hook to dismiss a dialogue triggered by closing a Revit transaction.

I investigated using a hook, and this is the solution :-)
I created a hook in the main Revit window with SetWindowsHookEx and send messages to the dialogue when it is detected.
Initially I sent a WM\_CLOSE message to close it, later I changed it to click the OK button instead.
This works really well, even if the dialog is shown when ending a transaction, in which case the DialogBoxShowing event generates an exception if I try to hide the dialog with OverrideResult.

My hook function is:
```csharp
private static void ProcessWindow(
IntPtr hwnd )
{
  if (App.OcultarMensajesRevit)
  {
    // Buscamos el botón de aceptar y lo pulsamos.
    int ID\_OK = 1;
    IntPtr hwndOk = Win32.Functions.GetDlgItem(
      hwnd, ID\_OK );
    if (hwndOk != IntPtr.Zero)
    {
      Win32.Functions.SendMessage(
        hwndOk, (uint)Win32.Messages.WM\_LBUTTONDOWN,
        (int)Win32.KeyStates.MK\_LBUTTON, 0 );
      Win32.Functions.SendMessage(
        hwndOk, (uint)Win32.Messages.WM\_LBUTTONUP,
        (int)Win32.KeyStates.MK\_LBUTTON, 0 );
    }
    //Win32.Functions.SendMessage(
    // hwnd, (uint)Win32.Messages.WM\_CLOSE, 0, 0 );
  }
}
```

App.OcultarMensajesRevit is a variable I created to enable dialogue auto closing.
Instead of sending a WM\_CLOSE message to the window to cancel it, I observed I needed to push the OK button for all things to work, so I search for that button and send it a button down and a button up message to simulate a mouse click.

To connect the hook function, I used the
[WindowInterceptor class](http://www.codeproject.com/KB/dialog/WindowInterceptor.aspx)
I found at CodeProject.

Then I add the following in the Startup event of my application:
```csharp
Process process = Process.GetCurrentProcess();
IntPtr hwnd = process.MainWindowHandle;
App.\_windowsInterceptor = new WindowInterceptor(
hwnd, ProcessWindow );
```

To clean up at the end, I add the following on shutdown:
```csharp
App.\_windowsInterceptor.Stop();
```