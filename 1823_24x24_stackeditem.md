---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.9
content_type: qa
optimization_date: '2025-12-11T11:44:16.818298'
original_url: https://thebuildingcoder.typepad.com/blog/1823_24x24_stackeditem.html
post_number: '1823'
reading_time_minutes: 5
series: general
slug: 24x24_stackeditem
source_file: 1823_24x24_stackeditem.md
tags:
- csharp
- elements
- parameters
- revit-api
- sheets
- transactions
- views
- walls
- windows
title: 24x24 Stackeditem
word_count: 1042
---

### 24x24 StackedItem, Ribbon Utils, Journal Encoding
Let's start the week with some ribbon button item and encoding topics:
- [How to create 24x24 stacked ribbon items](#2)
- [Update on moving a ribbon button between panels](#3)
- [Revit journal file character encoding](#4)
#### How to Create 24x24 Stacked Ribbon Items
Jameson [jnyp](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/4918309) Nyp raised and solved this issue in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [24x24 StackedItems](https://forums.autodesk.com/t5/revit-api-forum/24x24-stackeditems/m-p/9337695):
\*\*Question:\*\* This may be an easy one, but so far I am struggling to find anything specific about it.
How do you make a `StackedItem` where the icons are 24x24 when there are only 2 in the stack?
It seems like it should be possible as it is used multiple times in the modify tab (see example below).
![Icon sizes](img/icon_sizes.png "Icon sizes")

Icon sizes

I have been able to set the `ShowText` property to false to get the 3 stacked icons, but when I use the same methodology with the 2 icon stack it remains 16x16 regardless of the icon resolution.
I have tried to obtain and change the button's height and width, minWidth and minHeight through the Autodesk.Window.RibbonItem object to no avail.
Has anyone had any success in creating these icons?
\*\*Solution:\*\* I found a solution.
In order to display the button at the 24x24 size the Autodesk.Windows.RibbonItem.Size needs to be manually set to Autodesk.Windows.RibbonItemSize.Large enum and a 24x24 icon needs to be set to the LargeImage property of the button.
I have included a code example below.
Forgive me for any poor coding techniques.
I am only a couple months into my C# developer life.
```csharp
using Autodesk.Revit.UI;
using Autodesk.Windows;
using System.Collections.Generic;
using System.IO;
using System.Reflection;
using System.Windows.Media.Imaging;
using YourCustomUtilityLibrary;
namespace ReallyCoolAddin
{
public class StackedButton
{
public IList Create( RibbonPanel ribbonPanel )
{
// Get Assembly
Assembly assembly = Assembly.GetExecutingAssembly();
string assemblyLocation = assembly.Location;
// Get DLL Location
string executableLocation = Path.GetDirectoryName( assemblyLocation );
string dllLocationTest = Path.Combine( executableLocation, "TestDLLName.dll" );
// Set Image
BitmapSource pb1Image = UTILImage.GetEmbeddedImage( assembly, "Resources.16x16_Button1.ico" );
BitmapSource pb2Image = UTILImage.GetEmbeddedImage( assembly, "Resources.16x16_Button2.ico" );
BitmapSource pb1LargeImage = UTILImage.GetEmbeddedImage( assembly, "Resources.24x24_Button1.ico" );
BitmapSource pb2LargeImage = UTILImage.GetEmbeddedImage( assembly, "Resources.24x24_Button2.ico" );
// Set Button Name
string buttonName1 = "ButtonTest1";
string buttonName2 = "ButtonTest2";
// Create push buttons
PushButtonData buttondata1 = new PushButtonData( buttonName1, buttonTextTest, dllLocationTest, "Command1" );
buttondata1.Image = pb1Image;
buttondata1.LargeImage = pb1LargeImage;
PushButtonData buttondata2 = new PushButtonData( buttonName2, buttonTextTest, dllLocationTest, "Command2" );
buttondata2.Image = pb2Image;
buttondata2.LargeImage = pb2LargeImage;
// Create StackedItem
IList ribbonItem = ribbonPanel.AddStackedItems( buttondata1, buttondata2 );
// Find Autodes.Windows.RibbonItems
UTILRibbonItem utilRibbon = new UTILRibbonItem();
var btnTest1 = utilRibbon.getButton( "Tab", "Panel", buttonName1 );
var btnTest2 = utilRibbon.getButton( "Tab", "Panel", buttonName2 );
// Set Size and Text Visibility
btnTest1.Size = RibbonItemSize.Large;
btnTest1.ShowText = false;
btnTest2.Size = RibbonItemSize.Large;
btnTest2.ShowText = false;
// Return StackedItem
return ribbonItem;
}
}
}
```
\*\*Question:\*\* Hi Jameson, in your code above, you use a `UTILRibbonItema` class.
What is the that?
I wasn't able to find it anywhere on the Internet.
\*\*Answer:\*\* The `UTILRibbonItem` class is a helper class that I use to go find RibbonItems through the Autodesk.Windows (AW) API.
It goes in to the AW and recursively searches through the tabs, panels and buttons to find the button you feed to it.
Taking all of the logic and putting it in its own class allows for easier reuse.
A larger discussion of what that class contains can be found in the other discussion thread
on [adding a new custom ribbon panel to a Revit built-in tab](https://forums.autodesk.com/t5/revit-api-forum/add-a-new-custom-ribbon-panel-to-a-revit-built-in-tab/td-p/5538772)
Here is a basic implementation:
```csharp
using AW = Autodesk.Windows;
public AW.RibbonItem GetButton( string tabName,
string panelName, string itemName )
{
AW.RibbonControl ribbon = AW.ComponentManager.Ribbon;
foreach( AW.RibbonTab tab in ribbon.Tabs )
{
if( tab.Name == tabName )
{
foreach( AW.RibbonPanel panel in tab.Panels )
{
if( panel.Source.Title == panelName )
{
return panel.FindItem( "CustomCtrl_%CustomCtrl_%"
+ tabName + "%" + panelName + "%" + itemName,
true ) as AW.RibbonItem;
}
}
}
}
return null;
}
```
Just beware that the AW API is not a documented API, so use it at your own risk as it can be changed without letting anyone know.
Many thanks to Jameson for implementing and sharing this nice clean solution, and congratulations on such prowess "only a couple months into his C# developer life"!
#### Update on Moving a Ribbon Button Between Panels
Jameson's links above prompted me to revisit
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [adding a new custom ribbon panel to a Revit built-in tab](https://forums.autodesk.com/t5/revit-api-forum/add-a-new-custom-ribbon-panel-to-a-revit-built-in-tab/td-p/5538772)
that I referred to here on the blog in 2014 in the article
on [moving an external command button within the ribbon](https://thebuildingcoder.typepad.com/blog/2014/07/moving-an-external-command-button-within-the-ribbon.html).
I noticed that the thread was updated after the initial publication.
Above all, the link to the sample code provided back then is no longer valid, so here is
a [local copy of it, RibbonMoveExample.zip](zip/RibbonMoveExample.zip).
Thanks again to Scott for sharing it back then.
#### Revit Journal File Character Encoding
Finally, a quick note on the Revit journal file character encoding shared
by Андрей [apavlovY5SDS](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/7264445) Павлов (Andrey Pavlov) in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [Revit journal file encoding](https://forums.autodesk.com/t5/revit-api-forum/revit-journal-file-encoding/m-p/9330501):
\*\*Question:\*\* What is the Revit journal file encoding?
The default file path is \*C:\Users\ {Username} \AppData\Local\Autodesk\Revit\Autodesk Revit 2020\Journals\*.
I have trouble decoding Cyrillic characters.
\*\*Answer:\*\* Windows-1251, confidence 0.9824519, tested
with [errepi/ude](https://github.com/errepi/ude),
a C# port of the Mozilla Universal Charset Detector.
Many thanks to Andrey for raising and clarifying this issue.