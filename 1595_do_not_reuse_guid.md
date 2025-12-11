---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:16.362216'
original_url: https://thebuildingcoder.typepad.com/blog/1595_do_not_reuse_guid.html
post_number: '1595'
reading_time_minutes: 1
series: general
slug: do_not_reuse_guid
source_file: 1595_do_not_reuse_guid.md
tags:
- parameters
- revit-api
- sheets
title: Do Not Reuse Guid
word_count: 285
---

### Do Not Reuse an Existing GUID
Several Revit API objects make use of
a [GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) to
uniquely identify themselves.
When you copy and paste source code including any such GUID, you need to take care to replace the original GUID by your own one.
You can easily create a new GUID using the Visual Studio GUID generator tool `guidgen.exe` and by other means, cf. the explanation on creating
an [add-in client id](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html#4) for
a Revit add-in manifest.
Boost your BIM recently encountered and reported this issue in
its [quick tip – change those GUIDs!](https://boostyourbim.wordpress.com/2017/10/13/quick-tip-change-those-guids)
Here is another case illustrating the kind of problems that can occur if you simply copy and reuse an existing GUID:
\*\*Question:\*\* I recently updated to Revit 2018.2.
As a result, a custom dockable panel now throws an error at load.
This error was not encountered until after the update:

```
Error Message:
Cannot register the same dockable pane ID more than once.
Parameter name: id
```

This is the code generating the error:
```csharp
DockablePaneId id
= new DockablePaneId( new Guid(
"{D7C963CE-B7CA-426A-8D51-6E8254D21157}" ) );
uiApp.RegisterDockablePane( id, "Xyz",
XyzPanelClass as IDockablePaneProvider );
```
What happened?
\*\*Answer:\*\* Several developers reported this issue.
I suspect people are copying code straight from
the [simpler dockable panel sample](http://thebuildingcoder.typepad.com/blog/2013/05/a-simpler-dockable-panel-sample.html),
not realizing they need to replace the GUID with their own one.
Create your own GUID to replace the original one and the problem will be resolved.
![GUID collision](img/guid_collision.png)