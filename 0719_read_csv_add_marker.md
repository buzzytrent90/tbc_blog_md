---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:14.459341'
original_url: https://thebuildingcoder.typepad.com/blog/0719_read_csv_add_marker.html
post_number: 0719
reading_time_minutes: 3
series: general
slug: read_csv_add_marker
source_file: 0719_read_csv_add_marker.htm
tags:
- csharp
- family
- references
- revit-api
title: Add Reference Points Read From CSV File
word_count: 517
---

### Add Reference Points Read From CSV File

For your convenience and my own future reference, here is a summary of a recent
[conversation](http://forums.autodesk.com/t5/Autodesk-Revit-API/reading-a-csv-file-code-evolved/td-p/3303833) with
Dermott Mcmeel and a pointer to his
[video of the resulting add-in in action](http://vimeo.com/36167862).

**Question:** I'm using this
[wicked tutorial](http://nmillerarch.blogspot.com/2010/01/streaming-grasshopper-points-into-revit.html)
as a basis for a plug-in that will read info out of a CSV file.
However, it's a few years old and from what I gather some of the definitions have been redefined our outdated.
I've done the 'First Revit API plug-in tutorial' and am stoked about my success.
But I'm new to VB and C#, so I was wondering if anyone has updated info?
Or could point out changes and where updated info might be found?

**Answer:** I converted the
[VB code to read a CSV file and generate Revit reference points](http://nmillerarch.blogspot.com/2010/01/streaming-grasshopper-points-into-revit.html) to
C# for you:
```csharp
  string filepath = "insert file path here";

  if( File.Exists( filepath ) )
  {
    StreamReader s = new StreamReader(
      filepath );

    while( -1 != s.Peek() )
    {
      string line = s.ReadLine();

      string[] data = line.Split(
        new char[] { ',' } );

      XYZ p = app.Create.NewXYZ(
        double.Parse( data[0] ),
        double.Parse( data[1] ),
        double.Parse( data[2] ) );

      ReferencePoint rp = doc.FamilyCreate
        .NewReferencePoint( p );
    }
  }
```

**Response:** The code is very much appreciated.
I am however having some trouble pasting it into the right context...

I've tried some remedial trouble shooting, but as I fix one error it creates another one.

I pasted the code straight into a blank C# class library template (as per Revit API tutorial 1) but get lots of errors – I assume I'm making a stupid mistake in the syntax but can't figure it out.
Any help appreciated... and again thanks for the helping hand.

**Answer:** Please work through one of the getting started tutorials first, before even thinking about CSV:

- [Getting started with the Revit API](http://thebuildingcoder.typepad.com/blog/2011/10/getting-started-with-the-revit-2012-api.html)- [Preparing for a hands on Revit API training](http://thebuildingcoder.typepad.com/blog/2012/01/preparing-for-a-hands-on-revit-api-training.html)

Then you will have a reliable working skeleton in place and almost certainly gotten a better hang of it.

**Response:** Thanks for all your help!

See the results:

[Revit API for CSV reading](http://vimeo.com/36167862) from [Dermott Mcmeel](http://vimeo.com/user2916257) on [Vimeo](http://vimeo.com).

**Answer:** For the sake of completeness, here is
[ReadCsvPoints.zip](zip/ReadCsvPoints.zip) containing
Jeremy's C# version (and complete Visual Studio solution with add-in manifest) of an add-in to read point data from a CSV file and generate Revit reference points from it, and a snapshot of Dermott's
[WIP C# code](zip/ReadCsvPointsDm.cs) (work in progress).

Many thanks to Dermott for the interesting topic, his research, and sharing the code!