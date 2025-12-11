---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.9
content_type: code_example
optimization_date: '2025-12-11T11:44:14.129149'
original_url: https://thebuildingcoder.typepad.com/blog/0538_zero_doc_ribbon.html
post_number: 0538
reading_time_minutes: 4
series: general
slug: zero_doc_ribbon
source_file: 0538_zero_doc_ribbon.htm
tags:
- python
- revit-api
- transactions
title: Enable Ribbon Items in Zero Document State
word_count: 842
---

### Enable Ribbon Items in Zero Document State

Several people have asked about how to activate their custom add-in panel items when Revit is in zero document state, i.e. when no document has been opened in the Revit user interface:

![Zero document state](img/zero_doc_state.png)

For an external command, it is no big deal: if you set both its transaction mode and regeneration option to Manual, it will appear in the Revit ribbon panel under Add-Ins > External > External Tools and also be available in zero document state:

![Zero document state external tools](img/zero_doc_state_external_tools.png)

However, as you can see, all of the panels defined by external applications are disabled in this state.

Here is the code of an initial version of the external application ZeroDocPanel creating the right-most ribbon panel and push button that you see above and which is disabled in zero document state:
```python
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
class App : IExternalApplication
{
  public const string Caption = "Zero Doc Panel";
  public const string Caption2 = "Zero Doc Button";

  static string \_path
    = System.Reflection.Assembly
      .GetExecutingAssembly().Location;

  public Result OnStartup( UIControlledApplication a )
  {
    RibbonPanel p = a.CreateRibbonPanel( Caption );

    PushButton b = p.AddItem(
      new PushButtonData( Caption2, Caption2,
        Path.GetDirectoryName( \_path ),
        "ZeroDocPanel.Command" ) ) as PushButton;

    b.ToolTip = "This is the zero doc panel button tooltip";
    return Result.Succeeded;
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    return Result.Succeeded;
  }
}
```

This leads to the following question:

**Question:** I want to enable the ribbon items in a panel defined by my external application in zero document state.
How can I achieve that, please?

In the Revit API help file RevitAPI.chm, I see a statement saying that "The push button will be disabled when there is no active document unless availability class of that button returns true".
Could you please elaborate on this?

**Answer:** Yes, this statement is correct.
Buttons defined by an external application are disabled by default in zero document state.
If you provide an appropriate external command availability class implementation and return true, they will be enabled. Have you tried this?

We discussed the availability class in the presentation of the
[RevitAddInUtility](http://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html),
and also touched on it briefly when looking at the
[Revit 2011 product GUIDs](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-product-guids.html#1), the
[add-in manifest tags](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html#2), and the
[pipe to conduit converter](http://thebuildingcoder.typepad.com/blog/2010/05/pipe-to-conduit-converter.html)

Besides being applied to external commands, you can also use the
[PushButton AvailabilityClassName property](http://thebuildingcoder.typepad.com/blog/2010/09/simulating-a-ribbon-textbox-label.html) to
control visibility of individual buttons.

By the way, you have to read the documentation of the IExternalCommandAvailability interface pretty carefully.
One of the statements it makes is "This interface should share the same assembly with add-in External Command".

Let's enhance the external application presented above to enable its panel in zero document state.
As said, we need to implement the IExternalCommandAvailability interface.
Here is a very trivial implementation that simply always returns true:
```python
public class Availability
  : IExternalCommandAvailability
{
  public bool IsCommandAvailable(
    UIApplication a,
    CategorySet b )
  {
    return true;
  }
}
```

Here is the enhanced code of the ZeroDocPanel external application making use of the command availability class:
```python
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
class App : IExternalApplication
{
  public const string Caption = "Zero Doc Panel";
  public const string Caption2 = "Zero Doc Button";

  static string \_path
    = System.Reflection.Assembly
      .GetExecutingAssembly().Location;

  public Result OnStartup(
    UIControlledApplication a )
  {
    PushButtonData d = new PushButtonData(
      Caption2, Caption2, \_path,
      "ZeroDocPanel.Command" );

    d.AvailabilityClassName
      = "ZeroDocPanel.Availability";

    RibbonPanel p = a.CreateRibbonPanel( Caption );

    PushButton b = p.AddItem( d ) as PushButton;

    b.ToolTip
      = "This is the zero doc panel button tooltip";

    //b.AvailabilityClassName
    //  = "ZeroDocPanel.Availability";

    return Result.Succeeded;
  }

  public Result OnShutdown(
    UIControlledApplication a )
  {
    return Result.Succeeded;
  }
}
```

One important thing to note is that the AvailabilityClassName property must be set on the PushButtonData instance before adding the push button to the panel, i.e. before calling the AddItem method with the PushButtonData information.

Setting this property afterwards will have no effect!

Alternatively, one can set the AvailabilityClassName property on the resulting PushButton instance itself after it has been added.

And, as said, the command availability class and external command implementation must live in the same .NET assembly.
In our case, both of them and the external application implementation as well all share the same assembly.

Here is the ZeroDocPanel external application showing its activated push button in zero document state:
![Zero document state ribbon panel enabled](img/zero_doc_state_panel_enabled.png)

Here is
[ZeroDocPanel.zip](zip/ZeroDocPanel.zip) containing
the entire source code and Visual Studio solution implementing this add-in.

**Addendum:** As Arnošt points out in his comment below, the Transaction attribute shown above is unnecessary and will be ignored.
I discuss this issue in more detail in an updated note on
[external application attributes](http://thebuildingcoder.typepad.com/blog/2011/02/external-application-attributes.html),
and removed the attribute in the zip file above.