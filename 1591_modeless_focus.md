---
post_number: "1591"
title: "Modeless Focus"
slug: "modeless_focus"
author: "Jeremy Tammik"
tags: ['revit-api', 'sheets', 'views', 'windows']
source_file: "1591_modeless_focus.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1591_modeless_focus.html"
---

### Modeless Form Keep Revit Focus and On Top
I am back from a nice break in Italy, including a visit to
the [Giardino dei Tarocchi](https://en.wikipedia.org/wiki/Tarot_Garden),
the Tarot Garden sculpture garden based on the tarot cards, created by Niki de Saint Phalle
([photos](https://flic.kr/s/aHsm6U4nnA)):
![Breakfost on the beach](img/975_beach_breakfast_1k.jpg)
Before I get back to work properly attending the European Autodesk University in Darmstadt, Germany, here is a note from Hank DiVincenzo, Sr. Software Engineer at [Ideate, Inc](http://www.ideateinc.com), on keeping Revit focused and on top when working with a modeless form, and an important heads-up warning from the Revit development team on a future change coming:
- [Two issues](#2)
- [First issue problem](#3)
- [The first resolution](#4)
- [Code example for first resolution](#5)
- [Second issue context](#6)
- [The second problem](#7)
- [The second resolution](#8)
- [Code example for second resolution](#9)
- [Warning! Things will change in the next release](#10)
We here at Ideate Software are seeing what appears to be Revit add-in ownership issues with Revit's main window.
The behavior has changed between Revit 2017 and Revit 2018 for modeless add-ins.
For Revit 2018, when a modeless add-in is closed, Revit does not retain is focus; it is pushed behind another application.
With some experimentation, it was found the application that had focus before Revit is the one that gains focus after the modeless add-in is closed.
Revit 2016 and 2017 do not show this behavior, only Revit 2018 does.
Also Note:
What we understand as the preferred method of getting Revit's MainWindowHandle has issues when there are multiple modeless add-ins opened.
When the MainWindowHandle is gotten from the Revit process (i.e. using the `Process.GetCurrentProcess()` `MainWindowHandle` property), modeless add-in ownership is incorrect.
Any modeless add-in that is started after the first one has its ownership set to that previous modeless add-in. This can be seen by closing any of the interim modeless add-ins and any modeless add-ins started after the one being closed, also closes.
What I have called 'owner' is also known as the parent-child window relationship.
Here is the write-up of the two window issues we ran into and have solved.
To demonstrate, I am also sharing the Visual Studio 2013
project [ModelessAddinSolution.zip](zip/hdv_ModelessAddinSolution.zip).
It implements an add-in that can be used to demonstrate the focus issue and its solution following the instructions provided below. It also includes the add-in manifest `.addin` file in the \*External Application\* folder.
#### Two Issues
This describes my journey to solve two issues with Revit add-in windows.
The first issue was a general problem seen in all version of Revit, the second issue was specific to Revit 2018.
#### First Issue Problem
While developing a second modeless add-in, we ran into an issue with add-in window parenting/ownership. The problem manifested itself when having two modeless add-ins open: when the first add-in opened was closed, the second add-in opened also closed. With this behavior, it was apparent the second add-in was somehow being associated with the first add-in. We were using \*Process.GetCurrentProcess().MainWindow\* to get main Revit window. Somehow, once the first modeless add-in was opened, the `MainWindow` property for the Revit process changed. A new method for getting Revit's main window was needed.
#### The First Resolution
An alternate solution was needed to get the window handle for Revit's main window, the window that holds the ribbon and is the first window opened.
The following solution was implemented:
- Find all windows within the Revit process
- Find the Revit windows that do not have a parent assigned to them, i.e. their parent/owner property is null.
The thought here is windows that have no parent are those opened by the OS and not another part of Revit.
- Then, for the Revit windows not having a parent, find the one window that has “Autodesk Revit” within its title.
This method seems fragile because it depends on text from Revit's main window title, but it consistently gets the Revit main window no matter what add-ins are open. And localization is not an issue, the main window title text “Autodesk Revit” does not change for other languages, cf. the following two language examples.
French:
![French Revit title bar](img/revit_title_bar_french.jpg)
German:
![German Revit title bar](img/revit_title_bar_german.png)
#### Code Example for First Resolution
Class usage within the add-in's main window; the add-in is WPF a app:
```csharp
// Set Revit as owner of given child window
WindowHandleSearch handle
= WindowHandleSearch.MainWindowHandle;
handle.SetAsOwner(this);
```
Main Window Handle Class:
```csharp
///
/// Wrapper class for window handles
/// summary>
public class WindowHandleSearch
: IWin32Window, System.Windows.Forms.IWin32Window
{
#region Static methods
///
/// Revit main window handle
/// summary>
static public WindowHandleSearch MainWindowHandle
{
get
{
// Get handle of main window
var revitProcess = Process.GetCurrentProcess();
return new WindowHandleSearch(
GetMainWindow( revitProcess.Id ) );
}
}
#endregion
#region Constructor
///
/// Constructor - From WinForms window handle
/// summary>
/// Window handleparam>
public WindowHandleSearch( IntPtr hwnd )
{
// Assert valid window handle
Debug.Assert( IntPtr.Zero != hwnd,
"Null window handle" );
Handle = hwnd;
}
#endregion
#region Methods
///
/// Window handle
/// summary>
public IntPtr Handle { get; private set; }
///
/// Set this window handle as the owner
/// of the given window
/// summary>
/// Child window
/// whose parent will be set to be this window
/// handleparam>
public void SetAsOwner( Window childWindow )
{
var helper = new WindowInteropHelper(
childWindow )
{ Owner = Handle };
}
// User32.dll calls used to get the Main Window for a Process Id (PID)
private delegate bool EnumWindowsProc(
HWND hWnd, int lParam );
[DllImport( "user32.DLL" )]
private static extern bool EnumWindows(
EnumWindowsProc enumFunc, int lParam );
[DllImport( "user32.dll", ExactSpelling = true,
CharSet = CharSet.Auto )]
private static extern IntPtr GetParent( IntPtr hWnd );
[DllImport( "user32.dll", SetLastError = true )]
private static extern uint GetWindowThreadProcessId(
IntPtr hWnd, out uint processId );
[DllImport( "user32.DLL" )]
private static extern IntPtr GetShellWindow();
[DllImport( "user32.DLL" )]
private static extern bool IsWindowVisible( HWND hWnd );
[DllImport( "user32.DLL" )]
private static extern int GetWindowTextLength( HWND hWnd );
[DllImport( "user32.DLL" )]
private static extern int GetWindowText( HWND hWnd,
StringBuilder lpString, int nMaxCount );
///
/// Returns the main Window Handle for the
/// Process Id (pid) passed in.
/// IF the Main Window is not found then a
/// handle value of Zreo is returned, no handle.
/// summary>
/// param>
/// returns>
public static IntPtr GetMainWindow( int pid )
{
HWND shellWindow = GetShellWindow();
List windowsForPid = new List();
try
{
EnumWindows(
// EnumWindowsProc Function, does the work
// on each window.
delegate ( HWND hWnd, int lParam )
{
if( hWnd == shellWindow ) return true;
if( !IsWindowVisible( hWnd ) ) return true;
uint windowPid = 0;
GetWindowThreadProcessId( hWnd, out windowPid );
// if window is from Pid of interest,
// see if it's the Main Window
if( windowPid == pid )
{
// By default Main Window has a
// Parent Window of Zero, no parent.
HWND parentHwnd = GetParent( hWnd );
if( parentHwnd == IntPtr.Zero )
windowsForPid.Add( hWnd );
}
return true;
}
// lParam, nothing, null...
, 0 );
}
catch( Exception )
{ }
return DetermineMainWindow( windowsForPid );
}
///
/// Finds Revit's Main Window from the list of
/// window handles passed in.
/// If the Main Window for Revit is not found
/// then a Null (IntPtr.Zero) handle is returnd.
/// summary>
/// param>
/// returns>
private static IntPtr DetermineMainWindow(
List handles )
{
// Safty conditions, bail if not met.
if( handles == null || handles.Count <= 0 )
return IntPtr.Zero;
// default Null handel
HWND mainWindow = IntPtr.Zero;
// only one window so return it,
// must be the Main Window??
if( handles.Count == 1 )
{
mainWindow = handles[0];
}
// more than one window
else
{
// more than one candidate for Main Window
// so find the Main Window by its Title, it
// will contain "Autodesk Revit"
foreach( var hWnd in handles )
{
int length = GetWindowTextLength( hWnd );
if( length == 0 ) continue;
StringBuilder builder = new StringBuilder(
length );
GetWindowText( hWnd, builder, length + 1 );
// Depending on the Title of the Main Window
// to have "Autodesk Revit" in it.
if( builder.ToString().ToLower().Contains(
"autodesk revit" ) )
{
mainWindow = hWnd;
break; // found Main Window stop and return it.
}
}
}
return mainWindow;
}
#endregion
}
```
#### Second Issue Context
With the main window found and working for Revit 2016 and 2017, all was good and a couple more WPF modeless applications were developed. Then came Revit 2018...
#### The Second Problem
While porting our add-in(s) to Revit 2018, we noticed Revit lose focus from time to time. When our modeless (WinForms based) add-in window was closed, Revit was not left on top with focus; instead, the application that had focus **before** Revit was placed on top. This was a surprise, being something that Windows should handle. Notable was this new losing focus behavior was only seen in Revit 2018, and not in 2017 or 2016.
An example of the sequence would be:
- Open Revit
- Open Windows File Manager and ensure it is placed on top of Revit
- Switch back to Revit
- Open the add-in
- Close the add-in
File Manager would now be placed on top, i.e., Revit lost its position in the application stack/show, and was replaced by File Manager.
The above was the case for the WinForms based add-ins, but WPF based add-ins needed additional steps:
- Open Revit
- Open Windows File Manager and ensure it is placed on top of Revit
- Switch back to Revit
- Open the add-in
- Open a modal dialog from the modeless add-in
- Close the modal dialog
- Close the add-in
File Manager would then be placed on top, replacing Revit as the top application.
#### The Second Resolution
The resolution for the loss of focus ended up being somewhat simple.
I registered for the `OnClosing` event in the add-in's main window. Within the `OnClosing` event handler, I set Revit to be on top with the User32.dll call `SetForegroundWindow`.
Because the owner/parent of the add-in window was set correctly for Revit, setting the add-in's owner (Revit) on closing to be in the foreground solved the focus problem.
NOTE: See the section “Code for Second Resolution” below for the solution to this issue.
To Run the Sample Add-In:
- Open Revit 2018
- Open File Manager, ensure it is on top of Revit
- Switch back Revit, bring Revit to the top
- To see the solution
- Pick the “Id8 Revit 2018 Focus Solution” command From the Add-Ins Panel > Extern Tools
- Press the “Start Dialog” button
- Close the new dialog
- Close the Add-In dialog
- Revit Stays on top
- To see the Focus issue
- Pick the “Id8 Revit 2018 Focus Solution” command From the Add-Ins Panel > Extern Tools
- Uncheck the “Fix Focus Issue” checkbox
- Press the “Start Dialog” button
- Close the new dialog
- Close the Add-In dialog
- Revit is no longer on top, replaced by File Manager
#### Code Example for Second Resolution
```csharp
///
/// User32 calls used to set Revit focus
/// summary>
[DllImport( "USER32.DLL" )]
internal static extern bool SetForegroundWindow( HWND hWnd );
///
/// Use the OnClose event to ensure Revit is brought back into focus.
/// summary>
/// param>
protected override void OnClosing( CancelEventArgs e )
{
// do the base work first.
base.OnClosing( e );
// Set Revit to the foreground
try
{
IntPtr ownerIntPtr = new WindowInteropHelper( this ).Owner;
if( ownerIntPtr != IntPtr.Zero )
User32.SetForegroundWindow( ownerIntPtr );
}
catch( Exception )
{ }
}
```
Very many thanks to Hank for his persistent research and sharing this valuable solution.
#### Warning! Things Will Change in the Next Release
I myself always used the `JtWindowHandle` class to retrieve the main Revit window handle in the past, and use that
to [set up the parent window relationship for my modeless forms](http://thebuildingcoder.typepad.com/blog/2010/06/revit-parent-window.html) for
simple single modeless forms.
I have been warned by the development team that this method will cease to work in the next major release of Revit, after Revit 2018:
A developer followed the advice
to [ensure a WPF add-in remains in foreground](http://thebuildingcoder.typepad.com/blog/2012/10/ensure-wpf-add-in-remains-in-foreground.html) in
building a new add-in.
This was written about 5 years ago.
As of the next major release of Revit, the method described there will no longer work.
An easy way to fix this will be provided. The Revit API will include new API calls providing an official way to get the application window handle:
- Get the handle of the Revit main window.
- Return the main window handle of the Revit application. This handle should be used when displaying modal dialogs and message windows to ensure that they are properly parented. This property replaces the \*System.Diagnostics.Process.GetCurrentProcess()\* `MainWindowHandle` property.