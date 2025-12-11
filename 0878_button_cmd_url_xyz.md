---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.785695'
original_url: https://thebuildingcoder.typepad.com/blog/0878_button_cmd_url_xyz.html
post_number: 0878
reading_time_minutes: 8
series: geometry
slug: button_cmd_url_xyz
source_file: 0878_button_cmd_url_xyz.htm
tags:
- csharp
- elements
- python
- revit-api
- views
- windows
- geometry
title: URL and Other Buttons, XYZ Points and Vectors
word_count: 1512
---

### URL and Other Buttons, XYZ Points and Vectors

Let me summarise a very few of the topics I have been chatting with people about in the past few days:

- [Ribbon button identification](#2)
- [Opening a URL from a ribbon button](#3)
- [XYZ comparison and point and vector behaviour](#4)
- [Autodesk API overview](#5)
- [The Green Building Studio GBS REST API](#6)

Actually, to tell the truth, it is just one day, so far, this week.
Wow, my days are too full.
I am getting nothing else done!

#### Ribbon Button Identification

Here is a simple question that came up a few times in the past:

**Question:** I have implemented a ribbon push button.
Now I would like to find out which button was pushed in the event handler, i.e. external command, so that I can attach the same external command to handle multiple different buttons.
How can this be achieved, please?

I would like to identify the button pushed and access its data in the external command Execute method.

**Answer:** The official Revit API user interface design expects you to implement a separate command class for each push button.

If you do so, then you can identify the push button clicked by the user by the command class whose Execute method was triggered.

If you choose to circumvent this official approach somehow, e.g. by attaching the same external command implementation to several different push buttons, than you will have to identify them by some other means.

When you define a push button, you also specify its Name via the RibbonItemData.Name property. This name is unique across the entire ribbon, or at least within the panel containing the button, so you could use this to identify the button.

Unfortunately, the Revit API does not provide any support for finding out which push button was used to trigger the external command.

While this functionality is not provided by the Revit API, you may be able to make use of the generic .NET Framework libraries to find out.

For instance, you can
[subscribe to certain events](http://thebuildingcoder.typepad.com/blog/2011/01/subscribing-to-ui-automation-events.html) to
receive a notification whenever a ribbon button is pressed by making use of the .NET
[UI Automation](http://thebuildingcoder.typepad.com/blog/automation) library.

A concrete example of subscribing to a UI Automation event telling you which button was clicked is given by Rodolf Honke to
[pimp my ribbon](http://thebuildingcoder.typepad.com/blog/2011/02/pimp-my-autocad-or-revit-ribbon.html),
where he shows how to implement an event listener for both Revit and AutoCAD ribbon systems:

```csharp
ComponentManager.UIElementActivated
  += new EventHandler<Autodesk.Windows
    .UIElementActivatedEventArgs>(
      ComponentManager\_UIElementActivated );

void ComponentManager\_UIElementActivated(
  object sender,
  Autodesk.Windows.UIElementActivatedEventArgs e )
{
  // e.UiElement.PersistId says which item has been pressed
}
```

You will need to test whether this is of any use to you. This event listener may be pretty inefficient and slow.

The use of these APIs is unsupported and at your own risk.
If you wish to use the unsupported UI Automation option, you will have to find out for yourself how to do so using the ample information available on the web and elsewhere.

Probably the easiest way to go is indeed tom implement a separate command handler for each button.
If you have many commands requiring similar functionality, e.g. an identical initialisation procedure, you can either derive them all from one common base class, or wrap the common functionality in a method that you can call from each separate external command Execute method.

#### Opening a URL from a Ribbon Button

Another, simpler, question on hooking up ribbon buttons was also happily and rapidly resolved:

**Question:** I am attempting to create a URL link like this:

![URL ribbon button](img/url_button_1.png)

How can I enable the user to click on the button and get the link to open in a web browser?
I know how to create the ribbon panel and push button, but not how to connect to the URL in a web browser.

**Answer:** Two options:

1. Official, easy, supported: implement an external command and launch the browser from that.
2. Unofficial, harder, unsupported: use something else, e.g. the .NET framework UI Automation library discussed above.

**Response:** Option #1 worked and it was very easy indeed.
All it required was adding just one line of code in the external command, System.Diagnostics.Process.Start, and then simply implementing it in the external application file.

This enables us to populate a custom ribbon panel that includes URLs linking to PDFs and webpages containing office Revit standards.
This provides our staff a quick and easy way to access our standards within the Revit environment.

External command:

![External command opening a URL](img/url_button_2.png)

External application:

![External application setting up ribbon button](img/url_button_3.png)

Revit ribbon:

![URL ribbon button](img/url_button_4.png)

#### XYZ Comparison and Point and Vector Behaviour

A completely different topic deals with the Revit API XYZ class, its dual use to represent points and vectors, and how to compare them, raised by Graham Cook in the Revit API discussion forum in the thread on a
[XYZ.IsAlmostEqualTo problem](http://forums.autodesk.com/t5/Autodesk-Revit-API/XYZ-IsAlmostEqualTo-problem/td-p/3701684):

**Question:** Maybe I'm misinterpreting the IsAlmostEqualTo method.
I want to determine whether two points are within 4 inches of each other, i.e. 0.3333 feet.
I thought the following code would achieve that:

```
  XYZ a = new XYZ(41.7, -76, 0);
  XYZ b = new XYZ(4.7, -76, 0);
  bool almostEqual = a.IsAlmostEqualTo(b, 0.333);
```

But even though in this example the two points are 37 feet apart, IsAlmostEqualTo returns true.
Am I misunderstanding the tolerance part of the method?

**Answer:** The behaviour you observe is correct and intended and has a simple explanation:

IsAlmostEqualTo is implemented for vector comparison, not point comparison, and is mostly designed around small tolerances.
The default for IsAlmostEqualTo with no tolerance input argument given is 1e-09.
At small tolerances, points and vectors are more interchangeable, so I would use the default IsAlmostEqualTo to find "equivalent" points too.

Because this is a directional comparison, not a distance one, 0.333 for the epsilon means to check if the two vectors are within a significant angular range of each other, which the example input points actually pass.
Even if the points are treated as vectors and normalized, they will still pass the comparison because of the wide epsilon permitted.

If you want to compare points with a non-tiny epsilon using the allowed distance between them, use XYZ.DistanceTo method instead.

The Util.cs module of The Building Coder samples provides example implementations of point comparison predicate methods.
Here is
[version 2013.0.99.4](http://thebuildingcoder.typepad.com/files/bc_13_99_4.zip) from the
[slab boundary command update](http://thebuildingcoder.typepad.com/blog/2012/10/slab-boundary-revisited.html).

#### Autodesk API Overview

Autodesk provides a large number of programming platforms and APIs.
An overview is given by the
[Platform Technologies](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=5781281) overview on the Autodesk Developer Network.
Just to see and be astounded by their sheer number, here is a list of links to the associated developer pages:

- [3DS Max](http://www.autodesk.com/develop3dsmax)
- [AutoCAD Architecture](http://www.autodesk.com/developadt)
- [Autodesk Exchange Apps Developer Center](http://www.autodesk.com/developapps)
- [AutoCAD](http://www.autodesk.com/developautocad)
- [AutoCAD Civil 3D](http://www.autodesk.com/developcivil)
- [Cloud & Mobile](http://www.autodesk.com/developcloud)
- [DWF](http://www.autodesk.com/developdwf)
- [FBX](http://www.autodesk.com/developfbx)
- [Inventor](http://www.autodesk.com/developinventor)
- [AutoCAD Map 3D](http://www.autodesk.com/developmap)
- [Autodesk Infrastructure Map Server AIMS](http://www.autodesk.com/developmapguide)
- [Maya](http://www.autodesk.com/developmaya)
- [MotionBuilder](http://www.autodesk.com/developmotionbuilder)
- [Navisworks](http://www.autodesk.com/developnavisworks)
- [Revit](http://www.autodesk.com/developrevit)
- [Vault](http://www.autodesk.com/developvault)
- [Wiretap](http://www.autodesk.com/developwiretap)
- [ObjectARX](http://www.autodesk.com/objectarx)
- [AutoCAD OEM](http://www.autodesk.com/oem)
- [RealDWG](http://www.autodesk.com/realdwg)

#### The Green Building Studio GBS REST API

I recently played around with the
[BIM 360 Glue REST API](http://thebuildingcoder.typepad.com/blog/2012/12/the-bim-360-glue-viewer-and-rest-api.html)
and showed how to
[use Python to access it interactively](http://thebuildingcoder.typepad.com/blog/2012/12/bim-360-glue-rest-api-authentication-using-python.html).
I also recently mentioned the new
[building performance analysis blog](http://autodesk.typepad.com/bpa) which
discusses topics such as the
[Project Falcon](http://thebuildingcoder.typepad.com/blog/2012/12/solid-centroid-and-volume-calculation.html#0) computational
fluid dynamics Revit add-in and GBS, the Autodesk Green Building Studio cloud-based energy analysis web service.

Emile Kfouri now pointed out this discussion of the
[GBS REST API](http://autodesk.typepad.com/bpa/2013/01/green-building-studio-api-part-i.html) that
will certainly enable you (yes, you!) to achieve very cool things.
I am looking forward to hearing about your ideas!