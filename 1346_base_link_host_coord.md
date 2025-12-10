---
post_number: "1346"
title: "Internal vs. Base Point and Link to Host Coordinates"
slug: "base_link_host_coord"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api']
source_file: "1346_base_link_host_coord.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1346_base_link_host_coord.html"
---

### Internal vs. Base Point and Link to Host Coordinates

Let's look at two questions on various coordinate systems raised by Simon Jones and Miroslav Schonauer:

- [Translate coordinates from link to host](#2)
- [Project Base Point versus Project Internal](#3)

#### Translate Coordinates from Link to Host

**Question:**
Take this snippet of code:

```csharp
  foreach (RevitLinkInstance lnk in selectedLinks)
  {
    Document doc = lnk.GetLinkDocument();
    mDocumentList.Add(doc);
    projectLocation = doc.ActiveProjectLocation;
    XYZ origin = newXYZ(0, 0, 0);

    ProjectPosition position = projectLocation
      .get\_ProjectPosition(origin);

    LocationPoint linkLocationPt = lnk.Location
      as LocationPoint;

    Location loc = lnk.Location;

    SiteLocation siteLocation = doc.SiteLocation;
  }
```

I'm trying to determine the position of elements within a link in relation to the host file for which I'm assuming I need the link's location and rotation (perhaps I'm still thinking too much like AutoCAD with the insertion and rotation of an xref).

However, the LocationPoint is always coming back as zero.

What is it I need to use to translate coordinates from the link to the host?

**Answer:**
Nice problem.

Does the discussion on
[determining host document location of a linked element](http://thebuildingcoder.typepad.com/blog/2013/11/determining-host-document-location-of-a-linked-element.html) help?

**Response:**
Thanks – that did the trick.

The key function was:

```csharp
  Transform t = lnk.GetTotalTransform();
```

Then the points can be transformed to the host's coordinates with:

```csharp
  pt1 = t.OfPoint( pt1 );
```

Seems to be working fine   :-)

#### Project Base Point versus Project Internal

Coordinates received from and requested by the API should indeed always be in Project Internal.

**Question:**
Sorry for a silly question.

I discovered that the Project Base Point (PBP) in the Revit UI can be unclipped and moved away from the Project Internal (PI) – I even don't want to go there and ask why, nor about unclipping Survey Point from Shared Coords – which raised a simple API question:

Do all Revit API coordinates need to be provided in and are returned relative to:

- PI (I hope so!)
- PBP

**Answer:**
I'm 99.9% sure it's PI as I've done some tests confirming that assumption.

For all API methods, it doesn't matter if one unclips the PBP from PI and/or unclips the Survey Point from the Shared Coordinates – a.k.a 'Sites' in UI or ProjectLocation's ProjectPosition in API terms, to make it all completely inconsistent in naming terminology – the only difference is that such unclipped-then-moved PBP and SP will reflect in the Parameters (this time given in Shared Coords!) of the API BasePoint's equivalents of PBP and SP (there are always two BasePoints in the model, IsShared=true is SP, IsShared=false is PBP, as already mentioned in the discussion on the [survey and project base point](http://thebuildingcoder.typepad.com/blog/2012/11/survey-and-project-base-point.html)).

Phew, I hope this will help spare other people the effort of having to find all this out.

Thank you for these helpful explanations, Simon and Miro!