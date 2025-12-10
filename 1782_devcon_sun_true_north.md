---
post_number: "1782"
title: "Devcon Sun True North"
slug: "devcon_sun_true_north"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'parameters', 'python', 'revit-api', 'sheets', 'views']
source_file: "1782_devcon_sun_true_north.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1782_devcon_sun_true_north.html"
---

### DevCon, Sun with True North and RVT without Revit
Please note that the European Forge DevCon in Darmstadt is looming imminent.
Furthermore, here are some notes on two recent Revit programming discussions and a pointer to some of the top-rated online classes:
- [DevCon Darmstadt](#2)
- [Personal DevCon invitation from Jim Quanci](#3)
- [Sun direction adjusted for project true north](#4)
- [Reading an RVT file without Revit](#5)
- [The top 100 free online courses](#6)
![Online course](img/online-course.png)
#### DevCon Darmstadt
Are you interested in streamlining and future-proofing your application workflow?
Integrating with the web, providing access from mobile devices, automating steps, freeing your application workflow from the desktop limitations, embracing mobile access and flexible interaction?
You can achieve all this and still make use of the full power of Revit and its API via the Forge Design Automation API.
To find out more about all aspects of Forge, hear the latest news, meet and discuss with the experts driving it, join us at the European Forge DevCon on Monday, October 14, in Darmstadt, Germany.
Check out the [agenda and all other information](https://forge.autodesk.com/devcon-2019),
including a broader program offering sessions in both AEC and Manufacturing.
You can [register here and now](https://www.rayseven.com/r7/runtime/autodesk/devcon2019/registration.visitor.php?l=en&x=1099c3252688305da2cc88dac21208ccbe8e53f5&s=input) if
you like.
DevCon Germany is an English language event taking place on the Monday directly preceding AU Germany.
The modest fee of €160 per person covers sessions, lunch, gourmet dinner, beer, wine, and juices at the evening reception.
#### Personal DevCon Invitation from Jim Quanci
Here is a personal invitation to the event from Jim Quanci, Senior Director, Forge Platform Development and Autodesk Developer Network:
This is the third European Forge DevCon. The Forge DevCon is a 'pre-conference' to Autodesk University Germany. But don't let the 'pre-conference' bit fool you. Forge DevCon is a full-fledged conference in its own right and delivered in English (unlike AU Germany THAT is mostly in German). Of course, MANY attendees will be staying ON for AU Germany, but Forge DevCon is well worth the trip on its own.
The theme for this year’s Forge DevCon is \*Digital Transformation\*. Whether you are taking your own business through a digital transformation or helping your customers through their digital transformation. We've lined up 15 classes for you and ensured there's something for everyone. They include deep technical classes for coders; roadmap, case study and business-oriented classes for business decision makers; and even a series of ‘Academia’ presentations to see how researchers are using Forge in their research efforts. See the [full agenda here](https://www.rayseven.com/r7/runtime/autodesk/devcon2019/registration.visitor.php?l=en&x=8a134f3d57581d2a5ff10fd83e2020b23c390d20&s=agenda).
What’s new to learn at the DevCon that shouldn’t be missed?
Forge Design Automation with the release of Revit, Inventor and 3ds Max _engines in the cloud_ (along with AutoCAD that is already released). What are Autodesk partners and customers doing with design automation? Automate. Automate. Automate. From automating design to automating model checking and more. Forge Model Derivative and Viewer using next generation “OTG” format. This is all about an order of magnitude increase in model size and performance. Work with your largest models in the browser at high speed and on hardware limited mobile devices (phone, tablet, AR/VR and more). New BIM 360 APIs. Model Coordination (clash), Cost, Submittals, and more.
The Forge team will be present at the conference and at your disposal to answer any of your questions, up close and personal help with your challenges. Face to face time is important when you are climbing the learning curve and driving change in your organization – so come and get face time with Autodesk decision makers and engineers – and share learnings with other developers just like you.
Convince your manager today that he needs to send you to the Forge DevCon – and then [register for the conference now](https://www.rayseven.com/r7/runtime/autodesk/devcon2019/registration.visitor.php?l=en&x=8a134f3d57581d2a5ff10fd83e2020b23c390d20&s=input).
It's only three weeks away!
Can't come to Darmstadt? Maybe we will catch in at [another great Forge DevCon venue in Las Vegas, USA](https://forge.autodesk.com/devcon-2019) on November 18th, the day before Autodesk University Las Vegas.
Questions (of any sort)? Unsure? Want to have a one-on-one meeting while I am in Darmstadt and AU?
Please do feel free to reach out to me directly.
Just [drop me an email](mailto:jim.quanci@autodesk.com),
[tweet @jimquanci](https://twitter.com/jimquanci),
call or text me.
Really.
![Jim Quanci, sailor](img/jim_quanci_sailor.png)
#### Sun Direction Adjusted for Project True North
Back to the Revit API, Mohsen Assaqqaf shared a method `GetSunDirection` for obtaining the vector of the sun taking the project True North angle into account
in [his comment](https://thebuildingcoder.typepad.com/blog/2013/06/sun-direction-shadow-calculation-and-wizard-update.html#comment-4614771756) on
the [Sun Direction and Shadow Calculation](https://thebuildingcoder.typepad.com/blog/2013/06/sun-direction-shadow-calculation-and-wizard-update.html#comment-4614771756) discussion.
I added it as follows
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
in [release 2020.0.147.8](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2020.0.147.8), cf.
the [diff to the previous version](https://github.com/jeremytammik/the_building_coder_samples/compare/2020.0.147.7...2020.0.147.8):
```csharp
#region Get Sun Direction Adjusted for Project True North
// Shared by Mohsen Assaqqaf in a comment on The Building Coder:
// https://thebuildingcoder.typepad.com/blog/2013/06/sun-direction-shadow-calculation-and-wizard-update.html#comment-4614771756
// I found that this method for getting the vector
// of the sun does not take into account the True
// North angle of the project, so I updated it
// myself using the following code:
///
/// Get sun direction adjusted for project true north
/// summary>
static XYZ GetSunDirection( View view )
{
Document doc = view.Document;
// Get sun and shadow settings from the 3D View
SunAndShadowSettings sunSettings
= view.SunAndShadowSettings;
// Set the initial direction of the sun at ground level (like sunrise level)
XYZ initialDirection = XYZ.BasisY;
// Get the altitude of the sun from the sun settings
double altitude = sunSettings.GetFrameAltitude(
sunSettings.ActiveFrame );
// Create a transform along the X axis based on the altitude of the sun
Transform altitudeRotation = Transform
.CreateRotation( XYZ.BasisX, altitude );
// Create a rotation vector for the direction of the altitude of the sun
XYZ altitudeDirection = altitudeRotation
.OfVector( initialDirection );
// Get the azimuth from the sun settings of the scene
double azimuth = sunSettings.GetFrameAzimuth(
sunSettings.ActiveFrame );
// Correct the value of the actual azimuth with true north
// Get the true north angle of the project
Element projectInfoElement
= new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_ProjectBasePoint )
.FirstElement();
BuiltInParameter bipAtn
= BuiltInParameter.BASEPOINT_ANGLETON_PARAM;
Parameter patn = projectInfoElement.get_Parameter(
bipAtn );
double trueNorthAngle = patn.AsDouble();
// Add the true north angle to the azimuth
double actualAzimuth = 2 \* Math.PI - azimuth + trueNorthAngle;
// Create a rotation vector around the Z axis
Transform azimuthRotation = Transform
.CreateRotation( XYZ.BasisZ, actualAzimuth );
// Finally, calculate the direction of the sun
XYZ sunDirection = azimuthRotation.OfVector(
altitudeDirection );
return sunDirection;
}
#endregion // Get Sun Direction Adjusted for Project True North
```
Many thanks to Mohsen for sharing this!
#### Reading an RVT File without Revit
This recurring topic now surfaced again in the StackOverflow question
on [reading RVT file](https://stackoverflow.com/questions/57936246/reading-rvt-file):
\*\*Question:\*\* When I open any .RVT file in notepad, I can search and find 'R E V I T', I tried everything to find the same text in C#, but no success to encode the .RVT file, any solution?
\*\*Answer:\*\* You may have more luck reading the text strings in Unicode encoding.
However, that will not help you decrypt very much.
To achieve more, you can read the RVT file as a structured OLE storage container.
Several different approaches are discussed in the following articles:
- [Open Revit OLE Storage](https://thebuildingcoder.typepad.com/blog/2010/06/open-revit-ole-storage.html)
- [Basic File Info and RVT File Version](http://thebuildingcoder.typepad.com/blog/2013/01/basic-file-info-and-rvt-file-version.html)
- [Custom File Properties](https://thebuildingcoder.typepad.com/blog/2015/09/lunar-eclipse-and-custom-file-properties.html#3)
- [Reading an RVT File without Revit](http://thebuildingcoder.typepad.com/blog/2016/02/reading-an-rvt-file-without-revit.html)
- [Determining RVT File Version Using Python](http://thebuildingcoder.typepad.com/blog/2017/06/determining-rvt-file-version-using-python.html)
- [Retrieve RVT Preview Thumbnail Image with Python](https://thebuildingcoder.typepad.com/blog/2019/06/accessing-bim360-cloud-links-thumbnail-and-dynamo.html#3)
#### The Top 100 Free Online Courses
You should never stop learning.
A good place to browse for fresh material and inspiration is the overview of
the [100 top free online courses of all time](https://www.freecodecamp.org/news/best-online-courses):
> Every year, Class Central releases a list of the Top Free Online Courses Of All Time, based on tens of thousands of user reviews.
This year, thanks to a growing number of reviews, the list is expanded from 50 to 100 courses.
The list covers various domains and classifies the courses into the following groups:
- Technology (23 courses)
- Business (16 courses)
- Humanities (16)
- Sciences (14 courses)
- Personal Development + Self Improvement (11)
- Health & Medicine (10 courses)
- Engineering (5)
- Language Learning (5)