---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: code_example
optimization_date: '2025-12-11T11:44:15.527231'
original_url: https://thebuildingcoder.typepad.com/blog/1215_extrusion_roof.html
post_number: '1215'
reading_time_minutes: 8
series: general
slug: extrusion_roof
source_file: 1215_extrusion_roof.htm
tags:
- elements
- filtering
- levels
- python
- references
- revit-api
- rooms
- schedules
- transactions
- views
title: Events, Again, and Creating an Extrusion Roof
word_count: 1650
---

### Events, Again, and Creating an Extrusion Roof

Jon and Scott Wilson discussed some issues
[creating extrusion roofs](http://forums.autodesk.com/t5/revit-api/extrusion-roof-fails/td-p/5300295) on
arbitrary planes in the Revit API discussion forum in the past couple of days.

One neat spin-off is a simple little external command that shows the basic use of the NewExtrusionRoof method, so I took the opportunity of adding that to The Building Coder samples.

Before getting to the details of that, here is a little update on the upcoming events:

- [October hackathons and AU Germany](#2)
- [Autodesk University in Las Vegas](#3)
- [DevDays registration and motto](#4)
- [DevDay meetups](#5)
- [Presentation best practices](#6)
- [Creating an extrusion roof](#7)

#### October Hackathons and AU Germany

October will be a busy travelling month for me, participating in hackathons in Zurich, Brussels and Berlin, with the German Autodesk University in Darmstadt taking place in between as well.

The
[DevDay, meetup and hackathon event calendar](http://thebuildingcoder.typepad.com/blog/2014/08/upcoming-event-calendar.html) I
published a month ago remains valid.

#### Autodesk University in Las Vegas

As planned and announced, I am participating at Autodesk University in Las Vegas and the ADN DevDay and DevHack events preceding it.

![Autodesk University 2014](img/AU14-Speaker-275x250.png)

My class proposals were not accepted, so my main responsibility besides the DevDay and DevHack will be to lead the Revit API expert panel
[SD5156 – Open House on the Factory Floor](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=5156).

You can check the
[AU 2014 class catalogue](https://events.au.autodesk.com/connect/dashboard.ww) for
the complete list of classes.

#### DevDays Registration and Motto

The
[DevDays registration](http://autodeskdevdays.com) is
now open to ADN members. The participation is free of charge.

At several sites worldwide, you can stay on for a DevHack or an evening meetup.

The meetups are open for non-ADN members as well.

The
[list of sites](http://thebuildingcoder.typepad.com/blog/2014/08/upcoming-event-calendar.html#2) I
published a month ago remains valid.

This year's DevDays theme is **10x**, sharing how Autodesk is expanding its business and how you can join in on this 10x journey by using Cloud, Mobile and Desktop technologies to expand your user base by 10x while growing your revenue and profits.

#### DevDay Meetups

The associated meetups offer an opportunity to gather with ADN partners, Autodesk and other local technologists in your community for a social and educational evening. The focus will be on bringing 3D to the web and mobile devices. Autodesk is investing in technology to make it easy for people to bring 3D content to the web across a whole range of industries and applications in addition to design and engineering. We'll have lively discussions and sharing on where broader technologies are heading as well as how to make the web an engaging 3D experience for your customers and target users.

These meetups are open to non-ADN members, so feel free to invite your technology friends and join us for some food, beverages and engaging conversation!

You can check the locations at which evening meetups will be held by visiting
[autodeskdevdays.com/meetups](http://autodeskdevdays.com/meetups).

#### Presentation Best Practices

With all these events and conferences coming up, we will be doing a lot of presenting.

Here is a list of 33 presentation tips and tricks, pointed out by Jim Quanci, courtesy of
[Geoffrey James](http://www.inc.com/geoffrey-james/33-tricks-that-make-presentations-memorable.html),
plus another nine important suggestions specific to discussing API and source code contributed by Stephen Preston:

1. Build your presentation around the decision you want made.
2. Structure the presentation as a journey through a series of emotions.
3. Keep it simple. Your audience doesn't want or need more complexity.
4. Cut your intro down to a maximum of two sentences.
5. Cut your presentations to half as long as you originally thought they should be.
6. Include only simple graphics and highlight important data points.
7. Use a slide background that's simple with a neutral color.
8. Eliminate slides that you might need to skip due to time constraints.
9. Expunge business buzzwords from your slides (and your vocabulary).
10. Use large fonts in simple faces, like Arial.
11. Limit the distracting use of special effects and visual gimcracks.
12. Customize every presentation because every audience is different.
13. Unless it's a keynote, cut your presentation down to 20 minutes or less.
14. No matter what, rehearse your presentation at least 20 times.
15. Schedule presentations when audiences can give you their full attention.
16. Check your equipment setup well in advance.
17. Have somebody else introduce you; it creates anticipation.
18. Don't start with an apology lest you seem like a victim.
19. Don't review your company's history because nobody cares.
20. Kick off with a shocking fact, surprising insight, or unique perspective.
21. Keep your front to your audience; don't look back at the screen.
22. Be yourself; audiences immediately sense if you're not being genuine.
23. Don't fidget with papers, jewelry, glasses, or clothing.
24. Speak directly to individuals in the audience, moving from person to person.
25. Tell a story that casts your audience as the hero and you as the sidekick.
26. Use facts that are quantifiable, verifiable, memorable, and dramatic.
27. Slow down. Slow down. Slow down.
28. Never read from your slides. It irritates everyone. Never ever.
29. Be upbeat and positive but don't tell jokes unless you're a comedian.
30. Avoid hot buttons (e.g., politics) that will distract from your message.
31. Identify and emphasize the next step you want the audience to take.
32. Don't ask for extra time because you're late; instead end on schedule.
33. Have a question or two up your sleeve in case nobody has one.
34. Use a mouse if you're demoing something that requires zoom, pan and rotate (e.g. the viewer).
35. Showing function signatures or very small snippets of code on your slides is OK; don't copy large code snippets into slides – switch to the IDE instead.
36. Adjust the IDE font to be large enough to be seen from the back of the room.
37. If you're live coding, make sure you have backup code in case you make a mistake – it's really hard to spot typos in your code when 500 people are staring at you – and don't spend more than one minute trying to fix your code before going to your backup.
38. Live coding is cool, but not if its long or repetitive. Have code snippets available to copy and paste so you don't have to type everything.
39. If you're talking through the code – run it in the debugger and step through it – don't just point at static lines of code and explain what would have happened if it was running.
40. It's often a good idea to demonstrate what your finished code does before going back and showing how to write it.
41. When you're sitting at your laptop debugging code, your head tends to drop down and you voice becomes quieter. Fight this tendency by sticking a post-it note at the top of your screen that says ‘speak up' or similar.
42. Only sit behind your computer for as long as it takes to demo the code – then stand up again and move out from behind the table.

Sounds too good to be true, doesn't it?
Let's see how well we can abide by them all :-)

#### Creating an Extrusion Roof

Back to some hard-core Revit API usage.

In their discussion on
[extrusion roof failure](http://forums.autodesk.com/t5/revit-api/extrusion-roof-fails/td-p/5300295) on
arbitrary planes in the Revit API discussion forum, Jon and Scott Wilson are looking at more subtle issues.

Ignoring those for the moment, I just grabbed Jon's example code to demonstrate the basic use of the NewExtrusionRoof method, using it to add a new external command named CmdNewExtrusionRoof to The Building Coder samples.

It generates this stair-shaped element, which in actual fact really is a roof:

![NewExtrusionRoof](img/NewExtrusionRoof.png)

The completely self-contained external command Execute method mainline looks like this:

```python
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  using( Transaction tx = new Transaction( doc ) )
  {
    tx.Start( "NewExtrusionRoof" );

    RoofType fs
      = new FilteredElementCollector( doc )
        .OfClass( typeof( RoofType ) )
        .Cast<RoofType>()
        .FirstOrDefault<RoofType>( a => null != a );

    Level lvl
      = new FilteredElementCollector( doc )
        .OfClass( typeof( Level ) )
        .Cast<Level>()
        .FirstOrDefault<Level>( a => null != a );

    double x = 1;

    XYZ origin = new XYZ( x, 0, 0 );
    XYZ vx = XYZ.BasisY;
    XYZ vy = XYZ.BasisZ;

    SketchPlane sp = SketchPlane.Create( doc,
      new Autodesk.Revit.DB.Plane( vx, vy,
        origin ) );

    CurveArray ca = new CurveArray();

    XYZ[] pts = new XYZ[] {
      new XYZ( x, 1, 0 ),
      new XYZ( x, 1, 1 ),
      new XYZ( x, 2, 1 ),
      new XYZ( x, 2, 2 ),
      new XYZ( x, 3, 2 ),
      new XYZ( x, 3, 3 ),
      new XYZ( x, 4, 3 ),
      new XYZ( x, 4, 4 ) };

    int n = pts.Length;

    for( int i = 1; i < n; ++i )
    {
      ca.Append( Line.CreateBound(
        pts[i - 1], pts[i] ) );
    }

    doc.Create.NewModelCurveArray( ca, sp );

    View v = doc.ActiveView;

    ReferencePlane rp
      = doc.Create.NewReferencePlane2(
        origin, origin + vx, origin + vy, v );

    rp.Name = "MyRoofPlane";

    ExtrusionRoof er
      = doc.Create.NewExtrusionRoof(
        ca, rp, lvl, fs, 0, 3 );

    Debug.Print( "Extrusion roof element id: "
      + er.Id.ToString() );

    tx.Commit();
  }
  return Result.Succeeded;
```

You can download the most up-to-date version from
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) GitHub
repository.

The version discussed above is
[2015.0.112.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.112.0).