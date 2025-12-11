---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 11.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.690235'
original_url: https://thebuildingcoder.typepad.com/blog/0830_room_loc_web_serv.html
post_number: 0830
reading_time_minutes: 22
series: general
slug: room_loc_web_serv
source_file: 0830_room_loc_web_serv.htm
tags:
- csharp
- doors
- elements
- geometry
- levels
- parameters
- references
- revit-api
- rooms
- selection
- views
- windows
title: Mobile Device Room Location
word_count: 4500
---

﻿

### Mobile Device Room Location

I had a so-called education day last Friday, and one of our current goals for these days is to learn about mobile and cloud computing.
Here is my account of activities and research that day.
Please excuse me for exposing my fallibility in this rambling account of a spontaneous unstructured single-person work-in-progress project.

The task I set myself was to query the location of a mobile device and determine the room it is located in in the Revit model.

That led to a number of interesting issues and explorations, some of which I was able to complete right away, and others to leave for another day:

1. [Scenario](#1).- [Architectural considerations](#2).- [Mobile indoor positioning](#3).- [Coordinate system and multi-user considerations](#4).- [Android location and altitude](#5).- [Scalable Vector Graphics SVG](#6).- [Azure web service](#7).- [Installing the Windows Azure SDK](#8).- [Installing ASP.NET MVC 4](#9).- [Implementing an ASP.NET web service](#10).- [Enabling a non-numerical id in the URL](#11).- [Thoughts on the web service grammar](#12).- [Adding a levels controller](#13).- [Summary and next steps](#14).

I did think of breaking this up into several posts, and cleaning up some of the results a bit more before publishing, then decided to leave it as is.

Please be aware that this is work in progress, or even less, notes on research and general head-scratching in progress.
It would be useful to discuss my ideas and results with peers before writing about them, but I am here all by my little lonesome and will simply share them with you instead.
These notes will be useful for me personally for future reference and may or may not be of interest to others.

Especially the ASP.NET results are still pretty wobbly :-)

#### 1. Scenario

Imagine the following scenario:

I am running around inside a building with a mobile device, taking notes of things.
As I take notes, I wish the device to automatically determine which room or space it is located in.
Does that make sense?

I thought of various approaches to implement this kind of room determination.

I am assuming we have a Revit model specifying the rooms and their geometry.
The room geometry could be handled as 3D volumes, or we could work with 2D boundary polygons in conjunction with the level information.

We are dealing with at least two and possibly three active agents: the Revit session on a PC somewhere, the mobile device, and potentially a web service on the cloud, accessible to both.

I am looking at the interaction between two or three agents here: the Revit session with the BIM, presumably running some custom add-ins, the mobile device and its location sensor, and optionally a web service on the cloud.
I'll call them 'Revit', 'mobile device' and 'web service' henceforth.

I have several choices to make, such as which of the three agents to set up to perform the actual calculation?
There are various pros and cons here:

- Calculate in Revit? That would be really easy, once we have the GPS data converted to the local model coordinates, because we can simply call the GetRoomAtPoint method. The hard part is to have Revit running as a ubiquitously accessible web service. There is a license issue, and even if we implemented it, we would still have to put the calculation into something that can be driven from outside, like an Idling event handler.- Calculate on the mobile device? That would require many copies of the data on each device.- Calculate in a web service?

Actually, the latter looks like the cleanest choice.

#### 2. Architectural Considerations

Let's revisit the issue from a slightly different perspective.

Here are another couple of sketches of scenarios for locating the room in a building from the mobile device location:

1. Obtain the 3D location from the mobile device and pass it to a web service which communicates real-time with a running Revit session. In the Revit session, open the appropriate document, if needed, translate the mobile device location information to a model space XYZ point, and call the Document.GetRoomAtPoint method to determine the room.- Ask the mobile device user to specify what building level she is working on. Use that to enhance the 3D location from the mobile device and pass it to a web service which communicates real-time with a running Revit session. In the Revit session, open the appropriate document, if needed, translate the mobile device location information to a model space XYZ point, and call the Document.GetRoomAtPoint method to determine the room.- Export the Revit model room 3D volumes to the cloud. On the mobile device, use the location, optionally enhanced with user-defined level data, and find the volume containing that point.- Export the Revit model room 2D polygons to the cloud in separate levels. On the mobile device, use the location to determine the 2D location, and ask the user to specify the level, then find the polygon on the appropriate level containing the given 2D point.

