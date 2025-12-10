---
post_number: "0534"
title: "Status Bar Text"
slug: "status_bar_text"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'windows']
source_file: "0534_status_bar_text.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0534_status_bar_text.html"
---

### Status Bar Text

The ski tour season in the alps has opened again, and I went on my first tour of the year last weekend, to the Ramoz hut and the Arosa Rothorn:

![Ski tour on Arosa Rothorn](img/2011-02-04_arosa_rothorn_collage.jpg)

Many thanks to Chris for the
[beautiful pictures](https://picasaweb.google.com/lh/sredir?uname=ruppchrissi&target=ALBUM&id=5570993040937253329&authkey=Gv1sRgCPrZv8zbi-mCXg&invite=CNHl7asN&feat=email)!

Here is another idea from Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de).
He says:

I previously explained how you can use
[UIAutomation event handlers](http://thebuildingcoder.typepad.com/blog/2011/01/subscribing-to-ui-automation-events.html) in Revit.

When playing around a bit further with this, I thought about how to **display** the events.

Using good old P/Invoke, you can simply display any text in the Revit status bar (<http://www.pinvoke.net> helps a lot):
```csharp
  [DllImport( "user32.dll",
    SetLastError = true,
    CharSet = CharSet.Auto )]
  static extern int SetWindowText(
    IntPtr hWnd,
    string lpString );

  [DllImport( "user32.dll",
    SetLastError = true )]
  static extern IntPtr FindWindowEx(
    IntPtr hwndParent,
    IntPtr hwndChildAfter,
    string lpszClass,
    string lpszWindow );

  public static void SetStatusText( string text )
  {
    IntPtr statusBar = FindWindowEx(
      m\_mainWndFromHandle, IntPtr.Zero,
      "msctls\_statusbar32", "" );

    if( statusBar != IntPtr.Zero )
    {
      SetWindowText( statusBar, text );
    }
  }
```

This is a comfortable way to show the events being fired.
Here is an example of resizing the main Revit window:

![Status bar displaying UI Automation event](img/rh_status_bar_1.png)

Resizing the main Revit window again:

![Status bar displaying UI Automation event](img/rh_status_bar_2.png)

Selecting the Home ribbon tab:

![Status bar displaying UI Automation event](img/rh_status_bar_3.png)

Selecting the Insert ribbon tab:

![Status bar displaying UI Automation event](img/rh_status_bar_4.png)

Selecting a button on the Annotation ribbon tab:

![Status bar displaying UI Automation event](img/rh_status_bar_5.png)

One point to keep in mind is that Revit will overwrite your text as soon as it sees fit, which may be within a few milliseconds, depending on the command you invoked.

Here is another button being selected and the corresponding UI Automation event displayed:

![Status bar displaying UI Automation event](img/rh_status_bar_6.png)

In this case, it is immediately overwritten by Revit:

![Status bar displaying UI Automation event](img/rh_status_bar_7.png)

Actually, this function could be combined with another function to replace the global variable 'm\_mainWndFromHandle':
```csharp
public static void SetStatusText( string text )
{
  IntPtr mainWindowHandle = IntPtr.Zero;

  Process[] processes
    = Process.GetProcessesByName( "Revit" );

  if( 0 < processes.Length )
  {
    mainWindowHandle
      = processes[0].MainWindowHandle;

    IntPtr statusBar = FindWindowEx(
      mainWindowHandle, IntPtr.Zero,
      "msctls\_statusbar32", "" );

    if( statusBar != IntPtr.Zero )
    {
      SetWindowText( statusBar, text );
    }
  }
}
```

So, this function could be called without filling the global Revit app handle before.
Or even shorter:
```csharp
public static void SetStatusText( string text )
{
  Process[] processes
    = Process.GetProcessesByName( "Revit" );

  if( 0 < processes.Length )
  {
    IntPtr statusBar = FindWindowEx(
      processes[0].MainWindowHandle, IntPtr.Zero,
      "msctls\_statusbar32", "" );

    if( statusBar != IntPtr.Zero )
    {
      SetWindowText( statusBar, text );
    }
  }
}
```

Think of a situation there more than one instance of Revit is running, RAC and MEP, for example.
Using the original code, the wrong window might be addressed.

Jeremy adds: I went ahead and implemented a minimal new external command CmdStatusBar for The Building Coder samples to demonstrate this.
I actually make use of the GetCurrentProcess method instead, since I am inside the Revit process, like this:
```csharp
public static void SetStatusText(
  IntPtr mainWindow,
  string text )
{
  IntPtr statusBar = FindWindowEx(
    mainWindow, IntPtr.Zero,
    "msctls\_statusbar32", "" );

  if( statusBar != IntPtr.Zero )
  {
    SetWindowText( statusBar, text );
  }
}

public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  IntPtr revitHandle = System.Diagnostics.Process
    .GetCurrentProcess().MainWindowHandle;

  SetStatusText( revitHandle, "Kilroy was here." );

  return Result.Succeeded;
}
```

It works fine.
Here is
[version 2011.0.83.0](zip/bc_11_83.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.