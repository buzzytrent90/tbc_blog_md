---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.651515'
original_url: https://thebuildingcoder.typepad.com/blog/1274_pipe_sizes.html
post_number: '1274'
reading_time_minutes: 2
series: mep
slug: pipe_sizes
source_file: 1274_pipe_sizes.htm
tags:
- csharp
- elements
- filtering
- revit-api
- mep
title: List Pipe Sizes and More Obsolete API Usage Removal
word_count: 438
---

### List Pipe Sizes and More Obsolete API Usage Removal

I was ill for a few days last week, with a fever and a virus.

I also made a quick trip to Moscow, where we held the final event in this year's sequence of DevDays conferences.

For the first time in my life, I gave my presentations in a woolly hat :-)

I also rented my first [airbnb](https://www.airbnb.com) flat ever, on Arbat street in the center of town, and was pretty happy with that.

I returned back to Switzerland on Friday, and back to health during the weekend.

During last week, I also preformed quite a bit of cleanup on the Revit API code samples on GitHub, continuing the on-going task of
[eliminating all deprecated API usage](http://thebuildingcoder.typepad.com/blog/2014/11/determining-intersecting-elements-and-continued-futureproofing.html#2) before
the methods and properties become obsolete and are completely removed.

Another enhancement was the addition of a new external command CmdListPipeSizes prompted by
[Drew's suggestion](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html?cid=6a00e553e16897883301b7c73f5a49970b#comment-6a00e553e16897883301b7c73f5a49970b) on
how to obtain a list of all pipe sizes being used in a project, based on a code snippet provided in the Revit API help file RevitAPI.chm description of the Segment class.

It retrieves all pipe segments in the project and lists their nominal, outer and inner diameter to a text file:

```csharp
  const string \_filename = "C:/pipesizes.txt";

  string FootToMmString( double a )
  {
    return Util.FootToMm( a )
      .ToString( "0.##" )
      .PadLeft( 8 );
  }

  /// <summary>
  /// List all the pipe segment sizes in the given document.
  /// </summary>
  /// <param name="doc"></param>
  void GetPipeSegmentSizes(
    Document doc )
  {
    FilteredElementCollector segments
      = new FilteredElementCollector( doc )
        .OfClass( typeof( Segment ) );

    using( StreamWriter file = new StreamWriter(
      \_filename, true ) )
    {
      foreach( Segment segment in segments )
      {
        file.WriteLine( segment.Name );

        foreach( MEPSize size in segment.GetSizes() )
        {
          file.WriteLine( string.Format( "  {0} {1} {2}",
            FootToMmString( size.NominalDiameter ),
            FootToMmString( size.InnerDiameter ),
            FootToMmString( size.OuterDiameter ) ) );
        }
      }
    }
  }
```

Running it in the Autodesk Waltham office MEP sample model *Autodesk Waltham - MEP.rvt* generates this
[pipesizes.txt output file](zip/pipesizes.txt).

As always,
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples) hosts
the most up-to-date version, including:

- [release 2015.0.116.8](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.116.8) – updated copyright year to 2015
- [release 2015.0.116.9](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.116.9) – eliminated obsolete API usage, [15 warnings left](zip/bc_migr_2015_g.txt)
- [release 2015.0.117.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.117.0) – implemented CmdListPipeSizes