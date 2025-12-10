---
post_number: "0846"
title: "Room Polygon and Furniture Picker in SVG"
slug: "svg_room_furnit_pick"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'levels', 'revit-api', 'rooms', 'selection', 'views', 'windows']
source_file: "0846_svg_room_furnit_pick.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0846_svg_room_furnit_pick.html"
---

### Room Polygon and Furniture Picker in SVG

I documented a number of personal and possibly naive ideas and experiments regarding
[mobile device room location](http://thebuildingcoder.typepad.com/blog/2012/09/mobile-device-room-location.html) as
a result of my previous so-called education day four weeks back.
Well, last Friday I had another one :-)

This time I took a deeper and hopefully very slightly wiser look at ideas for enabling the following kind of workflow:

- Extract relevant data from the BIM.- Populate a cloud database to provide ubiquitous access to relevant data.- Serve up relevant data in an easily consumable format, e.g. for mobile devices.- Enable interaction with the data, e.g. query and update.

I spent some time thinking about how to implement a suitable viewer with support for picking.

My colleagues
[Philippe](http://adndevblog.typepad.com/cloud_and_mobile/philippe-leefsma.html) and
[Adam](http://adndevblog.typepad.com/cloud_and_mobile/adam-nagy.html) have
developed and documented a powerful and impressive
[cloud and mobile based workflow](http://adndevblog.typepad.com/cloud_and_mobile) including
a 3D viewer implementation supporting object selection for a number of CAD authoring environments, obviously including AutoCAD, Inventor and Revit, posting data to the cloud, e.g. Azure, for consumption on various Android and iOS mobile devices, etc.

I would love to be able to achieve the same effect with significantly less effort.
[Lazy](http://programmer.97things.oreilly.com/wiki/index.php/Be_Stupid_and_Lazy)...

I already suggested that it might be possible to use
[scalable vector graphics](http://en.wikipedia.org/wiki/Scalable_Vector_Graphics) SVG
for this kind of thing last time I looked at the cloud and mobile topic.

I still think it really does provide everything I need, since its
[base functionality](http://en.wikipedia.org/wiki/Scalable_Vector_Graphics#Functionality) includes
support for interactivity, linking and metadata, among many other things.

My goal here today is to prove that SVG is a good choice for
providing, displaying, and interacting with 2D graphical data.

It is supported by all devices and requires only minimal JavaScript programming to achieve useful results, so an easily tested and extremely portable cross-platform solution seems very feasible.

Today my goal is to supply
[the proof of the pudding](http://www.phrases.org.uk/meanings/proof-of-the-pudding.html).
Specifically, I wish to prove that I can use SVG to achieve the following:

- Display a room boundary and contained items as coloured polygons using SVG.
  The items may be furniture, or any kind of family instance, e.g. simply represented by their bounding boxes.- Embed the SVG into a standard HTML page displayable on any browser, so that it can be viewed and manipulated on any mobile device.- Identify and interact with the displayed objects, e.g. identify the room or furniture and display information about a selected item.

I'll discuss the following steps towards this goal:

1. [Embed SVG in HTML](#1)- [Colour picker](#2)- [Polygons and groups](#3)- [Room and furniture picker](#4)- [Struggle with Safari](#5)

#### Embed SVG in HTML

I would very much like to show you the results of my SVG experiments right here and now on this page.

An easy way to almost achieve this would be to simply provide normal hypertext links to the different SVG web pages I describe.

A much nicer, more elegant and compelling approach is obviously to embed the SVG directly into this page, which presumably ultimately consists of HTML.

There are a number of different possibilities to
[embed SVG into HTML](http://www.w3schools.com/svg/svg_inhtml.asp),
which as always are somewhat browser dependent.

After some experimenting in the Typepad blog editor, I decided to use the embed tag, which is consequently the technique you will be enjoying further down if you read on.

All my following examples are accessible in two ways, both using a hypertext link to open up a new page presenting the SVG, and an embedded inline version that you can interact with in situ.
You can of course view this page's source in your browser to examine both methods.

#### Colour Picker

I started out searching the web for the terms "svg pick".

One hit was a pretty impressive
[SVG colour picker](http://www.kevcom.com/colorpicker) implemented
and published by Kevin Hughes in 2004 as a way to learn SVG:

![SVG Colour Picker](img/svg_color_picker.png)

It includes documentation, links to learn SVG, and additional SVG notes.

A colour picker seems like a good starting point, although I found this particular example a bit overwhelming for a project that I was hoping to finish in just a couple of hours.
I searched for something simpler instead, and turned up a presentation on
[using canvas in SVG](http://www.svgopen.org/2009/papers/12-Using_Canvas_in_SVG) by
Klaus Förster of the University of Innsbruck including a much simpler
[CSS3 colour picker](http://tirolatlas.uibk.ac.at/papers/svgopen2009/ex_colors.html) which
helped get me started implementing my very own
[colour picker](file:///C:/a/src/web/svg/color_picker_simple/color_picker_jt.svg):

Do as the man says: click on one of the little coloured rectangles at the top, and the selected colour is displayed in the bottom.

That provides pretty conclusive evidence that we really can display simple geometric shapes, interactively pick them, and react on the selection, all within a standard web page in a browser with no additional software installed, doesn't it?

How is this achieved?

This implementation requires three files: an image displaying the coloured boxes to select at the top, a JavaScript helper file, and the SVG itself.

The main component is the following SVG code, embedded into the HTML page using the embed tag (copy to a text editor to see the truncated lines in full):
```csharp
<?xml version="1.0" encoding="us-ascii" ?>
<svg xmlns="http://www.w3.org/2000/svg"
     xmlns:xlink="http://www.w3.org/1999/xlink">
<title>Jeremy's SVG color picker</title>
<script type="text/javascript" xlink:href="SVGCanvasElement.js" />
<script type="text/javascript"><![CDATA[
window.onload = function() {
  var canvas = new SVGCanvasElement(10,10,240,240);
  var context = canvas.getContext('2d');
  var image = document.getElementById('svgImage');
  var hint = document.getElementById('svgHint').lastChild;
  var active = document.getElementById('activeColor');
  var marked = document.getElementById('markedColor');

  context.drawImage(
    canvas.importImage(image),0,0,canvas.width,canvas.height
  );
  context.canvas.parentNode.style.display = 'none';

  image.onclick = function(evt) {
    var sx = evt.clientX-image.x.baseVal.value;
    var sy = evt.clientY-image.y.baseVal.value;
    var pxArr = context.getImageData(sx,sy,1,1).data;
    var fill = 'rgb('+pxArr[0]+','+pxArr[1]+','+pxArr[2]+')';
    var opac = 1.0/255.0\*pxArr[3];
    active.setAttributeNS(null,"fill",fill);
    active.setAttributeNS(null,"fill-opacity",opac);
    marked.x.baseVal.value = evt.clientX - (sx % 20);
    marked.y.baseVal.value = evt.clientY - (sy % 20);
    hint.firstChild.nodeValue = "fill='"+fill+"' fill-opacity='"+opac+"'";
  };
};
]]></script>

<image id="svgImage" x="10" y="10" width="240" height="240" xlink:href="img/svg\_css3\_colors.png" />
<rect id="activeColor" x="10" y="260" width="240" height="30" stroke="#000" fill="none" shape-rendering="crispEdges"/>
<rect id="markedColor" x="-20" y="-20" width="19" height="19" stroke="#000" fill="none" shape-rendering="crispEdges"/>
<text id="svgTipp" x="10" y="310" font-size="16px">click CSS3 color palette to pick a color</text>
<text id="svgHint" x="10" y="330" font-size="16px"><tspan></tspan><tspan>picked</tspan></text>
</svg>
```

I must admit that I did have to fiddle around considerably in the blog hosting file manager to figure out where best to place the three inter-dependant files.
The best solution, as always, was the simplest.

This colour picker is completely based on Klaus' work, with only minimal cleanup and perfectionistic improvements, such as painting a 19 by 19 rectangle around the picked colour instead of a 20 by 20 one.

Again, how does it work?
The SVG code defines five elements:

- The image element to display an image presenting all the selectable colours.- A large rectangle at the bottom to be filled with the currently selected colour.- A small 19 by 19 pixel rectangle to be moved around to mark the current colour selection inside the palette image.- A static text element prompting you to pick a colour.- A dynamic text element displaying the large rectangle's current fill and fill-opacity attribute values.

These elements are retrieved using a snippet of JavaScript code executed on loading the page.
The code also subscribes to the JavaScript onclick event.
The event handler manipulates the elements as shown.
Specifically, the getImageData function retrieves the colour and opacity of the picked point, which is used to calculate the settings and populate the various elements appropriately.

I would say that this provides a credible proof of concept.

#### Polygon and Groups

Next step: as said, I want to display a room retrieved from the building model and various items contained in it using polygons.

How do I display an arbitrary polygon in SVG?

Searching the web turned up this succinct and complete
[polygon with a hole](http://upload.wikimedia.org/wikipedia/commons/5/55/SFA_Polygon_with_hole.svg) sample
SVG file which basically says it all with no need for any further explanation at all:

It displays a quadrilateral polygon with a triangular hole, which requires just one single SVG element, and adds a group around it for identification purposes:
```csharp
  <g id="polygon" transform="translate(0.5,50.5)">
    <path d="M 35 -10 L 10 -20 15 -40 45 -45 Z M 20 -30 L 35 -35 30 -20 Z"
          style="fill:blue;fill-opacity:0.2;stroke:blue;stroke-width:1"
          fill-rule="evenodd" />
  </g>
```

I must admit that I went no further to read any documentation;
I just based all my further experiments on the following assumptions and guesswork.
Will I regret this?
It was quick and fun, anyway.
And it worked :-)

Just guessing, I assume that 'M' stands for 'move to', and 'L' stands for 'draw a line to', and 'Z' closes the polygon.
These keywords are followed by pairs of X and Y coordinates to define the polygon loop start and vertex points.
Note the 'evenodd' fill-rule attribute, which I assume leaves holes in odd-levelled interior loops.
I further assume that an interior loop within a hole is considered filled again with this setting.
Cool.

As you can try out and see for yourself in the room picker provided below, the hole is indeed not considered to be part of the polygon, and picking inside the hole will return nil.

In addition to the polygon, which is what it is all about, this neat little example also adds X and Y gridlines to simplify orientation and analysis, plus square markers for the polygon vertices, filled for the start and empty for the subsequent points.

Here is the complete SVG file implementing all this.
Note the three groups defined around the grid lines, polygon, and marker points:
```csharp
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!-- Created on 15 Mar 2011 by Mike Toews in Vim -->
<svg
  xmlns:svg="http://www.w3.org/2000/svg"
  xmlns="http://www.w3.org/2000/svg"
  version="1.1"
  width="51"
  height="51">
  <g id="background">
    <rect width="51" height="51" fill="white" />
  </g>
  <g id="grid" transform="translate(0.5,0.5)">
    <line x1="0" y1="0" x2="50" y2="0" style="stroke:#ddd;stroke-width:1"/>
    <line x1="0" y1="10" x2="50" y2="10" style="stroke:#ddd;stroke-width:1"/>
    <line x1="0" y1="20" x2="50" y2="20" style="stroke:#ddd;stroke-width:1"/>
    <line x1="0" y1="30" x2="50" y2="30" style="stroke:#ddd;stroke-width:1"/>
    <line x1="0" y1="40" x2="50" y2="40" style="stroke:#ddd;stroke-width:1"/>
    <line x1="0" y1="50" x2="50" y2="50" style="stroke:#ddd;stroke-width:1"/>
    <line x1="0" y1="0" x2="0" y2="50" style="stroke:#ddd;stroke-width:1"/>
    <line x1="10" y1="0" x2="10" y2="50" style="stroke:#ddd;stroke-width:1"/>
    <line x1="20" y1="0" x2="20" y2="50" style="stroke:#ddd;stroke-width:1"/>
    <line x1="30" y1="0" x2="30" y2="50" style="stroke:#ddd;stroke-width:1"/>
    <line x1="40" y1="0" x2="40" y2="50" style="stroke:#ddd;stroke-width:1"/>
    <line x1="50" y1="0" x2="50" y2="50" style="stroke:#ddd;stroke-width:1"/>
  </g>
  <g id="polygon" transform="translate(0.5,50.5)">
    <path d="M 35 -10 L 10 -20 15 -40 45 -45 Z M 20 -30 L 35 -35 30 -20 Z"
          style="fill:blue;fill-opacity:0.2;stroke:blue;stroke-width:1"
          fill-rule="evenodd" />
  </g>
  <g id="points" transform="translate(-1.5,48.5)">
    <rect id="outer point 1/5" width="4" height="4"
          transform="translate(35,-10)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="outer point 2" width="4" height="4"
          transform="translate(10,-20)"
          style="fill:white;stroke:blue;stroke-width:1" />
    <rect id="outer point 3" width="4" height="4"
          transform="translate(15,-40)"
          style="fill:white;stroke:blue;stroke-width:1" />
    <rect id="outer point 4" width="4" height="4"
          transform="translate(45,-45)"
          style="fill:white;stroke:blue;stroke-width:1" />
    <rect id="inner point 1/4" width="4" height="4"
          transform="translate(20,-30)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="inner point 2" width="4" height="4"
          transform="translate(35,-35)"
          style="fill:white;stroke:blue;stroke-width:1" />
    <rect id="inner point 3" width="4" height="4"
          transform="translate(30,-20)"
          style="fill:white;stroke:blue;stroke-width:1" />
  </g>
</svg>
```

#### Room and Furniture Picker

With all of this in hand, let's hand-craft a sample SVG file displaying a room and a couple of bits of furniture, allowing us to pick individual elements
, reacting to and displaying the pick results.

After the preparation above, it is surprisingly easy, actually, and impressively self-contained.
Here is
[Jeremy's room and furniture picker](file:///C:/a/src/web/svg/room_pick/room_pick.svg):

Here is the entire implementation, omitting most of the trivial grid line definition (copy and paste to an editor or view selection source to see the full truncated lines):
```csharp
<?xml version="1.0" encoding="us-ascii" ?>
<svg
  xmlns:svg="http://www.w3.org/2000/svg"
  xmlns="http://www.w3.org/2000/svg"
  version="1.1"
  width="241"
  height="201">
  <title>Jeremy's SVG Room and Furniture Picker</title>
  <script type="text/javascript">
    <![CDATA[
window.onload = function() {
  var result = document.getElementById('result');
  window.onclick = function(e) {
    var id = e.target.id;
    if(0==id.length){id="<nil>";}
    var x = e.clientX;
    var y = e.clientY;
    var s = "id='" + id + "' @ " + x + "," + y;
    result.firstChild.nodeValue = s;
  };
};
]]>
  </script>

  <g id="background">
    <rect width="300" height="200" fill="white" />
  </g>
  <g id="grid" transform="translate(0.5,0.5)">
    <line x1="0" y1="0" x2="240" y2="0" style="stroke:#ddd;stroke-width:1"/>
...
  </g>
  <g transform="translate(0.5,0.5)">
    <path id="room"
      d="M 20 20 L 220 20 220 150 150 150 150 100 20 100 Z M 160 120 L 210 120 210 140 160 140 Z"
      style="fill:blue;fill-opacity:0.2;stroke:blue;stroke-width:1"
      fill-rule="evenodd" />
  </g>
  <g id="furniture" transform="translate(0.5,0.5)">
    <rect id="table" width="100" height="20"
          transform="translate(50,50)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair1" width="8" height="10"
          transform="translate(37,55)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair2" width="10" height="8"
          transform="translate(60,75)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair3" width="10" height="8"
          transform="translate(80,75)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair4" width="10" height="8"
          transform="translate(100,75)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair5" width="10" height="8"
          transform="translate(120,75)"
          style="fill:blue;stroke:blue;stroke-width:1" />
  </g>
  <text id="result" x="20" y="180" font-size="16px">Welcome. Please click around.</text>
</svg>
```

Well, that was interesting for me at least.

A logical next step would be the implementation of a web service as described above to be populated with room and furniture data and serve it up on request in SVG and other formats.

No need for an own viewer implementation; I can use the functionality built into the browser on my mobile device.

#### Struggle with Safari

Oh no!

There I was, proudly bragging my head off about the cross-browser capabilities of SVG, and thinking I had solved all problems in the world.

I just tested my furniture picker on both an iOS iPad and an Android tablet.

Thank God it works on one of them.

Unfortunately the Safari browser does not react to the picks.

I may or may not have made some error in my SVG or JavaScript coding that Safari does not like and other browsers overlook.

It may or may not work with some other browser on iOS.

I may or may not find a solution that is better still.

Ouch, painful.
Sorry.
Still, there we are for the moment.

A happy update: I found a solution by
[Phil Archer](http://philarcher.org) of
[W3C](http://www.w3.org) of
that does work on Safari as well as my PC Firefox and Android tablet browser, a
[simple SVG in HTML test page](http://philarcher.org/diary/svgtest).

Phil says that the preferred method is to use the `object` element and define the SVG file
using the `src` attribute.

He further presents samples using `embed` and `iframe`.

All three work fine on all browsers I tested, including Safari on the iPad.

I reimplemented my room and furniture picker JavaScript code based on Phil's sample SVG.
Here is a normal hypertext link to the updated cross-country implementation, as well as inline versions using both embed and object tags.
They all work on all browsers I tested, which include Firefox on the PC, Safari on the iPad, and the Android browser:

Hypertext link:

[room\_pick\_3.svg](http://thebuildingcoder.typepad.com/files/room_pick_3-1.svg) on the web,
[room\_pick\_3.svg](file:///C:/a/src/web/svg/room_pick/room_pick_3.svg) locally.

Embed web:

Embed local:

Object:

alt : Your browser has no SVG support. Please install
[Adobe SVG Viewer](http://www.adobe.com/svg/viewer/install/) plugin (for Internet Explorer) or use
[Firefox](http://www.getfirefox.com/), [Opera](http://www.opera.com/) or
[Safari](http://www.apple.com/safari/download/) instead.

For completness' sake, here is the updated source code:
```csharp
<svg
   xmlns='http://www.w3.org/2000/svg'
   xmlns:xlink='http://www.w3.org/1999/xlink'
   onload='startup(evt)'>

<script><![CDATA[
  var SVGDocument = null;
  var SVGRoot = null;
  var count=0;
  var svgns = 'http://www.w3.org/2000/svg';
  var xlinkns = 'http://www.w3.org/1999/xlink';
  var result = null;

function startup(evt){
  O=evt.target
  SVGDoc = O.ownerDocument;
  SVGRoot = SVGDoc.documentElement;
  result = SVGDoc.getElementById('result');
  O.setAttribute("onclick","report(evt)");
}
function report(evt){
  id = evt.target.id;
  if(0==id.length){id="<nil>";}
  x=evt.clientX;
  y=evt.clientY;
  s = "id='" + id + "' @ " + x + "," + y;
  result.firstChild.nodeValue = s;
}
]]></script>

  <g id="background">
    <rect width="300" height="200" fill="white" />
  </g>
  <g id="grid" transform="translate(0.5,0.5)">
    <line x1="0" y1="0" x2="240" y2="0" style="stroke:#ddd;stroke-width:1"/>
...
    <line x1="240" y1="0" x2="240" y2="200" style="stroke:#ddd;stroke-width:1"/>
  </g>
  <g transform="translate(0.5,0.5)">
    <path id="room"
      d="M 20 20 L 220 20 220 150 150 150 150 100 20 100 Z M 160 120 L 210 120 210 140 160 140 Z"
      style="fill:blue;fill-opacity:0.2;stroke:blue;stroke-width:1"
      fill-rule="evenodd" />
  </g>
  <g id="furniture" transform="translate(0.5,0.5)">
    <rect id="table" width="100" height="20"
          transform="translate(50,50)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair1" width="8" height="10"
          transform="translate(37,55)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair2" width="10" height="8"
          transform="translate(60,75)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair3" width="10" height="8"
          transform="translate(80,75)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair4" width="10" height="8"
          transform="translate(100,75)"
          style="fill:blue;stroke:blue;stroke-width:1" />
    <rect id="chair5" width="10" height="8"
          transform="translate(120,75)"
          style="fill:blue;stroke:blue;stroke-width:1" />
  </g>
  <text id="result" x="20" y="180" font-size="16px">Welcome. Please click around.</text>
</svg>
```

Of course, if you are a real programmer, you will not trust me that the code presented above is up to date, follow the links above to the real live SVG file, and view its source instead.

Wow, that was quite a bit of extra unexpected effort.
I'm afraid my education day stretched on a little bit beyond Friday...
I hope you find it useful.

#### Immutable UV and XYZ Classes

Turning to a completely different topic:

The UV and XYZ classes were made
[immutable in Revit 2011](http://thebuildingcoder.typepad.com/blog/2010/04/xyz-immutable.html),
obviously also affecting other elements like the
[PointLoad Force and Moment properties](http://thebuildingcoder.typepad.com/blog/2010/09/immutable-pointload-force.html).

I provided some explanations for this change back then in 2010, and would now like to add the point provided by Albert Szilvasy via Kean Walmsley to explain why AutoCAD.NET
[Point2d and Point3d objects are immutable](http://through-the-interface.typepad.com/through_the_interface/2012/10/why-are-point2d-and-point3d-objects-immutable.html):
> The same reason why System.String is immutable.
>
> The problem is that a mutable type is confusing to use when it appears as a property return value. For example:
> ```csharp
> class A
> {
> public XYZ prop {get;}
> };
> A a;
> a.prop.X = 1.0;
> ```
>
> Would this syntax mean that I can set the X value of a read-only property? Or would this simply mean that I set the X value of a temporary point object returned by prop?

Just as Kean says: there you have it. :-)

Unfortunately, there are some other areas in the Revit API where this very confusion still arises from time to time.

---

# Cloud and Mobile

### Real Cross-Platform Interactive 2D Graphics Using SVG

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

Hi everybody.

This is my first post to the Cloud and Mobile DevBlog.

All I will do here is point out that I spent my education day last Friday exploring how to display and interact with 2D graphics in a browser using SVG and JavaScript by implementing a very simple
[room polygon and furniture picker in SVG](http://thebuildingcoder.typepad.com/blog/2012/10/room-polygon-and-furniture-picker-in-svg.html).

The main goal was to enable picking and identifying objects and ensuring that the same code and interaction is supported on all mobile devices, in particular both Android and Safari on iOS.

The latter added an additional little challenge at the end.

Here is the final result:

Try it out on your phone or somewhere!

I hope you find it useful.