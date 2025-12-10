---
post_number: "1386"
title: "Stacked Split Button"
slug: "stacked_split_button"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'python', 'references', 'revit-api', 'rooms', 'selection', 'sheets', 'views', 'walls', 'windows']
source_file: "1386_stacked_split_button.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1386_stacked_split_button.html"
---

### Adding a Stacked Split Button to the Ribbon
I have been rather busy on the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) these last few days.
One of the issues I got involved with was Michał Helt's thread
on [RibbonPanel.AddStackedItems and SplitButton](http://forums.autodesk.com/t5/revit-api/ribbonpanel-addstackeditems-and-splitbutton/m-p/5949953),
discussing how to add a split button to a ribbon panel using the AddStackedItems method.
\*\*Question:\*\* Currently, Split Buttons cannot be created in the ribbon using the AddStackedItems method.
Only PushButtons, PulldownButtons, ComboBoxes and TextBoxes can be added this way.
Would it be possible to remove this limitation?
I would like to achieve a button similar to this selected one:
![Split button](img/split_button.png)
I've already overcome this using the ribbon features from AdWindows.dll, but I am curious whether the lack of it in RevitAPIUI.dll is intentional or not.
\*\*Answer:\*\* Thank you for reporting this!
This issue has already been logged as REVIT-71373 \*As a Revit add-in developer, I would like to be able to add a split button to a stacked section of buttons, so I can create the UI I need for my application\* and is currently being addressed, so you will have official access to this functionality in future.
\*\*Response:\*\* Thank you.
Here is my current workaround making use of
the [unsupported](http://thebuildingcoder.typepad.com/blog/about-the-author.html#4) `AdWindows.dll` functionality:

```
  var bd0 = new PulldownButtonData( "A", "A" );
  var bd1 = new PulldownButtonData( "B", "B" );
  var bd2 = new PulldownButtonData( "C", "C" );

  var stackedItems = ribbonPanel.AddStackedItems(
    bd0, bd1, bd2 );

  var button0 = (PulldownButton) stackedItems[0];

  string sid = string.Join(
    "%",
    "CustomCtrl_",
    "CustomCtrl_",
    ribbonTabName,
    ribbonPanel.Name,
    button0.Name );

  var splitButton = (Autodesk.Windows.RibbonSplitButton)
    UIFramework.RevitRibbonControl.RibbonControl
      .findRibbonItemById( sid );

  splitButton.IsSplit = true;
  splitButton.IsSynchronizedWithCurrentItem = true;
```

That's the cleanest solution that I have found so far.
Thank you, Michał, for raising the issue and sharing this solution.
Don't forget to replace this by the official solution once the new functionality mentioned above becomes available.