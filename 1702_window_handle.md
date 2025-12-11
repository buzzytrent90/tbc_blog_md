---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:16.558959'
original_url: https://thebuildingcoder.typepad.com/blog/1702_window_handle.html
post_number: '1702'
reading_time_minutes: 4
series: general
slug: window_handle
source_file: 1702_window_handle.md
tags:
- csharp
- family
- levels
- revit-api
- sheets
- views
- windows
title: Window Handle
word_count: 837
---

### Revit Window Handle and Parenting an Add-In Form
Access to the Revit main window handle changed in Revit 2019, raising a couple of questions:
- [Making Revit the add-in parent](#2)
- [The Revit 2019 `MainWindowHandle` API](#3)
- [Docking system and multiple main window explanation](#4)
- [Updating The Building Coder samples](#5)
![Shattered window](img/shattered_window.jpg)

Shattered window © Benoit Brummer, [@Trougnouf](https://commons.wikimedia.org/wiki/User:Trougnouf)

#### Making Revit the Add-In Parent
A question came up in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) question
on [how to get plugin UI to be at the same level as Revit](https://forums.autodesk.com/t5/revit-api-forum/how-to-get-plugin-ui-to-be-at-the-same-level-as-revit/m-p/8392848) that
has in fact been asked repeatedly in the past.
Some of my past answers can be found by searching the forum for 'jtwindowhandle'.
As of Revit 2019, however, the answer needs to be modified and updated, so let's do so here and now:
\*\*Question:\*\* I'm currently setting my plug-in's UI to `TopMost`.
However, if I minimize Revit, my plugin stays on top.
Is there a way to have my plug-in's UI to match the functionality of Revit?
\*\*Answer:\*\* You have to ensure that your control is assigned the Revit main window as parent.
For instance, if you display your form using
the [.NET `ShowDialog` method](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.form.showdialog),
make use of its overload taking an IWin32Window argument:
- `ShowDialog(IWin32Window)` – Shows the form as a modal dialog box with the specified owner.
This approach was explained back in 2010 in the discussion on setting
the [Revit parent window](https://thebuildingcoder.typepad.com/blog/2010/06/revit-parent-window.html).
Please note that
the [Revit 2019 API provides direct access to the Revit main window handle](https://thebuildingcoder.typepad.com/blog/2018/08/whats-new-in-the-revit-20191-api.html#3.1.4).
#### The Revit 2019 MainWindowHandle API
Here is a brief quote from
the [Revit 2019 API news](https://thebuildingcoder.typepad.com/blog/2018/08/whats-new-in-the-revit-20191-api.html)
on the [direct access to the Revit main window handle](https://thebuildingcoder.typepad.com/blog/2018/08/whats-new-in-the-revit-20191-api.html#3.1.4):
> **1.4. UI API changes**
> **1.4.1. Main window handle access**
> Two new properties in the `Autodesk.Revit.UI` namespace provide access to the handle of the Revit main window:
> - `UIApplication.MainWindowHandle`
> - `UIControlledApplication.MainWindowHandle`
> This handle should be used when displaying modal dialogs and message windows to ensure that they are properly parented.
Use these properties instead of System.Diagnostics.Process.GetCurrentProcess().MainWindowHandle,
which is no longer a reliable method for retrieving the main window handle starting with Revit 2019.
The change was also pointed out by The Building Coder in October 2017 with
a [warning that things will change in the next release](https://thebuildingcoder.typepad.com/blog/2017/10/modeless-form-keep-revit-focus-and-on-top.html#10).
#### Docking System and Multiple Main Window Explanation
Revitalizer explains the need for the new property in his notes
on [assigning a name to a string in C# and `Process.GetCurrentProcess().MainWindowTitle`](https://forums.autodesk.com/t5/revit-api-forum/assign-a-name-to-a-string-c-process-getcurrentprocess/m-p/8365316):
> Due to the new docking system introduced in Revit 2019, both the `UIApplication` and `UIControlledApplication` classes now sport a `MainWindowHandle` property.
> It returns an `IntPtr` window handle that you can [P/Invoke `GetWindowText`](http://pinvoke.net/default.aspx/user32/GetWindowText.html) on to retrieve the window caption text.
> In Revit 2019, if view windows are pulled off the main window, there may be more than one Revit application window.
> If you open views in just one single Revit 2019 window, of course the 2018 code might still function, since it just finds the only one.
#### Updating The Building Coder Samples
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) were
still using the now obsolete `JtWindowHandle` class up
until [release 2019.0.143.9](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.143.9).
The update to switch to the new Revit `MainWindowHandle` property instead was prompted
by [issue #8](https://github.com/jeremytammik/the_building_coder_samples/issues/8) about
a keyboard shortcut problem with `CmdPressKeys` in Revit 2019.
The code in CmdPressKeys.cs was still retrieving the Revit main window handle via a call to `GetCurrentProcess`:
```csharp
IntPtr revitHandle = System.Diagnostics.Process
.GetCurrentProcess().MainWindowHandle;
```
I modified it to use `UiApplication MainWindowHandle` instead and removed the use of `JtWindowHandle`
in [release 2019.0.143.10](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.143.10).
Look at the modifications made to the modules CmdPlaceFamilyInstance.cs, CmdPressKeys.cs and JtWindowHandle.cs in
the [diff between the two versions](https://github.com/jeremytammik/the_building_coder_samples/compare/2019.0.143.9...2019.0.143.10).