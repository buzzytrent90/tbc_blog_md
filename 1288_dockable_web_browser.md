---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.687396'
original_url: https://thebuildingcoder.typepad.com/blog/1288_dockable_web_browser.html
post_number: '1288'
reading_time_minutes: 7
series: general
slug: dockable_web_browser
source_file: 1288_dockable_web_browser.htm
tags:
- csharp
- elements
- revit-api
- windows
title: A Dockable Web Browser
word_count: 1493
---

### A Dockable Web Browser

The Revit API supports add-ins defining their own dockable panels, similar to the built-in Revit project browser and element properties panels.

Here is The Building Coder topic list of [Dockable Panel](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) related discussions:

- [What's New in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html)
- [A Simpler Dockable Panel Sample](http://thebuildingcoder.typepad.com/blog/2013/05/a-simpler-dockable-panel-sample.html)
- [RevitRubyShell for Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/05/revitrubyshell-for-revit-2014.html)
- [Revit 2014 Update Release 1](http://thebuildingcoder.typepad.com/blog/2013/08/revit-2014-update-release-1.html)
- [Open MEP Connector Warning](http://thebuildingcoder.typepad.com/blog/2013/08/open-mep-connector-warning.html)
- [Revit 2014 Update Release 2](http://thebuildingcoder.typepad.com/blog/2013/11/revit-2014-update-release-2.html)

I have repeatedly been asked bow to host a web browser in a dockable panel, and this query yesterday finally prompted me to take the plunge:

**Question:**
I am working on a project for Revit learning content and I would like to implement an add-in that opens a dockable window and basically opens a browser in that window that points to a URL (local or internet based).
I found this post on
[a simpler dockable panel sample](http://thebuildingcoder.typepad.com/blog/2013/05/a-simpler-dockable-panel-sample.html).

Through a little bit of hacking I figured out how to make it work for Revit 2015.
I thought maybe by looking at it I could figure out how the dockable window was populated and figure out how to launch a browser in there, but no luck.
I really have no clue when it comes to Revit API and coding at all.

Is this something you can help me with?
I feel like it is probably a pretty simple change to get this sample to open a browser in the window instead of whatever is there.
I could install the HTML files local I want to point at, but ideally it would allow me to put any URL as the destination.
I want to point at a specific Learning Content website I am working on, but would also like it to be able to allow a customer to point to their own internal learning content.

Any help you could give would be great.
I am trying to get a simple mock-up working so I can get someone interested enough in the concept to give me some development resources to make it a real feature in Revit.

**Answer:**
Here is a bit of research and a pointer to an existing blog post or two that might help answer the question:

First, you need code to open the HTML page.
That should happen automatically if you launch the HTML page using `Process.Start`.
That should open the browser on that page.

The use of `Process.Start` in a Revit add-in is described in
[launching a stand-alone executable](http://thebuildingcoder.typepad.com/blog/2014/07/launching-a-stand-alone-executable.html).

Oh dear, I see that you want the browser in your own window.

Cool that you already found and made use of the Simpler Dockable Panel Sample!

I was going to suggest that next.

Well, the third step would be to host a browser in that window.

If IE is enough of a browser for you, you can use the
[Microsoft .NET browser control](https://msdn.microsoft.com/en-us/library/aa752040(v=vs.85).aspx),
e.g. as described by CodeProject on
[using the WebBrowser control in .NET](http://www.codeproject.com/Articles/1102/Using-the-WebBrowser-control-in-NET).

There are alternatives to the built-in IE browser, and they obviously require more work, e.g.,
[replacing the .NET WebBrowser control with a better browser like Chrome](http://stackoverflow.com/questions/790542/replacing-net-webbrowser-control-with-a-better-browser-like-chrome).

Since you went to the effort to migrate the
[original DockableDialog sample](http://thebuildingcoder.typepad.com/blog/2013/05/a-simpler-dockable-panel-sample.html) to
Revit 2015, others will probably be interested in that as well.
I therefore created a
[DockableDialog GitHub repository](https://github.com/jeremytammik/DockableDialog) for
it and repeated that exercise.

You say, "Through a little bit of hacking I figured out how to make it work for Revit 2015."

My migration was totally straightforward and required no hacking whatsoever.

You might want to compare your version with mine, and take a look at the minimal changes I made for the migration.

The flat migrated add-in is provided in
[version 2015.0.0.0](https://github.com/jeremytammik/DockableDialog/releases/tag/2015.0.0.0),
and the changes from the Revit 2014 version are encapsulated in
[this commit](https://github.com/jeremytammik/DockableDialog/commit/df96cf4f267cce8a98b8baacea57251dfadab463).

**Response:**
“Hacking” was probably the wrong word, or means something different to a guy like me who has NEVER written a line of code.
The ZIP file I downloaded did not want to run in 2015, so I had to figure out how to make changes to get it going.
It was all I could do to stumble through that. :-)

**Answer:**
Well done!

Next step: I now implemented a web browser in the dockable panel sample.

Check out the GitHub
[version 2015.0.0.1](https://github.com/jeremytammik/DockableDialog/releases/tag/2015.0.0.1).

Added a XAML web browser control, and you can click the buttons at the top to switch between The Building Coder and GitHub:

![DockableWebBrowser docked](img/DockableWebBrowser_docked.png)

This provides the proof of concept you require.

You could also use WinForms instead of WPF, so more .NET and C# rather than XAML, if you prefer that.

Good luck and have fun!

#### Script Error

Displaying The Building Coder web page hosted by Typepad in the WebBrowser control generates an error message saying, "An error has occurred in the script on this page":

![DockableWebBrowser script error](img/DockableWebBrowser_script_error.png)

I searched for a solution for that.

I started looking up this error and found things like
[how to handle script errors as a WebBrowser control host](http://support.microsoft.com/kb/261003) and
[script error in web browser](http://stackoverflow.com/questions/3202572/script-error-in-web-browser).

That was not what I wanted, so I narrowed it down by searching for 'disable' as well, discovering how to
[disable JavaScript error in WebBrowser control](http://stackoverflow.com/questions/2476360/disable-javascript-error-in-webbrowser-control) and
make use of the
[WebBrowser.ScriptErrorsSuppressed property](https://msdn.microsoft.com/en-us/library/system.windows.forms.webbrowser.scripterrorssuppressed(v=vs.110).aspx).

Unfortunately, the WPF control does not implement this property, so a further search for 'wpf web browser control disable script error' turned up numerous solutions such as how to
[disable script errors WebBrowser WPF](http://wpf-tutorial-net.blogspot.de/2013/11/disable-script-errors-webbrowser-wpf.html),
which is exactly what I need.

I added two URL definitions to the add-in XAML code, pointing to
[The Building Coder](http://thebuildingcoder.typepad.com) and this
[DockableDialog sample GitHub repository](https://github.com/jeremytammik/DockableDialog):

```csharp
  const string \_url\_tbc = "http://thebuildingcoder.typepad.com";
  const string \_url\_git = "https://github.com/jeremytammik/DockableDialog";
```

Here is the resulting XAML driver code driving the buttons to toggle the browser location back and forth between these two URLs and disable the JavaScript warning messages triggered by the Typepad framework hosting The Building Coder:

```csharp
  private void PaneInfoButton\_Click(
    object sender,
    RoutedEventArgs e )
  {
    web\_browser.Navigate( \_url\_tbc );
  }

  private void wpf\_stats\_Click(
    object sender,
    RoutedEventArgs e )
  {
    web\_browser.Navigate( \_url\_git );
  }

  private void btn\_getById\_Click(
    object sender,
    RoutedEventArgs e )
  {
    web\_browser.Navigate( \_url\_tbc );
  }

  private void btn\_listTabs\_Click(
    object sender,
    RoutedEventArgs e )
  {
    web\_browser.Navigate( \_url\_git );
  }

  private void DockableDialogs\_Loaded(
    object sender,
    RoutedEventArgs e )
  {
    web\_browser.Navigated += new NavigatedEventHandler(
      WebBrowser\_Navigated );

    web\_browser.Navigate( \_url\_tbc );
  }

  void WebBrowser\_Navigated(
    object sender,
    NavigationEventArgs e )
  {
    HideJsScriptErrors( (WebBrowser) sender );
  }

  public void HideJsScriptErrors( WebBrowser wb )
  {
    // IWebBrowser2 interface
    // Exposes methods that are implemented by the
    // WebBrowser control
    // Searches for the specified field, using the
    // specified binding constraints.

    FieldInfo fld = typeof( WebBrowser ).GetField(
      "\_axIWebBrowser2",
      BindingFlags.Instance | BindingFlags.NonPublic );

    if( null != fld )
    {
      object obj = fld.GetValue( wb );
      if( null != obj )
      {
        // Silent: Sets or gets a value that indicates
        // whether the object can display dialog boxes.
        // HRESULT IWebBrowser2::get\_Silent(VARIANT\_BOOL \*pbSilent);
        // HRESULT IWebBrowser2::put\_Silent(VARIANT\_BOOL bSilent);

        obj.GetType().InvokeMember( "Silent",
          BindingFlags.SetProperty, null, obj,
          new object[] { true } );
      }
    }
  }
```

As said, the complete code for this, Visual Studio solution and add-in manifest is provided in the
[DockableDialog GitHub repository](https://github.com/jeremytammik/DockableDialog),
and the version described here is
[release 2015.0.0.3](https://github.com/jeremytammik/DockableDialog/releases/tag/2015.0.0.3).