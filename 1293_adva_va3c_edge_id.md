---
post_number: "1293"
title: "State of the View and Data API, vA3C and Edge Ids"
slug: "adva_va3c_edge_id"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'geometry', 'parameters', 'references', 'revit-api', 'selection', 'views']
source_file: "1293_adva_va3c_edge_id.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1293_adva_va3c_edge_id.html"
---

### State of the View and Data API, vA3C and Edge Ids

Today, let's take a look again at the current state of the View and Data API, vA3C and a case or two from the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160):

- [View and Data API news](#2)
- [RvtVa3c enhancement filters parameters](#3)
- [Defining keys or geometric references for curve vertices](#4)

#### View and Data API News

**Question:**
I am interested in the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46) that
[you wrote about last year](http://thebuildingcoder.typepad.com/blog/2014/07/autodesk-view-and-data-api.html).

I am wondering what the current state and plans are for the Autodesk View & Data API.
As far as I can see it’s still in beta and not that much developments on this last months.
Or did I miss some information?
Do you know about the plans for this product?

**Answer:**
Yes, you definitely missed something :-)

It is in full swing, there has been a lot of development in the last few months, and the number of available samples has grown tremendously.

Take a look at the
[beautifully revamped entry page](http://developer-autodesk.github.io) and
see for yourself.

It is chock full of interactive samples and source code.

You are right, though, it is still not commercially released.

I do not know the plans beyond that this is very central to Autodesk and shows the direction of future development.

**Response:**
What does it mean for us as company that it’s not commercially released?

Am I allowed to connect and integrate this within my own products?

**Answer:**
You can currently use it as it is, and several developers are already making commercial use of it as well.

It is still in a state of flux, though, and the final business model has not been decided on.

If you click on the getting started button, you can register your own application to obtain a consumer key and secret and start coding and using and distributing it immediately.

#### Addendum – Beta Status and Pricing

Please note that a so-called 'Beta' of a desktop application is something completely different than a beta on the web.

Large-scale web updates are happening transparently all the time.

Even though it is officially in beta, some partners are already using it in production right now.

Funnily enough, because it is currently free and there is no price, people are suspicious.

Google maps was free for everyone originally. After four years, Google decided that anyone making more than X million API calls per year must pay.

Just like them, we need to measure usage before we can make any kind of decision.

We even thought of simply putting a price on this web service right now, not to earn money but to gain credibility.

One number that was mentioned hypothetically was $1 per model per year, with the first X models for free.

Other ideas might be based on the volume of data uploaded, e.g. pay per MB or GB, again only above a certain threshold.

#### RvtVa3c Enhancement Filters Parameters

Progress has also continued on the grass-roots, public domain and open source
[vA3C three.js based AEC viewer](https://va3c.github.io).

vA3C is an open source, browser-based 3D model viewer for AEC models that uses three.js to render 3D geometry in the browser.
vA3C enables authors in the AEC industry to easily publish their 3D design work on the web, for free.

Here are some of the previous discussions on the topic:

- [AEC Hackathon](http://thebuildingcoder.typepad.com/blog/2014/05/aec-hackathon-in-new-york-and-new-developer-guide-url.html)
- [AEC Hackathon – from the midst of the fray](http://thebuildingcoder.typepad.com/blog/2014/05/aec-hackathon-from-the-midst-of-the-fray.html)
- [RvtVa3c – Revit Va3c generic AEC viewer JSON export](http://thebuildingcoder.typepad.com/blog/2014/05/rvtva3c-revit-va3c-generic-aec-viewer-json-export.html)
- [RvtVa3c assembly resolver](http://thebuildingcoder.typepad.com/blog/2014/05/rvtva3c-assembly-resolver.html)
- [Significant progress on the vA3C project](http://thebuildingcoder.typepad.com/blog/2014/08/threejs-aec-viewer-progress-on-two-fronts.html#4)
- [Integrating RvtVa3c into three.js](http://thebuildingcoder.typepad.com/blog/2014/09/adn-labs-xtra-on-github-and-rvtva3c-in-threejs.html#5)
- [Custom user settings storage and RvtVa3c update](http://thebuildingcoder.typepad.com/blog/2014/10/berlin-hackathon-results-3d-viewer-and-web-news.html#7)
- [vA3C update](http://thebuildingcoder.typepad.com/blog/2015/01/3d-viewing-va3c-and-revitlookup-updates.html)

Now a new enhancement was added by
[Ana Puyol](https://github.com/anagpuyol) of
[Thornton Tomasetti](http://www.thorntontomasetti.com), the hosts of the New York AEC Hackathon where this project originated last year.

Here is Ana's own description in the original
[filter parameters pull request #6](https://github.com/va3c/RvtVa3c/pull/6):

Added UI to filter parameters. Only selected parameters will be included in the JSON file.

Step 1:

![RvtVa3c filter parameters](img/RvtVa3c_filter_parameters_1.png)

If Yes is clicked, the only elements that get exported are those that are visible in the view. Otherwise, the command is run as before. Also, included a toggle for the type parameters.

Step 2:

![RvtVa3c filter parameters](img/RvtVa3c_filter_parameters_2.png)

As always, the current version is available from the
[RvtVa3c GitHub repository](https://github.com/va3c/RvtVa3c),
and the version including Ana's enhancement is
[release 2015.0.0.30](https://github.com/va3c/RvtVa3c/releases/tag/2015.0.0.30).

This kind of interactive parameter selection and filtering may obviously be very useful for many other applications as well.

Thank you, Ana, for your contribution!

#### Defining Keys or Geometric References for Curve Vertices

Getting back to the Revit API discussion forum issues, one that I like a lot is the question on
[how to get vertices of a Revit edge along with unique id for each vertex](http://forums.autodesk.com/t5/revit-api/how-to-get-vertices-of-a-revit-edge-along-with-unique-id-for/m-p/5529303):

**Question:**
I am trying to query the end points i.e. vertices of an edge in Revit.

I tried using edge.AsCurve().GetEndPoint(0/1), which is returning the correct endpoints as an object of XYZ class.

But I also want a unique identifier of each vertex so that I can eliminate the overlapping vertices from my processing while creating Boundary Representation of a Revit body.

I tried using Edge.GetHashCode() which works good for Revit Edges i.e. the hash code for an edge queried from one face is same when we get the same edge in another face.

But XYZ.GetHashCode() is not returning same code for end points that are common for different edges/faces.

Am I querying the end points correctly?

Is there any other way to query a unique identifier for the Revit vertices and even edges?

**Answer:**
In general, defining any kind of stable identifier for geometry that is dynamically generated and can change at any time is very hard.

In the Revit API, you can request so-called references to geometry objects. To do so, you set ComputeReferences to true when querying an element for its geometry.

These references can be converted to a stable string representation for storage and later retrieval using the Reference.ConvertToStableRepresentation method.

I suggest a much simpler approach here that may be of interest to you,
[creating your own key based on geometric location](http://thebuildingcoder.typepad.com/blog/2012/03/great-ocean-road-and-creating-your-own-key.html#2).

That will obviously not be stable either, but it might provide what you need.

I hope this helps.

**Response:**
Using Dictionary for assigning unique ids to curve endpoints (XYZ) worked for me.

Thanks a lot!