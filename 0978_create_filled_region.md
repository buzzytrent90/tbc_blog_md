---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:15.036774'
original_url: https://thebuildingcoder.typepad.com/blog/0978_create_filled_region.html
post_number: 0978
reading_time_minutes: 2
series: general
slug: create_filled_region
source_file: 0978_create_filled_region.htm
tags:
- csharp
- elements
- filtering
- revit-api
- views
title: Create a Filled Region to use as a Mask
word_count: 374
---

### Create a Filled Region to use as a Mask

I am off in the mountains again today, after a long break due to bad weather and too many other activities.

Meanwhile, here is a pretty old item that has been hanging around in my to-do list for much too long, so let's get it out there to you:

**Question:** How can I programmatically create a masked region?

**Answer:** I am sorry to say you currently cannot.

Hoewever, one of the API features added in Revit 2013 was the abiility to create a filled region.

The FilledRegion class was extended to offer the ability to create a new filled region, get its boundaries, and apply a linestyle to all its boundary segments.

So, while there is no straightforward method to create a mask region, the closest we can get is by creating a filled region using the FilledRegion.Create method.
However, a filled region is not the same object as a mask region.
A filled region has a fill pattern, and its type can be changed.
The mask region type cannot be changed.
If you set a filled region's fill pattern to a solid blank pattern, it will look like a mask region.

Here is a sample code fragment showing how to create a FilledRegion object:
```csharp
  FilteredElementCollector fillRegionTypes
    = new FilteredElementCollector( doc )
      .OfClass( typeof( FilledRegionType ) );

  IEnumerable<FilledRegionType> myPatterns =
      from pattern in fillRegionTypes.Cast<FilledRegionType>()
      where pattern.Name.Equals( "Diagonal Crosshatch" )
      select pattern;

  foreach( FilledRegionType frt in fillRegionTypes )
  {
    List<CurveLoop> profileloops
      = new List<CurveLoop>();

    XYZ[] points = new XYZ[5];
    points[0] = new XYZ( 0.0, 0.0, 0.0 );
    points[1] = new XYZ( 10.0, 0.0, 0.0 );
    points[2] = new XYZ( 10.0, 10.0, 0.0 );
    points[3] = new XYZ( 0.0, 10.0, 0.0 );
    points[4] = new XYZ( 0.0, 0.0, 0.0 );

    CurveLoop profileloop = new CurveLoop();

    for( int i = 0; i < 4; i++ )
    {
      Line line = Line.CreateBound( points[i],
        points[i + 1] );

      profileloop.Append( line );
    }
    profileloops.Add( profileloop );

    ElementId activeViewId = doc.ActiveView.Id;

    FilledRegion filledRegion = FilledRegion.Create(
      doc, frt.Id, activeViewId, profileloops );

    break;
  }
```

You could simulate the creation of a mask region by modifying this to find a filled region type whose fill pattern is blank.