My current tendency is to simply export the room boundary polygons per level.

Other considerations include:

- Use 3D volume or 2D polygon plus level information?- Prompt user to manually enter current level? Yes, I think so.- Data format? Simple list of closed polygon points passed to the web service?- Data visualisation? Yes, we could easily implement a nice visualiser, e.g. using [SVG](#6).
        The web service could hold a database of polygon point data and render it to SVG on request.

At the moment, I am thinking of a room collector web service offering the following functionality:

- Add room: caller supplies room name, level, room boundary point data. Check for uniqueness, calculate bounding box and add to database, e.g. to dictionary within level dictionary.- Retrieve room: caller supplies level and mobile device location point data. Retrieve the containing room via level, bounding box analysis to speed up the search, and finally a point in polygon algorithm.- List levels: return all level names, so that they can be listed and selected on the mobile device.

Obviously this does not cover all the required functionality, so much more is still to be defined.

Anyway, it seems pretty straightforward to me to implement a mobile device room determination service using the following architecture and workflow:

- Implement a Revit add-in to export all the room polygons to the web service, separated by level.- Store the room polygon database on the cloud, e.g. using Windows Azure.- Optionally automate the storage, so that every document save updates the cloud database, as described by
      [Saikat](http://adndevblog.typepad.com/aec/2012/05/experiential-learning-with-net-web-application-with-sql-azure-tutorial.html).- Implement a web service query taking an input 3D point, optionally with a level attached, and determines the room polygon containing that point.- On the mobile device, determine the 3D location, possibly asking the user to manually define which level she is working on.- Query the web service for the room containing the current location point.- Implement room polygon visualisation: query the web service for a room or all rooms on an entire level to be rendered to SVG.

#### 3. Mobile Indoor Positioning

The
[In-Location](http://press.nokia.com/2012/08/23/accurate-mobile-indoor-positioning-industry-alliance-called-in-location-to-promote-deployment-of-location-based-indoor-services-and-solutions) Accurate
Mobile Indoor Positioning Industry Alliance was announced last month.

I guess I will go for this topic, even though it is hard to make real practical use of it immediately.

One basic question, before we even get started, is this:

**Question:** Does the mobile device location service work well enough at all to determine what room or space it is in within a building?

If so, does it always work that well, or only sometimes?

What can one do to improve precision if needed?

I asked Kean, and his reply is:

**Answer:** In my experience it’s not there yet.
Current technologies are based on proximity to Wi-Fi access points.
Best results are obtained when it’s possible to triangulate.

Currently, the best chance of solving this may be the recently founded
[In-Location accurate mobile indoor positioning industry alliance](http://press.nokia.com/2012/08/23/accurate-mobile-indoor-positioning-industry-alliance-called-in-location-to-promote-deployment-of-location-based-indoor-services-and-solutions).

Anyway, for the sake of argument and experimenting, let's assume that this issue can be solved.

If so, how can we correlate the mobile device positioning with the Revit BIM room boundaries?

#### 4. Coordinate System and Multi-user Considerations

One interesting open issue is what coordinate system to work in.

I'll use Android as the mobile device example here, leaving the iOS based ones to Adam et al.

The Android location service provides longitude and latitude via the GPS and Wi-Fi networks, so it makes sense to work in that.
The altitude is apparently given in meters as explained below.

Assuming that no two rooms or spaces can be in the same physical location, and that all the data passed in is correctly defined and positioned, that allows me to simply ignore all multi-user considerations.

I might decide to allow rooms to be identified by their unique Revit id, so that they can be told apart, and their data corrected.

Oh yes, it would also make sense to implement functionality to clear the entire database, and to delete individual levels and rooms.

#### 5. Android Location and Altitude

Retrieving a precise fix of the mobile device location is not trivial, as this discussion on
[the simplest and most robust way to get the user's current location in Android](http://stackoverflow.com/questions/3145089/what-is-the-simplest-and-most-robust-way-to-get-the-users-current-location-in-a) shows.

Obtaining the altitude is even more unreliable; in the words of
[Jonas](http://stackoverflow.com/users/213269/jonas) in
the question of
[how Android GPS Location getAltitude method works](http://stackoverflow.com/questions/2791927/how-does-getaltitude-of-android-gps-location-works):

The altitude value you get is in meters from
[the GPS (WGS84)](http://en.wikipedia.org/wiki/WGS84)
[reference ellipsoid](http://en.wikipedia.org/wiki/Reference_ellipsoid) and
not from the [geoid](http://en.wikipedia.org/wiki/Geoid).
From my experience, the GPS are really bad at altitude values.
Here is a quote from the
[GPS Status](http://eclipsim.com/gpsstatus) FAQ:

"GPS does not report the height above the mean sea level; rather the GPS system compares the height to the WGS84 reference ellipsoid which may be above or below the actual sea level.
In different parts of the earth it can be off by more than 200 meters (depending on the mass distribution of Earth).
For example the geoid's surface around Florida is above the mean sea level by a good 30-40 meters, which means that standing on the shore would show you -30m as altitude.
This is normal, and not an error, and caused by the fact that the altitude is relative to an artificial reference surface and not to the sea level.
If you are interested in this topic, I recommend reading
[Mean Sea Level, GPS, and the Geoid](http://www.esri.com/news/arcuser/0703/geoid1of3.html)."

In summary, the mobile device altitude reading might be too unreliable for locating rooms on different levels.

For an introduction to determining an Android mobile device location, here is a complete little tutorial describing the
[TrivialGPS application](http://foo.jasonhudgins.com/2007/12/cruising-around-with-android.html) which
will display a MapView and centre it on the current device location in real-time as it moves around.

For a complete video tutorial introduction to Android development and a huge number of other programming and non-programming topics, you can look at the
[New Boston](http://thenewboston.org),
with
[200 videos on Android development](http://thenewboston.org/list.php?cat=6) and
[37 on iPhone development](http://thenewboston.org/list.php?cat=28).
I had a quick look at its Android application development tutorial 142 on the
[LocationManager and location permissions](http://www.youtube.com/watch?v=Q5U2cbJNBM8).

#### 6. Scalable Vector Graphics SVG

Thinking about the room polygons, I also wondered about visualising them.

One handy way to achieve that is to use the
[Scalable Vector Graphics format SVG](http://en.wikipedia.org/wiki/Scalable_Vector_Graphics).

It is a
[W3C](http://en.wikipedia.org/wiki/World_Wide_Web_Consortium)
[open standard](http://en.wikipedia.org/wiki/Open_standard) XML-based
file format for two-dimensional vector graphics enabling a pretty compact representation of
[polygons](http://www.w3.org/TR/SVG11/shapes.html#PolygonElement),
and can be visualised on just about any browser, including
[mobile devices](http://en.wikipedia.org/wiki/Scalable_Vector_Graphics#Mobile_support).

By the way, reading the details of the SVG polygon specification, I really love the precision of the specification of its list of points, the
[grammar for points specifications](http://www.w3.org/TR/SVG11/shapes.html#PointsBNF):

The following is the Extended Backus-Naur Form (EBNF) for points specifications in 'polyline' and 'polygon' elements.
The following notation is used:

```
    *: 0 or more
    +: 1 or more
    ?: 0 or 1
    (): grouping
    |: separates alternatives
    double quotes surround literals

list-of-points:
    wsp* coordinate-pairs? wsp*
coordinate-pairs:
    coordinate-pair
    | coordinate-pair comma-wsp coordinate-pairs
coordinate-pair:
    coordinate comma-wsp coordinate
    | coordinate negative-coordinate
coordinate:
    number
number:
    sign? integer-constant
    | sign? floating-point-constant
negative-coordinate:
    "-" integer-constant
    | "-" floating-point-constant
comma-wsp:
    (wsp+ comma? wsp*) | (comma wsp*)
comma:
    ","
integer-constant:
    digit-sequence
floating-point-constant:
    fractional-constant exponent?
    | digit-sequence exponent
fractional-constant:
    digit-sequence? "." digit-sequence
    | digit-sequence "."
exponent:
    ( "e" | "E" ) sign? digit-sequence
sign:
    "+" | "-"
digit-sequence:
    digit
    | digit digit-sequence
digit:
    "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
wsp:
    (#x20 | #x9 | #xD | #xA)+
```

Cool, isn't it?
Leaves no open questions!

I'll leave SVG be for now, and probably return to it later on.
The visualisation is not an essential part of the room determination, of course.

#### 7. Azure Web Service

I installed the Windows Azure SDK for .NET to use with my Visual Studio 2010 SP1 installation.

I also found it handy to take a look at
[Saikat's experiences](http://adndevblog.typepad.com/aec/2012/05/experiential-learning-with-net-web-application-with-sql-azure-tutorial.html) with
this as well.

#### 8. Installing the Windows Azure SDK

I followed Kean's suggestions for
[exposing a web service using ASP.NET](http://through-the-interface.typepad.com/through_the_interface/2012/04/exposing-a-restful-web-service-for-use-inside-autocad-using-the-aspnet-web-api-part-1.html) and
first used the Visual Studio new project wizard to enable the Windows Azure tools:

![Azure](img/azure_1.png)

That initialises a download process:

![Azure](img/azure_2.png)

#### 9. Installing ASP.NET MVC 4

After installing the Windows Azure tools,
[Kean suggests](http://through-the-interface.typepad.com/through_the_interface/2012/04/exposing-a-restful-web-service-for-use-inside-autocad-using-the-aspnet-web-api-part-1.html) installing
[MVC 4 beta for Visual Studio 2010](http://asp.net/mvc/mvc4).

Finally, my Visual Studio installation is enhanced like this:

![Azure](img/azure_3.png)

#### 10. Implementing an ASP.NET Web Service

I created a new ASP.NET MVC 4 Web Application using the wizard presented by Visual Studio > New > Project > New Project dialogue.

I compiled it and hit F5 to debug it in the browser, which shows the default web-page provided in the boilerplate project:
![ASP.NET default home page](img/asp_net_default_home.png)

I have done nothing yet to implement any API, but I can still provide some input data and use the debugger to explore where it ends up in my new project.

Simply replace the original URL displayed in the address bar, in my case 'http://localhost:1574', with an address including some input data, such as 'http://localhost:1574/api/values'.
This triggers a breakpoint in the ValuesController Get method taking no arguments:
```csharp
  // GET api/values
  public IEnumerable<string> Get()
  {
    return new string[] { "value1", "value2" };
  }
```

The result returned to the browser is an XML document containing the list of predefined values:

![ASP.NET default value list](img/asp_net_default_values.png)

Providing an integer id as an additional argument, i.e. in this case as an additional node in the URL path, e.g. 'http://localhost:1574/api/values/12345', triggers the breakpoint in the next overload of the Get method, taking an integer input argument:
```csharp
  // GET api/values/5
  public string Get( int id )
  {
    return "value";
  }
```

This time, the result is an XML document containing the one single value specified:

![ASP.NET default specific value](img/asp_net_default_value.png)

I cannot specify an arbitrary string argument, such as 'jeremy', since the default implementation expects integer id values only:

![ASP.NET invalid value](img/asp_net_invalid_value.png)

#### 11. Enabling a Non-numerical Id in the URL

Realising that it will be hard for me to encode the mobile device location into a simple integer, I worked really hard to find a way to pass in a string via the URL.

After extensive research, I finally found a solution in two simple steps:

1. Specify non-standard constraints in the WebApiConfig implementation:
```csharp
  public static class WebApiConfig
  {
    public static void Register( HttpConfiguration config )
    {
      config.Routes.MapHttpRoute(
        name: "DefaultApi",
        routeTemplate: "api/{controller}/{id}",
        defaults: new { id = RouteParameter.Optional },
        constraints: new { id = "[0-9a-z]+" }
      );
    }
  }
```

This important solution is not explicitly pointed out, only sort of hinted at in the article by Mike Wasson on
[ASP.NET routing and action selection](http://www.asp.net/web-api/overview/web-api-routing-and-actions/routing-and-action-selection),
where it just says that you can "provide constraints, which restrict how a URI segment can match a placeholder".
Anyway, this modification finally allows me to enter non-numerical characters in the 'id' part of the URL.

2. The modified constraints triggered an error when executing the application, however, because my ApiController implementation only provides a Get method taking an int argument.

I removed that and replaced it by one taking a string instead:
```csharp
  public string Get( string id )
  {
    return string.Format(
      "The room containing the input location "
      + "'{0}' is still being determined...",
      id );
  }
```

While making these modifications, I also renamed my controller from ValuesController to RoomController.

Now I can pass in a string through the URL and receive the expected result:
![ASP.NET string value in URL](img/asp_net_string_value.png)

I don't know whether I ever previously needed such a long time to learn how to change such a small number of lines of code.

Anyway, now I have more control over my URL argument passing, and can start thinking about an effective format to pass in the mobile device location value and other data.

#### 12. Thoughts on the Web Service Grammar

Please excuse my rather unstructured gung-ho approach.
I am thinking about what to implement and how to do so at the same time.
Sorry about that.
Here goes anyway.
More elegantly, I might also simply say that this is work in progress, on all levels.

Now that we have some infrastructure in place, let me take a step back again and consider what I really want to achieve.
Basically, I want to query the system for the room containing a given location.
In order to do so, I also need to be able to first specify the room data.
Let's take a look at the input required for that.

Each room resides on a level.
I could specify the levels anew each time I specify a room.
A cleaner approach would be to first specify all the levels, supplying data such as their name and elevation, and returning a simple integer id for each.
Then I can use the level id when defining the room, which would help avoid a lot of level data repetition.

Once defined, it would also be nice to be able to query the system for the data it contains.
It might be useful to retrieve lists of all levels and rooms.
For each room, the room polygons could be returned as a list of points.

We could implement rendering to SVG, e.g. given an individual room or an entire level, return an SVG file displaying all its room polygons.

And of course the most basic functionality that this whole exercise is targeted at: given a specific level and location point, return the room containing it.

How can we define a grammar to support this functionality?

Here is a preliminary draft of a suggestion using the following notation:
I don't really like those horrible % sign escape codes commonly used in URL arguments.

- I – an integer number, e.g. 342.- L – an integer level id, i.e. the same as I.- R – a real number, e.g. 1.234. Might be encoded in the URL as 1p234.- E – an elevation, i.e. the same as R.- P – a 3D point, i.e. a comma-separated list of three real numbers, e.g. "R,R,R"; to encode this in a URL, we might want to use something like "RcRcR", i.e. "1p23c4p56c7p89".

With that notation, we might consider operations such as the following:

- levels – list all levels with their id, name and elevation.- rooms – list all rooms with their level, id and name.- rooms-L – list all rooms on the given level specified by id integer id N.- level-add-name-elevation- room-add-L ...

Actually, I will not even finish this list, just go ahead and start implementing some more right away.
Maybe I'll get around to completing and cleaning it up once I know better what I really want.

#### 13. Adding a Levels Controller

In starting to write the grammar and list of actions above, I immediately realise that it would be nice to have more than one controller, so that I can specify different more or less meaningful text snippets in my URL command.

At least I am pretty sure that I would like to be able to use the word 'levels' as well as 'room' or 'rooms', so let's add a LevelsController and see whether that works.

Here are the steps:

- Visual Studio solution explorer > RoomPolygon project > Controllers > right click for context menu > Add > Controller... > LevelsController.- Change the parent class from 'Controller' to 'ApiController'.- Replace the default Controller method Index by the ApiController methods Get, Post, Put and Delete, as we have them in the RoomController, which was based on the default whatever-it-was controller in the new project template.

![Adding a LevelsController](img/asp_net_levels_controller.png)

Very simple.

I am not doing everything the way the system expects, though, because my Get() method with no arguments is never called, and trying to call the service with a URL ending in just "api/levels" causes an error.
I must specify an id, maybe because my constraints require a non-empty string for the id.

I implemented code in my Get method taking the id argument to respond to the id "all" with a list of all levels.

Here is the resulting code with a dummy level class definition and a list of dummy level entries to play with:
```csharp
  class Level
  {
    public string Name { get; set; }
    public double Elevation { get; set; }

    public Level( string name, double elevation )
    {
      Name = name;
      Elevation = elevation;
    }

    public override string ToString()
    {
      return Name + ", "
        + Elevation.ToString( "0.###" );
    }
  }

  public class LevelsController : ApiController
  {
    static List<Level> \_levels = new List<Level>()
    {
      new Level ( "Level 1", 12.345 ),
      new Level ( "Level 2", 67.89 )
    };

    // GET api/levels
    public IEnumerable<string> Get()
    {
      return new string[] {
        "1: Level 1, 12.345",
        "2: Level 2, 67.89" };
    }

    public string Get( string id )
    {
      if( id.ToLower().Equals( "all" ) )
      {
        return Get().Aggregate<string>(
          ( current, next )
            => current + ", " + next );
      }
      try
      {
        int i = int.Parse( id );
        if( 0 > i )
        {
          return string.Format(
            "Invalid negative level id '{0}'",
            id );
        }
        if( \_levels.Count <= i )
        {
          return string.Format(
            "Invalid out-of-range level id '{0}'",
            id );
        }
        return \_levels[i].ToString();
      }
      catch( Exception ex )
      {
        return string.Format(
          "Invalid level id '{0}': {1}",
          id, ex.Message );
      }
    }

    // POST api/levels
    public void Post( [FromBody]string value )
    {
    }

    // PUT api/levels/5
    public void Put( int id, [FromBody]string value )
    {
    }

    // DELETE api/levels/5
    public void Delete( int id )
    {
    }
  }
```

With that in place, I can now run my ASP.NET RoomPolygon application in the debugger and provide test URLs to receive the following XML results:

- http://localhost:1574/api/levels/all –
  ```csharp
  <string>1: Level 1, 12.345, 2: Level 2, 67.89</string>
  ```- http://localhost:1574/api/levels/1 –
    ```csharp
    <string>Level 2, 67.89</string>
    ```

I should probably revisit the ASP.NET tutorial, though, and understand the implementation of the Get method with no argument, as well as the Post, Put and Delete methods.

#### 14. Summary and Next Steps

I completed my education day working through these thirteen (rather arbitrary) steps.
Let's stop at this special number and consider what has been achieved.

We have painted an interesting scenario, albeit possibly a little bit ahead of its time.

We have considered some architectural underpinnings and possible implementation approaches, and looked at some possible interesting bits and pieces of infrastructure and support, such as SVG.

We have installed support for and created an MVC 4 Azure web service, and made some useful modifications to it as a proof of concept for populating it with data from the Revit BIM as well as querying the service from the mobile device.

What to do next?

Well, we still have not implemented anything at all to run on the mobile device, so that is an important next step.

We need to implement the Revit add-in to call the web service and populate the level and room information in the cloud.

The web service itself needs to be implemented to receive the data from the BIM, save it in efficient internal data structures or a database, and reply to queries from the mobile device, possibly including requests for a graphical representation, potentially based on SVG.

As hinted, I am not completely happy with my understanding of the ASP.NET functionality yet.
I need to revisit that, and maybe restart from scratch with a better understanding.

It would help to first understand it workings in more depth, plan the full required functionality and how it should be made accessible, and only then start implementing :-)

Oh yes, the web service needs to be deployed to the cloud, as well.

Looks like I have enough material lined up for another couple of interesting education days.

Before I end, here is the initial
[version 0.0.1.0](zip/RoomPolygon_1.zip) of
my skeleton web service implementation, in case it is of interest to anyone.

It handles the grammar outlined above but does not execute any actions on the requests yet, beyond echoing them in the XML strings returned.