---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:16.685285'
original_url: https://thebuildingcoder.typepad.com/blog/1758_camera_plasma_da4r_ai.html
post_number: '1758'
reading_time_minutes: 5
series: general
slug: camera_plasma_da4r_ai
source_file: 1758_camera_plasma_da4r_ai.md
tags:
- elements
- family
- filtering
- parameters
- revit-api
- sheets
- transactions
- views
title: Camera Plasma Da4r Ai
word_count: 1051
---

### Revit Camera Settings, Project Plasma, DA4R and AI
Once again, I have been spending way too much time answering questions in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) and
much too little time writing for The Building Coder.
So, before the end of this week, here are some interesting items I want to share with you:
- [Map Forge Viewer camera settings back to Revit](#2)
- [Project Quantum becomes Plasma](#3)
- [Mikako's DA4R overview](#4)
- [AI is affecting human game strategies](#5)
- [AI can convert speech to gesture](#6)
- [Barcelona Forge accelerator](#7)
- [Open positions at Autodesk](#8)
#### Map Forge Viewer Camera Settings back to Revit
My colleague Eason Kang invested some research
in [converting the camera state of the Forge Viewer back to the Revit model via Revit API](https://forge.autodesk.com/blog/map-forge-viewer-camera-back-revit).
![Camera mapping](img/camera_mapping.jpg)
Here is the main gist of the Revit part of his solution:
```csharp
///
/// Create perspective view with camera settings
/// matching the Forge Viewer.
/// summary>
void CreatePerspectiveViewMatchingCamera(
Document doc )
{
using( var trans = new Transaction( doc ) )
{
trans.Start( "Map Forge Viewer Camera" );
ViewFamilyType typ
= new FilteredElementCollector( doc )
.OfClass( typeof( ViewFamilyType ) )
.Cast()
.First(
x => x.ViewFamily.Equals(
ViewFamily.ThreeDimensional ) );
// Create a new perspective 3D view
View3D view3D = View3D.CreatePerspective(
doc, typ.Id );
Random rnd = new Random();
view3D.Name = string.Format( "Camera{0}",
rnd.Next() );
// By default, the 3D view uses a default
// orientation. Change that by creating and
// setting up a suitable ViewOrientation3D.
var position = new XYZ( -15.12436009332275,
-8.984616232971192, 4.921260089050291 );
var up = new XYZ( 0, 0, 1 );
var target = new XYZ( -15.02436066552734,
-8.984211875061035, 4.921260089050291 );
var sightDir = target.Subtract( position ).Normalize();
var orientation = new ViewOrientation3D(
position, up, sightDir );
view3D.SetOrientation( orientation );
// Turn off the far clip plane, etc.
view3D.LookupParameter( "Far Clip Active" )
.Set( 0 );
view3D.LookupParameter( "Crop Region Visible" )
.Set( 1 );
view3D.LookupParameter( "Crop View" )
.Set( 1 );
trans.Commit();
}
}
```
I am sure this will prove very useful for anyone aiming to precisely adjust the camera settings in a Revit perspective view.
Many thanks to Eason for his careful research and documentation!
I added this
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
in the new method [CreatePerspectiveViewMatchingCamera](https://github.com/jeremytammik/the_building_coder_samples/commit/e332d672ccf4232aee7371a29a08a494dbbf248a).
#### Project Quantum Becomes Plasma
I read an interesting discussion by [AEC Magazine](https://aecmag.com) of the past, present, potential future of Revit and the long-term Autodesk vision for BIM:
[What comes after Revit? Autodesk aims to reinvent collaborative BIM](https://aecmag.com/technology-mainmenu-35/1821-beyond-revit-autodesk-seeks-to-reinvent-collaborative-bim).
Here is a quick overview and executive summary of the contents:
In 2016, Autodesk announced Project Quantum, described as a platform technology for “evolving the way BIM works, in the era of the cloud, by providing a common data environment”.
The project then went dark but now it’s back and called Project Plasma:
- Awe inspiring
- Data Contracts and Escrow
- Quality issues
- Asynchronous vs Synchronous
- Speeding up Revit?
- Revit development
- Replacing Revit
- Typical customers
- Timeframe
- Conclusion
The 'awe inspiring' section is based on an interview with chief software architect Jim Awe.
Data contracts also mention blockchain and other, more lightweight, legally secure and binding technologies.
Interestingly, AEC Magazine surmises that 'reading between the lines, Revit’s demise is a long way off, if ever'.
I found it a very interesting read.
#### Mikako's DA4R Overview
I have been rather silent lately on the topic
of [DA4R, Forge Design Automation for Revit](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.55),
mainly due to the fact that my Forge-focussed colleagues are dealing with that more than I am, focussing on the Revit instead.
Mikako Harada now published her own overview
of [DA for Revit learning materials](https://fieldofviewblog.wordpress.com/2019/05/24/da-for-revit-learning-materials)
and [where to get help about Forge](https://fieldofviewblog.wordpress.com/2016/10/27/where-to-get-help-about-forge) that
you might want to check out.
#### AI is Affecting Human Game Strategies
I enjoyed following the development
of [AlphaGo](http://thebuildingcoder.typepad.com/blog/2017/10/au-recording-books-education-and-units.html#6)
and [AlphaZero](https://thebuildingcoder.typepad.com/blog/2019/03/ai-trends-and-yearly-deprecated-api-usage-cleanup.html#2) quite closely in the past.
My daughter Marie now pointed out an interesting book review
on [Game Changer – AlphaZero's Groundbreaking Chess Strategies and the Promise of AI](https://www.goodreads.com/review/show/2731237101) by
Matthew Sadler.
Next on my own reading list
is [Ian McEwan](https://en.wikipedia.org/wiki/Ian_McEwan)'s
[Machines like me](https://en.wikipedia.org/wiki/Machines_Like_Me_(novel)),
partially inspired by the recent 'game changing' developments in AI.
#### AI can Convert Speech to Gesture
Continuing on the topic of AI, here is an quite fascinating two-and-a-half-minute video
on [learning individual styles of conversational gesture](https://youtu.be/xzte5sobpfy) describing
an AI system that generates realistic gestures and applies them to synthesise a video from a couple of photographs and an audio recording of a person speaking:

#### Barcelona Forge Accelerator
Where am I?
I just arrived in Barcelona to participate in next week's [Forge accelerator](http://autodeskcloudaccelerator.com/forge-accelerator) here.
Looking forward very much to meeting my colleagues again and working on some inspiring and exciting new projects!
#### Open Positions at Autodesk
Autodesk is recruiting, with many open positions.
One AEC-related European one is for a [Director of Named Accounts AEC Sales – #19WD33872](https://rolp.co/ceVeg):
> The Director of Named Accounts AEC Sales, EMEA & ANZ leads a sales organization responsible for selling our portfolio of products across Autodesk Named Accounts customers in the EMEA & ANZ region. Success in this role is measured in terms of ACV growth, providing direction on account strategy, effective sales management, and execution to drive business results and meet/exceed financial and business objectives. The incumbent possesses strong sales management skills, international sales experience, and business acumen skills necessary for driving an overall AEC sales strategy in conjunction with Business Strategy & Marketing (BSM) and Product Development (PDG) groups.
If that is not up your alley, check out the numerous other positions at [autodesk.com/careers](https://www.autodesk.com/careers).