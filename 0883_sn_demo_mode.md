---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.791735'
original_url: https://thebuildingcoder.typepad.com/blog/0883_sn_demo_mode.html
post_number: 0883
reading_time_minutes: 2
series: general
slug: sn_demo_mode
source_file: 0883_sn_demo_mode.htm
tags:
- csharp
- elements
- references
- revit-api
- windows
title: Determine Revit Demo Mode and Serial Number
word_count: 348
---

### Determine Revit Demo Mode and Serial Number

Last week's
[updated Revit demo mode determination](http://thebuildingcoder.typepad.com/blog/2013/01/determine-revit-demo-mode-revisited.html) prompted
a lively discussion between two strong Revit API experts and blog contributors,
[Victor Chekalin](http://www.facebook.com/profile.php?id=100003616852588) and
[Rudolf Honke](http://www.acadgraph.de/) the
Revitalizer, partly on how to further improve the demo mode detection, and mainly on the value and risks of using unsupported features in your products.

The
[thread](http://thebuildingcoder.typepad.com/blog/2013/01/determine-revit-demo-mode-revisited.html#comments) is
well worth a read, and I like Victor's final suggestion of reading the Revit serial number instead of using the language dependent title bar caption enough to update The Building Coder sample accordingly.
He says:
> Just reference the UIFrameworkServices.dll assembly and read the Revit serial number with the InfoCenterService class static method ProductSerialNumber.
> If Revit is running in demo mode, the serial number is 000-00000000.
>
> [Here](http://pastebin.com/Lt8WUVhE) is the full source code.
>
> Simple, isn't it?

Victor also points out that it is obviously much easier to use System.Diagnostics.Process.GetCurrentProcess().MainWindowTitle to read the Revit main window caption than to do so using the WinAPI and user32.dll functionality.

I updated the CmdDemoCheck to make use of this and report both the serial number and the demo status:

```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    // . . .

    // Language independent serial number check:

    string serial\_number = UIFrameworkServices
      .InfoCenterService.ProductSerialNumber;

    isDemo = serial\_number.Equals( "000-00000000" );

    string sDemo = isDemo ? "Demo" : "Production";

    TaskDialog.Show(
      "Serial Number and Demo Version Check",
      string.Format(
        "Serial number: {0} : {1} version.",
        serial\_number, sDemo ) );

    return Result.Succeeded;
  }
```

Running this command on my system now produces the following result:

![Serial number and demo mode report](img/sn_demo_mode.png)

Here is
[version 2013.0.100.1](zip/bc_13_100_1.zip) of
The Building Coder samples including the updated CmdDemoCheck command and serial number detection.

Many thanks to Victor and Rudolf for their discussion and numerous creative suggestions.