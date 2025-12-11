---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 11.3
content_type: qa
optimization_date: '2025-12-11T11:44:17.287466'
original_url: https://thebuildingcoder.typepad.com/blog/2019_cefsharp_roundtrip.html
post_number: '2019'
reading_time_minutes: 10
series: general
slug: cefsharp_roundtrip
source_file: 2019_cefsharp_roundtrip.md
tags:
- csharp
- elements
- geometry
- parameters
- references
- revit-api
- sheets
- views
- walls
- windows
title: Bridge Test
word_count: 1967
---

### 3D View, Curved Section and Browser Round-Trip
Yet another RevitLookup update, full roundtrip interaction between your own instance of the built-in Revit CefSharp Chromium browser and your Revit API add-in external command, different ways to locate a BIM element, pure structural 3D view and curved section view creation, and more:
- [RevitLookup 2024.0.10](#2)
- [Calling Revit command from Chromium browser](#3)
- [Chromium browser Js round trip callback](#4)
- [Determine element location](#5)
- [Create a structural-only 3D view](#6)
- [Creating a curved section in Dynamo](#7)
- [Carbon footprint of AI image generation](#8)
- [Sending data by pigeon](#9)
- [Permaculture farm regenerates natural habitat](#10)
- [The Valley of Code](#11)
#### RevitLookup 2024.0.10
[RevitLookup 2024.0.10](https://github.com/jeremytammik/RevitLookup/releases/tag/2024.0.10) is now available with the following enhancements:
- Introducing a brand new feature: Restore window size!
Now, effortlessly you will open RevitLookup with your preferred window dimensions with a simple click
- Add `MEPSystem` and `MEPSection` support for GetSectionByIndex, GetSectionByNumber, GetElementIds,
GetCoefficient, GetPressureDrop, GetSegmentLength and IsMain
- Show System.Object option (named Root hierarchy)
- Add generic type support for the help button
- Minor tooltip changes
- Fixed search that worked in the main thread
#### Calling Revit Command from Chromium Browser
Last week, Andrej Licanin of [Bimexperts](https://bimexperts.com/sr/home) shared
a nice solution demonstrating [how to use the Revit built-in CefSharp browser in WPF](https://thebuildingcoder.typepad.com/blog/2023/11/camera-target-and-toposolid-subdivision-material.html#2).
This week he expanded on that in his contribution
on [calling Revit command from Chromium browser](https://forums.autodesk.com/t5/revit-api-forum/calling-revit-command-from-chromium-browser/td-p/12413281):
This is another guide on Chromium browser using CefSharp, a continuation
of the [simple WPF with a Chromium browser guide](https://forums.autodesk.com/t5/revit-api-forum/simple-wpf-with-a-chromium-browser-guide/td-p/12396552).
Hope someone finds it useful.
Basically, what I wanted was for a button in the browser (on a webpage) to trigger a command in Revit.
This works by "binding" a JavaScript method to a C# object and its method.
In the Javascript we `await` for the object and call its function.
So, let's make a dummy object for binding and a method in it.
In order to call a Revit method it will need a reference to an external event handler and its event:

```
   public class BoundObject
   {
     public int Add(int a, int b)
     {
       ExtApp.handler.a = a;
       ExtApp.handler.b = b;
       ExtApp.testEvent.Raise();

       return a+b;
     }
   }
```

The event and its handler are saved in the external app as `static` for ease of access:

```
  internal class ExtApp : IExternalApplication
  {
    public static IExternalApplication MyApp;
    public static ChromiumWebBrowser browser;
    public static ExternalEvent testEvent;
    public static MyEvent handler;
    public Result OnShutdown(UIControlledApplication application)
    {
      // Cef.Shutdown();
      return Result.Succeeded;
    }

    public Result OnStartup(UIControlledApplication application)
    {
      MyApp = this;
      //code for making a button

      handler = new MyEvent();
      testEvent= ExternalEvent.Create(handler);

      return Result.Succeeded;
    }
  }
```

In the WPF control, the browser is embedded like this:

```

```

Here is the code behind the window:

```
    public TestWindow()
    {
      InitializeComponent();
      ChromiumBrowser.Address = "https://www.google.com";
      ChromiumBrowser.Address = "C:\Users\XXX\Desktop\index.html";
      BoundObject bo = new BoundObject();
      ChromiumBrowser.JavascriptObjectRepository.Register("boundAsync", bo, true, BindingOptions.DefaultBinder);
    }

    public void Dispose()
    {
      this.Dispose();
    }
```

So, to use it, make an `index.html` and submit the path to it in the browser address.
The Test webpage look like this:

```
  Action 1
  Action 2
  Action 3
```

The handler code:

```
  internal class MyEvent : IExternalEventHandler
  {
    public int a;
    public int b;
    public void Execute(UIApplication app)
    {
      TaskDialog.Show( "yoyoy",
        "data is " + a.ToString()
        + " and " + b.ToString() + ".");
    }

    public string GetName()
    {
      return "YOYOOY";
    }
  }
```

#### Chromium Browser Js Round Trip Callback
Next step: round-trip callback:
To make a callback from C# function to the browser, you just need an instance of the browser, and a function in the javascript code that will be called.
Here is an edited index.html with such a function to call:

```
  Action 1
  Action 2
  Action 3
```

In our bound class, we save a instance to the browser so we can use it on command:

```
  public class BoundObject
  {
    public int aS;
    public int bS;
    internal ChromiumWebBrowser browser;

    public void CallCSharpMethod()
    {
      MessageBox.Show("C# method called!");
      // Add more code here as needed
    }
    public int Add(int a, int b)
    {
      ExtApp.handler.a = a;
      ExtApp.handler.b = b;
      ExtApp.testEvent.Raise();

      return a+b;
    }

    public int SendSomeDataFromLocal(int a)
    {
      browser.ExecuteScriptAsync("showAlert("+a.ToString()+")");
      return a;
    }
  }
```

Pass it in when creating the browser in the window codebehind:

```
  public TestWindow()
  {
    InitializeComponent();
    ChromiumBrowser.Address = "https://www.google.com";
    ChromiumBrowser.Address = "C:\Users\XXX\Desktop\index.html";
    BoundObject bo = new BoundObject();
    //ExtApp.boundObj = bo;
    bo.browser = ChromiumBrowser;
    ChromiumBrowser.JavascriptObjectRepository.Register(
      "boundAsync", bo, true, BindingOptions.DefaultBinder);
  }
```

Finally, now, you can call it from Revit:

```
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements)
  {
    ExtApp.boundObj.SendSomeDataFromLocal(999);
    return Result.Succeeded;
  }
```

This concludes a round trip from the browser and back.
I hope anyone reading this finds it useful.
#### Determine Element Location
We put together a nice little overview on various methods to determine the location of a BIM element discussing
[how can the coordinates for a Revit fabrication part be obtained with the Revit API](https://stackoverflow.com/questions/77556660/how-can-the-coordinates-for-a-revit-fabricationpart-be-obtained-with-the-revit-a)?
\*\*Question:\*\* I need to obtain the coordinates for Revit MEP FabricationParts.
All of the elements I get have a `Location` property, but not all of them have either a `LocationPoint` or a `LocationCurve`.
More specifically, I am only able to get `XYZ` values through the `LocationCurve` for `Pipe` elements.
Elements such as Threadolet, Elbow, Weld and Fishmouth don't have either a `LocationPoint` or a `LocationCurve`.
\*\*Answer:\*\* Three options that can be used on almost all BIM elements are:
- Use the `Location` property
- Retrieve the element [`Geometry` property](https://www.revitapidocs.com/2024/d8a55a5b-2a69-d5ab-3e1f-6cf1ee43c8ec.htm), e.g., calculate the centroid of all the vertices
- Use the element [`BoundingBox` property](https://www.revitapidocs.com/2024/def2f9f2-b23a-bcea-43a3-e6de41b014c8.htm), e.g., calculate its midpoint
However, for these types of `FabricationParts` specifically,
[egeer](https://stackoverflow.com/users/15534202/egeer)
and [bootsch](https://stackoverflow.com/users/21999391/bootsch) suggest
using the element's connector locations instead:
For OLets and ThreadOLets, you can use the connector that connects to the main pipe as its insertion point, since that is technically where the element was inserted:

```
    Connector insertionPointConnector = OLet.ConnectorManager
        .Connectors
        .OfType()
        .FirstOrDefault(x => x.ConnectorType == ConnectorType.Curve);

    XYZ insertionPoint = insertionPointConnector?.Origin;
```

Since their connectors are atypical in that they do not connect to another connector, but instead a curve, you need to get the one that is `ConnectorType.Curve`.
For welds, elbows and other inline elements, you can similarly use the connectors and get their origins.
If you want the center of the element, you can use vector math to calculate that using the connector's direction and location.
The direction that the connector points is the `BasisZ` property of the Connector's `CoordinateSystem`.

```
    XYZ connectorDirection = insertionPointConnector?.CoordinateSystem.BasisZ;
```

The solution I end up with is a bit different from the answer given by egeer above:
I ended up getting a Connector for each element (the ones without a `LocationCurve` or `LocationPoint`).
Here's the code in VB:

```
    Dim insertionPointConnector As Connector = CType(e, FabricationPart).ConnectorManager.Connectors.OfType(Of Connector).FirstOrDefault()
    Dim elementOrigin as XYZ = Connector.insertionPointConnector.Origin
```

`e` is of type Element.
Many thanks to egeer and bootsch for jumping in with these good solutions!
#### Create a Structural-Only 3D View
Harry Mattison continues his AU solution spree presenting a nice code sample demonstrating how
to [create a 3D view showing only Revit wall structural layers](https://boostyourbim.wordpress.com/2023/12/04/create-a-3d-view-showing-only-revit-wall-structural-layers/)
which is discussed in further depth in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [create new View3D that just displays wall layers of "Structure" function](https://forums.autodesk.com/t5/revit-api-forum/create-new-view3d-that-just-displays-wall-layers-of-quot/td-p/12344156).
Harry's sample code performs the following steps:
- Create new 3D isometric view
- Set view parts visibility `PartsVisibility.ShowPartsOnly`
- Create parts from all walls
- For each part, retrieve its built-in parameter `DPART_LAYER_INDEX`
- Convert from string to wall compound structure layer index
- Hide part if its compound structure layer function differs from `MaterialFunctionAssignment.Structure`
Many thanks to Harry for addressing this need!
#### Creating a Curved Section in Dynamo
I have heard several requests for a curved section view, e.g., Alex Vila in 2019:
[Create curved sections!](https://forums.autodesk.com/t5/revit-api-forum/create-curved-sections/m-p/8931972)
Finally, the cavalry comes to the rescue in the shape
of [Anna Baranova](https://www.linkedin.com/in/baranovaanna/), presenting a 22-minute video tutorial
on [Dynamo: Curved Sections By Line (Part 1)](https://youtu.be/Fic5BD-s3A8):

Many thanks to Anna for this nice piece of work!
#### Carbon Footprint of AI Image Generation
Researchers quantify the carbon footprint of generating AI images:
[creating a photograph using artificial intelligence is like charging your phone](https://www.engadget.com/researchers-quantify-the-carbon-footprint-of-generating-ai-images-173538174.html):
![AI image generation carbon footprint](img/ai_image_carbon_footprint.png "AI image generation carbon footprint")
#### Sending Data by Pigeon
Talking about carbon footprint and the cost and efficiency of digital data transmission, there is obviously a point at which transmission of large data can be speeded up by putting it on a storage device and moving that around rather physically than squeezing it through the limited bandwidth of the Internet:
![Send data by pigeon](img/send_data_by_pigeon.jpg "Send data by pigeon")
- There is even an RFC 1149 for this concept,
the [Standard for the Transmission of IP Datagrams on Avian Carriers](https://datatracker.ietf.org/doc/html/rfc1149).
> This memo describes an experimental method for the encapsulation of IP datagrams in avian carriers.
This specification is primarily useful in Metropolitan Area Networks.
This is an experimental, not recommended standard.
- Never underestimate the bandwidth of a station wagon full of tapes hurtling down the highway,
cf. [Wikipedia on Sneakernet](https://en.wikipedia.org/wiki/Sneakernet).
- Reminds of this thread from 2012
about [transatlantic ping faster than sending a pixel to the screen](https://superuser.com/questions/419070/transatlantic-ping-faster-than-sending-a-pixel-to-the-screen)...
#### Permaculture Farm Regenerates Natural Habitat
Hope for the future from a five-minute video [drone tour of permaculture farm](https://youtu.be/TPxJtKob7Js):

> In this video I narrate a drone tour of our entire 250-acre farm showcasing some of the swale,
dam, dugout, aquaculture, livestock food forest, cover cropping and other permaculture
systems we have on our regenerative farm.
Presented by the [Coen Farm](https://www.coenfarm.ca), who say:
> We are literally eating ourselves and our planet to death.
Our mission is to provide nutrient-dense food, feed, and permaculture education to regenerate the planet and its people.
Personally, I was very touched watching and listening to it.
#### The Valley of Code
Quick return to digital before I end for today.
If you have friends or others wanting to quickly learn to code for the web, here is a great site to get them started:
- [The Valley of Code](https://thevalleyofcode.com/)
> Welcome to The Valley of Code.
Your journey in Web Development starts here.
In the fundamentals section you'll learn the basic building blocks of the Internet, the Web and how its fundamental protocol (HTTP) works.
Toc:
- Fundamentals
- HTML and CSS
- Tools
- Deployment
- JavaScript
- TypeScript
- More CSS
- More JavaScript
- DOM and Events
- Networking
- Server Runtimes
- HTTP Servers
- Forms
- Databases
- UI libraries
- Frameworks