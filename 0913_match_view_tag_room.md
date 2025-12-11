---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: qa
optimization_date: '2025-12-11T11:44:14.876351'
original_url: https://thebuildingcoder.typepad.com/blog/0913_match_view_tag_room.html
post_number: 0913
reading_time_minutes: 4
series: views
slug: match_view_tag_room
source_file: 0913_match_view_tag_room.htm
tags:
- csharp
- elements
- family
- filtering
- revit-api
- rooms
- views
title: Rename View by Matching Elevation Tag with Room
word_count: 704
---

### Rename View by Matching Elevation Tag with Room

Today is the last morning meeting with my European DevTech colleagues here in Brittany, and time to travel back to Switzerland.

Before leaving, here is a useful real-world productivity tool by Trevor Taylor of ZGF,
[Zimmer Gunsul Frasca Architects LLP](http://www.zgf.com),
with his own description of the task and its solution:

The task I want to address is to match interior elevation tags with the rooms they fall inside.

This is used to track back and rename the corresponding views.

Naming interior elevation views is a major time-burner as there can be thousands of views in a project to rename, and they have to be coordinated when the room numbers or names change.
If we don’t tie view names to rooms, we have no hope of ever keeping track of so many views.

I originally attacked this by trying to determine the location of the view tags in the model, but had to give up on that one.

Ben Bishoff of [Ideate](http://www.ideate.com) gave me the idea to approach the problem from the view rather than from the interior elevation tag, so I worked backwards from the view cropbox property using the RevitLookup app and arrived at this simple solution:

**1.** Collect all views of class ‘ViewSection’, then filter down to those of ‘Interior Elevation’ type:

```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( m\_doc )
      .OfClass( typeof( ViewSection ) );

  foreach( ViewSection current in collector )
  {
    ViewFamilyType vft = m\_doc.GetElement(
      current.GetTypeId() ) as ViewFamilyType;

    if( null != vft
      && vft.Name == "Interior Elevation" )
    {
      m\_int\_elevation\_views.Add( current );
    }
  }
```

**2.** Construct a point at the midpoint of the front bottom edge of each interior elevation view's CropBox.
The CropBox is conveniently relative to the projection plane of the view.
X is to right, Y is up, and Z is the view depth:
```csharp
  // Construct a point at the midpoint of the
  // front bottom edge of the elev view cropbox

  double xmax = v.CropBox.Max.X;
  double xmin = v.CropBox.Min.X;
  double zmax = v.CropBox.Max.Z;

  XYZ pt = new XYZ(
    xmax - 0.5 \* ( xmax - xmin ),
    1.0,
    zmin );
```

**3.** Get pt's translation to project's coordinate system:
```csharp
  // Get pt's translation to
  // project coordinate system

  pt = v.CropBox.Transform.OfPoint( pt );
```

That takes care of the heavy lifting. Pass pt to the GetRoomAtPoint method and you obtain the room, if there is one to get:

**4.** Retrieve the room:
```csharp
  Autodesk.Revit.DB.Architecture.Room rm
    = m\_doc.GetRoomAtPoint( pt );
```

It does exactly what I want it to now and will save our teams countless hours on large projects by organizing the names of interior elevation views by room name:

![Elevation renaming map](img/elevation_renaming_map.png)

Here is a
[complete sample project](zip/interior_elevation_view_organizer_2.zip) including a test model in case you’d like to check it out yourself.

Many thanks to Trevor for this useful tool, his research, implementation, and generous sharing.

Before I sign off, here are two other nice pointers, not related to Revit:

#### Au Bout du Monde and Super-Sonic Stereo

Brittany is at the end of the earth, or Finisterre, au bout du monde, and I have seen many restaurants, a parking place, and a number of other establishments named after that around here.

That reminded me of one of my favourite humorous YouTube films, a Russian cartoon also named
[Au Bout du Monde](http://www.youtube.com/watch?v=jsoKbk6GXQc),
by Konstantin Bronzit:

It was very fitting to share with my colleagues, since two of us are Russian as well.
We enjoyed it a lot, and I hope you like it as much as I do.

While we are at it, yesterday, my son Christopher pointed out another quite nice blog with explanations of what-if questions, the most recent one featuring sound sources, asking
[what if my stereo flew around at super-sonic speed?](http://what-if.xkcd.com/37)
I enjoyed that as well, and so might you.

Last but not least, today is the first day of spring and
[vernal equinox](http://en.wikipedia.org/wiki/Equinox)!

![Spring and vernal equinox](img/anoixi.jpeg)

Off we go to the airport...