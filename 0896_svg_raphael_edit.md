---
post_number: "0896"
title: "2D SVG Editing on Mobile Device with Raphaël"
slug: "svg_raphael_edit"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'revit-api', 'rooms', 'sheets', 'views', 'windows']
source_file: "0896_svg_raphael_edit.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0896_svg_raphael_edit.html"
---

### 2D SVG Editing on Mobile Device with Raphaël

As I mentioned last week, I took another look at displaying a 2D view of a Revit model on mobile devices using SVG to enhance the
[room polygon and furniture picker in SVG](http://thebuildingcoder.typepad.com/blog/2012/10/room-polygon-and-furniture-picker-in-svg.html).

That implementation displays a read-only view of the model, useful for picking and identifying elements, e.g. for querying or adding metadata to them.

It would also be nice to be able to edit some data graphically, e.g. enable translation and rotation of the furniture and other family instances.

The long-term goal is to implement a cloud-based round-trip 2D Revit model editing on any mobile device using SVG.

The advantage of SVG is that it enables display and edit of a 2D scene, e.g. a rendering of a Revit model, on any mobile device with no need for installation of any additional software whatsoever beyond a browser.

The parts that I still want to explore include a Revit add-in to export polygon renderings of room boundaries and other elements such as furniture and equipment to a cloud-based repository, and querying the repository from the mobile device.

To simplify editing the model, I wrapped all the SVG interaction in a JavaScript library.

After some research and comparison of different available open source SVG libraries and editors, I chose the
[Raphaël JavaScript library](#0).
I then worked through the following series of incremental implementation steps to first recreate the state I had already achieved previously, then add drag, rotate, and button support:

1. [Curves](#1)
2. [Grid](#2)
3. [Room](#3)
4. [Reporting text and click handler](#4)
5. [Furniture](#5)
6. [Drag](#6)
7. [Rotate](#7)
8. [Button](#8)
9. [Conclusion](#9)

To immediately jump into my testing playground, you can simply access
[The Building Coder SVG](http://thebuildingcoder.typepad.com/svg) folder.

#### The Raphaël JavaScript Library

[Raphaël](http://raphaeljs.com) ['ræfeɪəl] is
a small JavaScript library that simplifies work with vector graphics on the web.

Raphaël uses the SVG W3C Recommendation and VML as a base for creating graphics.
This means every graphical object is also a DOM object, so JavaScript event handlers can be attached and the objects can be dynamically modified.
The stated goal is to provide an adapter to make drawing vector art compatible cross-browser and easy.

Raphaël supports the current versions of Firefox, Safari, Chrome, Opera and Internet Explorer.

#### Curves

The first step I took was to extract one of the Raphaël demo pages, [curver.html](http://raphaeljs.com/curver.html), and host it on my own page to ensure that it has no other dependencies and continues to work in my environment:

[![The Building Coder Raphaël curves demo](img/svg_raph_01.png)](http://thebuildingcoder.typepad.com/svg/01-curves.htm)

[Curves](http://thebuildingcoder.typepad.com/svg/01-curves.htm)

It shows how curves can be drawn and hooked up with event handlers to react to dragging.

To explore this or any other of the Raphaël demo implementations, simply navigate to it and view the source.

#### Grid

Once I had that in place and up and running, my next steps dealt with reimplementing the state of the existing
[pure SVG equipment picker](http://thebuildingcoder.typepad.com/blog/2012/10/room-polygon-and-furniture-picker-in-svg.html) using
Raphaël instead.

The first step is easy, simply setting up the canvas and a rectangular grid:

[![Grid](img/svg_raph_02.png)](http://thebuildingcoder.typepad.com/svg/02-grid.htm)

[Grid](http://thebuildingcoder.typepad.com/svg/02-grid.htm)

Here is the source code HTML achieving this (copy to a text editor or view source to see the truncated lines in full):

```
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html lang="en">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ascii">
    <title>Jeremy's 2D Mobile Device Revit Model Editor Grid</title>
    <link rel="stylesheet" href="demo.css" type="text/css" media="screen">
    <link rel="stylesheet" href="demo-print.css" type="text/css" media="print">
    <script src="raphael-min-jt.js" type="text/javascript" charset="ascii"></script>
    <script type="text/javascript" charset="ascii">
      window.onload = function () {
        var w = 600, h = 400;
        var r = Raphael("holder", w, h);
        --w;
        --h;
        r.rect(0, 0, w, h, 10).attr({stroke: "#666"});
        // grid
        var d = 50, i;
        for (i = d; i < h; i += d) {
          r.path([[ "M", 0, i], ["L", w, i]]).attr({stroke: "#666"});
        }
        for (i = 50; i < 600; i += 50) {
          r.path([["M", i, 0], ["L", i, w]]).attr({stroke: "#666"});
        }
      };
    </script>
  </head>
  <body>
    <div id="holder"></div>
    <p id="copy">Jeremy Tammik, Autodesk Inc. using the <a href="http://raphaeljs.com">Raphael</a> JavaScript vector library</p>
  </body>
</html>
```

#### Room Polygon

Next, display the static room polygon on the grid:

[![Room](img/svg_raph_03.png)](http://thebuildingcoder.typepad.com/svg/03-room.htm)

[Room](http://thebuildingcoder.typepad.com/svg/03-room.htm)

Basically, we simply add the following line of code:

```csharp
// room - outer loop anti-clockwise, inner clockwise
r.path("M 40 40 L 440 40 440 300 300 300 300 200 40 200"
+ " Z M 320 240 L 320 280 420 280 420 240 Z")
.attr({stroke: "blue", fill:"lightblue"});
```

There are several different ways to implement the hole.
Note that the oddeven fill rule is applied by default and requires strict adherence to the clockwise and anti-clockwise orientation of loop vertices.
I found the hints on
[how to achieve donut holes with paths](http://stackoverflow.com/questions/3349651/how-to-achieve-donut-holes-with-paths-in-raphael) useful.

#### Reporting Text and Click Handler

The main feature of the original furniture picker was the identification and dynamic reporting of the room polygon, furniture, or equipment picked on screen.
The reporting function could of course be expanded to display or edit any other information desired, including updating some repository with modified data.

[![Text](img/svg_raph_04.png)](http://thebuildingcoder.typepad.com/svg/04-text.htm)

[Text](http://thebuildingcoder.typepad.com/svg/04-text.htm)

To achieve this in the Raphaël environment, I implemented a JavaScript event handler:

```
  jOnClick = function () {
    var s = "Id " + this.id.toString()
      + ": " + this.data("jid");
    t.attr({text:s});
  }
```

The clicked object is displayed using a new text element, and both the background rectangle and the room polygon are assigned a 'jid' identifier and subscribe to the click event to trigger the event handler:
```csharp
// room - outer loop anti-clockwise, inner clockwise
var room = r
.path("M 40 40 L 440 40 440 300 300 300 300 200 40 200 Z"
+ "M 320 240 L 320 280 420 280 420 240 Z")
.attr({stroke: "blue", fill:"lightblue"})
.data("jid", "room")
.click(jOnClick);
// reporting text
var t = r
.text(300, 380, "Welcome. Please click around.")
.attr({stroke: "#fff"});
```

#### Furniture and Equipment

Next, I added the furniture, consisting of table and chair placeholders:

[![Furniture](img/svg_raph_05.png)](http://thebuildingcoder.typepad.com/svg/05-furniture.htm)

[Furniture](http://thebuildingcoder.typepad.com/svg/05-furniture.htm)

Each one is implemented similarly to the room polygon, equipped with its own identifier, and obviously subscribing to the click event:

```
  // furniture

  r.rect(100, 100, 200, 40, 5)
    .attr({stroke: "blue", fill:"blue"})
    .data("jid", "table")
    .click(jOnClick);

  r.rect(75, 110, 16, 22, 5)
    .attr({stroke: "blue", fill:"blue"})
    .data("jid", "chair1")
    .click(jOnClick);

  r.rect(118, 150, 22, 16, 5)
    .attr({stroke: "blue", fill:"blue"})
    .data("jid", "chair2")
    .click(jOnClick);

  r.rect(158, 150, 22, 16, 5)
    .attr({stroke: "blue", fill:"blue"})
    .data("jid", "chair3")
    .click(jOnClick);

  r.rect(198, 150, 22, 16, 5)
    .attr({stroke: "blue", fill:"blue"})
    .data("jid", "chair4")
    .click(jOnClick);

  r.rect(238, 150, 22, 16, 5)
    .attr({stroke: "blue", fill:"blue"})
    .data("jid", "chair5")
    .click(jOnClick);
```

With that, we have arrived at the same stage of functionality as we already had using just pure SVG with no additional toolkit support.
Now comes the new fun stuff.

#### Drag

Raphaël supports dragging.
Here, the furniture has been dragged away from its original locations:

[![Drag](img/svg_raph_06.png)](http://thebuildingcoder.typepad.com/svg/06-drag.htm)

[Drag](http://thebuildingcoder.typepad.com/svg/06-drag.htm)

Drag event subscription requires event handlers for the drag initialisation, move and mouse up actions:

```
  // drag support

  var dragger = function () {
    this.ox = this.type == "rect" ? this.attr("x") : this.attr("cx");
    this.oy = this.type == "rect" ? this.attr("y") : this.attr("cy");
    this.animate({"fill-opacity": .5}, 500);
  },
  move = function (dx, dy) {
    var att = this.type == "rect"
      ? {x: this.ox + dx, y: this.oy + dy}
      : {cx: this.ox + dx, cy: this.oy + dy};
    this.attr(att);
    r.safari();
  },
  up = function () {
    this.animate({"fill-opacity": 1}, 500);
  },
```

To subscribe to the drag event efficiently, I reimplemented the furniture as an array that I can iterate over uniformly:

```
  // furniture

  furniture = [
    r.rect(100, 100, 200, 40, 5).data("jid", "table"),
    r.rect(75, 110, 16, 22, 5).data("jid", "chair1"),
    r.rect(118, 150, 22, 16, 5).data("jid", "chair2"),
    r.rect(158, 150, 22, 16, 5).data("jid", "chair3"),
    r.rect(198, 150, 22, 16, 5).data("jid", "chair4"),
    r.rect(238, 150, 22, 16, 5).data("jid", "chair5")
  ];

  var n = furniture.length;

  for (i = 0; i < n; ++i) {
    furniture[i].attr({stroke: "blue", fill:"blue"})
      .click(jOnClick)
      .drag(move, dragger, up);
  }
```

#### Rotate

I fiddled around for a while to implement the rotation.
Here is some dragged and rotated furniture:

[![Rotate](img/svg_raph_07.png)](http://thebuildingcoder.typepad.com/svg/07-rotate.htm)

[Rotate](http://thebuildingcoder.typepad.com/svg/07-rotate.htm)

In order to handle dragging, I ended up implementing a text element that triggers a rotation of the currently selected element.
A sibling text element reverses the rotation, i.e. rotates counter-clockwise.

The Raphaël rotation handling seems somewhat controversial, as far as I can tell from numerous discussions on the topic that I referred to.
When dragging a rotated element, I first unrotate it back to its original state.
Otherwise, the drag operation on a rotated element takes the rotation into account, causing the drag direction to differ from the visible cursor movement.

Here are the rotation handlers:

```
  // rototion support

  var current_furniture = null;

  function rotate_item (item, angle) {
    if( null != item ) {
      item.rotate( angle );
      item.data( "angle",
        item.data( "angle" ) + angle );
    }
  }

  function rotate_current_cw () {
    rotate_item( current_furniture, 5 );
  }

  function rotate_current_ccw () {
    rotate_item( current_furniture, -5 );
  }
```

Here are the two text elements used to trigger rotation of the currently selected element:

```
  // rotate selected element

  var t2 = r
    .text(500, 380, "Rotate selected item")
    .attr({fill: "#fff", "font-size": 12})
    .click( rotate_current_cw );

  var t3 = r
    .text(580, 380, "ccw")
    .attr({fill: "#fff", "font-size": 12})
    .click( rotate_current_ccw );
```

I had a problem with the rotation on the iPad: apparently, on that platform, the drag-move-up handler overrides the click handler, which initially made it impossible to click any element to select it for rotation.
The final solution I wound up with was adding functionality to the drag handler so that it handles the click event stuff as well.

#### Button

Finally, a rather trivial enhancement to clarify the usage of the text rotation elements.
I simply added a rectangle around them to make it more obvious that they are actually buttons:

[![Button](img/svg_raph_08.png)](http://thebuildingcoder.typepad.com/svg/08-button.htm)

[Button](http://thebuildingcoder.typepad.com/svg/08-button.htm)

#### Conclusion

So far I am happy with everything I have learned about both SVG and the Raphaël toolkit.

My original plan to implement a cloud-based round-trip 2D Revit model editing workflow on any mobile device using SVG still seems feasible.

One possible next step might be to implement a Revit add-in to export room boundaries and family instances representing furniture or other equipment.
For the latter, a simplistic approach might export only the bounding box, or the largest area horizontal face, or, better still, the real outline looking from above.
Another step would be implementing a cloud-based repository for this data.
It can be retrieved and displayed using SVG on the mobile device and edited as shown above.
The editing can include updating the repository data.
The original Revit model can be updated on an explicit command call, or automatically, e.g. using the Idling event, e.g. via a
[WCF service](http://thebuildingcoder.typepad.com/blog/2012/11/drive-revit-through-a-wcf-service.html).

---

# Cloud and Mobile

### 2D SVG Editing on Mobile Device with Raphaël

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

I spent some more time exploring how to display and interact with 2D graphics in a browser using SVG and JavaScript.

In my last foray into the area, I demonstrated display and interactive picking by implementing a
[room polygon and furniture picker in SVG](http://thebuildingcoder.typepad.com/blog/2012/10/room-polygon-and-furniture-picker-in-svg.html).

It enables picking and identifying objects and proves that the same code and interaction is supported on all mobile devices, in particular both Android and Safari on iOS.

I now went one step further and explored
2D SVG editing on a mobile device with Raphaël,
which adds interactive dragging and rotation of the polygons representing furniture or equipment, making use of the
[Raphaël](http://raphaeljs.com) JavaScript library.

If you would like to circumvent all explanations and just jump straight into the final result, take a look at
[The Building Coder SVG](http://thebuildingcoder.typepad.com/svg) folder,
or access the step-by-step evolution of the demo directly from here:

1. [Curves](http://thebuildingcoder.typepad.com/svg/01-curves.htm)
2. [Grid](http://thebuildingcoder.typepad.com/svg/02-grid.htm)
3. [Room](http://thebuildingcoder.typepad.com/svg/03-room.htm)
4. [Reporting text and click handler](http://thebuildingcoder.typepad.com/svg/04-text.htm)
5. [Furniture](http://thebuildingcoder.typepad.com/svg/05-furniture.htm)
6. [Drag](http://thebuildingcoder.typepad.com/svg/06-drag.htm)
7. [Rotate](http://thebuildingcoder.typepad.com/svg/07-rotate.htm)
8. [Button](http://thebuildingcoder.typepad.com/svg/08-button.htm)

[![2D SVG editing on a mobile device with Raphaël](img/svg_raph_09.png)](http://thebuildingcoder.typepad.com/svg/08-button.htm)

You can test each step interactively, then see how it works by viewing the source.

I hope you find this useful.
Have fun, and good luck!