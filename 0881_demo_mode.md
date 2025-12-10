---
post_number: "0881"
title: "Determine Revit Demo Mode Revisited"
slug: "demo_mode"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'levels', 'references', 'revit-api', 'views', 'windows']
source_file: "0881_demo_mode.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0881_demo_mode.html"
---

### Determine Revit Demo Mode Revisited

Almost a year back, we talked about how to determine whether the current running Revit application is a
[demo version](http://thebuildingcoder.typepad.com/blog/2012/03/determine-revit-demo-mode.html) or not.

The initial suggestion was to test this by trying to execute real functionality modifying the model and then save it.
That obviously incurs significant overhead and may cause other problems as well.

Rudolf Honke suggested simply reading and evaluating the Revit main window title text instead.

Now Madmed created and
[posted](http://thebuildingcoder.typepad.com/blog/2012/03/determine-revit-demo-mode.html?cid=6a00e553e168978833017ee738862f970d#comment-6a00e553e168978833017ee738862f970d) an
implementation of Rudolf's idea.
I took the opportunity of both adding that as a new command CmdDemoCheck to The Building Coder samples and simultaneously incrementing all the year number references in the source code comments from 2012 to 2013, so that that trivial yearly update step is over and done with.

Another obvious step to be performed is the elimination of all warning messages when compiling the code, since they indicate obsolete API usage that needs to be cleaned up.
Stay tuned for that anon.

Rudolf suggested checking whether the main window caption contains the words "VIEWER" and "MODE".
This is language dependent, of course, and would be "MODUS" in the German Revit, for instance.
He provided a link to
[C# and VB GetWindowText implementations](http://www.pinvoke.net/default.aspx/user32.getwindowtext) and
pointed out that this approach works in a zero-document scenario and will fail if the current document is named something like "VIEWER-MODE".
That problem could be worked around by more intelligent parsing of the string to extract the document name, for instance, before searching for the demo mode trigger string.

To begin with, here is Madmed's implementation:

```csharp
  [DllImport( "user32.dll", SetLastError = true,
    CharSet = CharSet.Auto )]
  static extern int GetWindowText(
    IntPtr hWnd,
    StringBuilder lpString,
    int nMaxCount );

  [DllImport( "user32.dll", SetLastError = true,
    CharSet = CharSet.Auto )]
  static extern int GetWindowTextLength(
    IntPtr hWnd );

  static StringBuilder GetStatusTextMadmed(
    IntPtr mainWindow )
  {
    StringBuilder s = new StringBuilder();
    if( mainWindow != IntPtr.Zero )
    {
      int length = GetWindowTextLength( mainWindow );
      StringBuilder sb = new StringBuilder( length + 1 );
      GetWindowText( mainWindow, sb, sb.Capacity );
      sb.Replace( "Autodesk Revit Architecture 2013 - ", "" );
      return sb;
    }
    return s;
  }
```

He suggests using it like this:

```csharp
  IntPtr revitHandle = System.Diagnostics.Process
    .GetCurrentProcess().MainWindowHandle;

  StringBuilder sb = GetStatusText(revitHandle);

  bool isDemo = sb.ToString()[0] != '['
```

When trying that out on my system, I noticed that I my Revit window title does not include the "Architecture" sub-string, since I am running the one-box version.

I also thought that it might be handy and cleaner to start off by implementing a method which simply returns the entire Revit main window caption string unmodified, for the caller to use freely in any desired fashion.
I ended up with the following implementation:

```csharp
  static string GetWindowText( IntPtr mainWindow )
  {
    if( IntPtr.Zero == mainWindow )
    {
      throw new ArgumentException(
        "Expected valid window handle." );
    }
    int len = GetWindowTextLength( mainWindow );
    StringBuilder sb = new StringBuilder( len + 1 );
    GetWindowText( mainWindow, sb, sb.Capacity );
    return sb.ToString();
  }
```

That returns "Autodesk Revit 2013 - Not For Resale Version - [Floor Plan: Level 1 - rac\_empty.rvt]" on my system.

The sample command making use of this currently looks like this:

```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    IntPtr revitHandle = System.Diagnostics.Process
      .GetCurrentProcess().MainWindowHandle;

    string s = GetWindowText( revitHandle );

    // My system returns:
    // "Autodesk Revit 2013 - Not For Resale Version
    // - [Floor Plan: Level 1 - rac\_empty.rvt]"

    bool isDemo = s.Contains( "VIEWER" );

    TaskDialog.Show( "Demo Version Check",
      ( isDemo ? "Demo" : "Production")
      + " version." );

    return Result.Succeeded;
  }
```

Here is
[version 2013.0.100.0](zip/bc_13_100_0.zip) of
The Building Coder samples with incremented year numbers throughout and including the new CmdDemoCheck command.

Many thanks to Rudolf for his helpful ideas and to Madmed for prompting me to pick this up again.

#### Retain or Update External Command GUID

Here are some thoughts by my colleague Joe Ye on whether to retain or update an external command GUID:

It is acceptable to keep the GUID unchanged for different versions of your Revit add-in.
If you release several different versions for the same Revit version, e.g. 2013, you obviously have to make sure that only one add-in manifest file to load it is found.

If you prefer, it is also okay to define a different GUID for each version of your add-in.
In that case, please note that an external command with the same name from two different versions can be loaded simultaneously.

If the add-in creates its own Ribbon items, trying to create the same items twice over will also cause an error, since ribbon item names are unique.

Thank you, Joe, for these hints.