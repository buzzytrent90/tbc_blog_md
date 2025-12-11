---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:13.318587'
original_url: https://thebuildingcoder.typepad.com/blog/0074_toolbar_image_list.html
post_number: '0074'
reading_time_minutes: 2
series: general
slug: toolbar_image_list
source_file: 0074_toolbar_image_list.htm
tags:
- csharp
- doors
- family
- geometry
- revit-api
title: Toolbar Image List
word_count: 473
---

### Toolbar Image List

**Question:**
How can I define a separate icon for each item in a toolbar?
I only see one single property on the Toolbar class, Image, which gets or sets the collection of images available to the toolbar button controls.
But how can this help me define the icon for an individual toolbar item?

**Answer:**
This is indeed the only way to define toolbar icons.
This is demonstrated by the Revit SDK samples.
For instance, in the DoorSwing sample, the code looks like this:

```csharp
// create custom Toolbar.
Toolbar doorToolbar
  = application.CreateToolbar();

doorToolbar.Name = "DoorSwing";

// The location of this command assembly
string currentCommandAssemblyPath
  = Assembly.GetExecutingAssembly().Location;

// The path of ourselves's DoorSwing.bmp
string toolbarImagePath
  = Path.GetDirectoryName(
    Path.GetDirectoryName(
      Path.GetDirectoryName(
        Path.GetDirectoryName(
          currentCommandAssemblyPath))));

toolbarImagePath
  = toolbarImagePath + "\\DoorSwing.bmp";

doorToolbar.Image
  = toolbarImagePath;

// the first button in the DoorSwing toolbar,
// used to invoke the InitializeCommand.
ToolbarItem iniButton
  = doorToolbar.AddItem(
    currentCommandAssemblyPath,
    typeof(InitializeCommand).FullName);

iniButton.ItemType
  = ToolbarItem.ToolbarItemType.BtnRText;

iniButton.ItemText
  = "Customize Door Opening Expression";

iniButton.ToolTip
  = "Customize the expression based on"
    + " family's geometry and country's standard.";
```

It continues similarly to add the second and third button.
One can see how the Image property is used to specify the file path of the bitmap file, which is set up to contain all the required icons for all buttons.
This pattern also reoccurs in the Toolbar SDK sample, as well as in the
[RvtMgdDbg](http://download.autodesk.com/media/adn/RvtMgdDbg2009_0429_2008.zip)
source code.

The repeated calls to GetDirectoryName are used to move up the directory hierarchy one step at a time, from the 'bin/Debug' to the 'bin' subdirectories, then to the 'CS' source code directory and finally to its parent directory, where the DoorSwing.bmp file is located.
A simpler way to achieve this would be to use something like this:

```
string p = @"c:\path1\path2\path3\path4";
p = System.IO.Path.GetFullPath( p + @"..\..\..\.." );
```

In this code, the ImageIndex property of the individual icons is never written to.
Apparently, they automatically increment by default, so that each toolbar item added to the toolbar gets the next icon in the list assigned to it unless specified differently.
In case you do make use of this property, the index is zero based.

The image file supplied in the DoorSwing sample looks like this:

![DoorSwing Image List](img/doorswing_image_list.png)

It contains three icons.
Its pixel size is 48 wide by 16 high, so each icon is 16 x 16 pixels.

To avoid errors, one should ensure that the bitmap file exists and has the correct dimensions, and that any indices used are within the valid range. To catch errors, the entire handling might be enclosed in a try-catch handler.

Many thanks to the company Tecton Limited of Hong Kong for raising this issue and also pointing out many of the details involved.