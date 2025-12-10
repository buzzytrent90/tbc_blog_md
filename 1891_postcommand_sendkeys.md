---
post_number: "1891"
title: "Postcommand Sendkeys"
slug: "postcommand_sendkeys"
author: "Jeremy Tammik"
tags: ['revit-api', 'schedules', 'sheets', 'views', 'windows']
source_file: "1891_postcommand_sendkeys.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1891_postcommand_sendkeys.html"
---

### Birthday, DevDays, PostCommand + SendKeys
Birthday in the past, DevDays in the future, and running a Revit command in the present moment:
- [Happy Birthday, Autodesk!](#2)
- [DevDays online 2021](#3)
- [For everyone](#2.1)
- [For ADN members](#2.2)
- [`PostCommand` + `SendKeys`](#4)
- [SVG tutorial](#5)
#### Happy Birthday, Autodesk!
As Shaan Hurley pointed out,
[Autodesk turned 39 years old](https://autodesk.blogs.com/between_the_lines/2021/01/autodesk-turns-39-years-old.html) on
January 30, last Saturday.
Enjoy this snapshot of the flying Autodesk founders:
![Autodesk founders](img/autodesk_founders.png "Autodesk founders")
From left to right: Rudolf Künzli, Mike Ford, Dan Drake, Mauri Laitinen, Greg Lutz, David Kalish, Lars Moureau, Richard Handyside, Kern Sibbald, Hal Royaltey, Duff Kurland, John Walker, Keith Marcelius.
Rudolf Künzli was still actively leading the Swiss office in Gundeldingen, Basel, when I first joined Autodesk in 1988.
Kern Sibbald was my direct manager when he started leading the European Technical Centre in Neuchâtel, and he didn't even realise so for over half a year, until I happened to mention the fact while chatting together on a ferry in Gothenburg :-)
#### DevDays Online 2021
Back to modern times...
just as usual, the annual DevDays Online events are taking place in the beginning of March, presented by my team, the Autodesk DAS or Developer Advocacy and Support, formerly ADN, Autodesk Developer Network.
The first week will focus on public information, mainly news related to
the [Forge platform](https://forge.autodesk.com).
The second week is for
registered [Autodesk Developer Network members](https://autodesk.com/joinadn) and
focuses mainly on our desktop products and APIs.
Please [refer to the official announcement for more complete information](https://adndevblog.typepad.com/autocad/2021/01/join-us-for-our-devdays-online-webinars.html).
All webinars start at 4pm GMT | 17h00 CET | 11am EST | 8am PST.
The sessions will be recorded, in case the timing doesn’t work for you.
Here is the schedule overview including links to register to each session:
#### For Everyone
- March 2 – [DevDays Keynotes](https://autodesk.zoom.us/webinar/register/WN_gWKcZ9miQaWPsLL8MQkn-w) – Jim Quanci
- March 3 – [Forge API update](https://autodesk.zoom.us/webinar/register/WN_unMOWlS_QIWa5zFHsToEvw) – Augusto Goncalves
- March 4 – [Autodesk Construction Cloud and API update](https://autodesk.zoom.us/webinar/register/WN_YMXE1hErSuuCysUFWyDnzQ) – Mikako Harada
#### For ADN Members
- March 9 – [The next release of AutoCAD APIs](https://autodesk.zoom.us/webinar/register/WN_-lPyJKCxSayfTqyj39FKpA) – Madhukar Moogala
- March 10 – [Revit API, Civil 3D & InfraWorks updates](https://autodesk.zoom.us/webinar/register/WN_vOr2gzcDSgKbfjQ67D7bTw)
- Civil 3D and InfraWorks update (10 minutes)
- Revit API update (45 minutes)
- March 11 – [Inventor, Vault and Fusion 360 API update](https://autodesk.zoom.us/webinar/register/WN_r6eNOmsuRgaxP9jIdZaDRA)
![Forge developers](img/devdays_2021.jpg "Forge developers")
#### PostCommand + SendKeys
Diving back into the Revit API, Yuko of shared a very nice solution in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [TwinMotion dynamic link export FBX automatically](https://forums.autodesk.com/t5/revit-api-forum/twinmotion-dynamic-link-export-fbx-automatically/m-p/10028748),
showing how to launch a built-in Revit command using `PostCommand` and then proceed to programmatically accept all its default UI setting by using `SendKeys` to simulate the user input:
\*\*Question:\*\* I have been trying to export FBX using the TwinMotion Dynamic Link.
I would like to export FBX files from many Revit files, so I would like to know how I can use `PostCommand` and then operate Windows forms on the export panel.
I tried to use `SendKeys` but I couldn't make it.
Here are the forms I need to step through:
![TwinMotion FBX export](img/twinmotion_fbx_export_1.png "TwinMotion FBX export")

![TwinMotion FBX export](img/twinmotion_fbx_export_2.png "TwinMotion FBX export")

![TwinMotion FBX export](img/twinmotion_fbx_export_3.png "TwinMotion FBX export")
I am a very new on Revit API forum. Any advice would be greatly appreciated! :-)
\*\*Answer:\*\* Welcome to the Revit API!
Unfortunately, the Revit API provides no support for the scenario you describe.
The native Windows API does provide all the required functionality to simulate any user input you like.
Therefore, you can use the Windows API to drive the required workflow.
I used such a mechanism to implement [JtClicker, a simple Windows form clicker](https://github.com/jeremytammik/JtClicker).
You can try to implement something similar for your requirements.
However, as said, that has nothing whatsoever to do with the Revit API.
\*\*Response:\*\* Thank you for your reply!
I am able to export automatically by Windows API as you showed me the example!
```csharp
void OnDialogBoxShowing(
object sender,
DialogBoxShowingEventArgs args )
{
//DialogBoxShowingEventArgs args
TaskDialogShowingEventArgs e2 = args
as TaskDialogShowingEventArgs;
e2.OverrideResult( (int) TaskDialogResult.Ok );
}
static async void RunCommands(
UIApplication uiapp,
RevitCommandId id_addin )
{
uiapp.PostCommand( id_addin );
await Task.Delay( 400 );
SendKeys.Send( "{ENTER}" );
await Task.Delay( 400 );
SendKeys.Send( "{ENTER}" );
await Task.Delay( 400 );
SendKeys.Send( "{ENTER}" );
await Task.Delay( 400 );
SendKeys.Send( "{ESCAPE}" );
await Task.Delay( 400 );
SendKeys.Send( "{ESCAPE}" );
}
public void myMacro( Document doc )
{
//Document doc = this.ActiveUIDocument.Document;
Application app = doc.Application;
UIApplication uiapp = new UIApplication(app);
try
{
RevitCommandId id = RevitCommandId
.LookupPostableCommandId(
PostableCommand.PlaceAComponent );
string name = "CustomCtrl_%CustomCtrl_%"
+ "Twinmotion 2020%Twinmotion Direct Link%"
+ "ExportButton";
RevitCommandId id_addin = RevitCommandId
.LookupCommandId( name );
if( id_addin != null )
{
uiapp.DialogBoxShowing += new
EventHandler(
OnDialogBoxShowing );
RunCommands( uiapp, id_addin );
}
}
catch
{
TaskDialog.Show( "Test", "error" );
}
finally
{
uiapp.DialogBoxShowing
-= new EventHandler(
OnDialogBoxShowing );
}
}
```
Many thanks to Yuko for sharing this nice clean solution, and congratulations for getting up to speed with the Revit API so fast!
#### SVG Tutorial
I really like the quick and compelling introduction to SVG presented by Aleksandr Hovhannisyan in
his [SVG Tutorial: How to Code SVG Icons by Hand](https://www.aleksandrhovhannisyan.com/blog/svg-tutorial-how-to-code-svg-icons-by-hand).