---
post_number: "1016"
title: "Access to Individual Elements in Linked Projects"
slug: "linked_project_element"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'python', 'revit-api', 'transactions', 'views', 'walls']
source_file: "1016_linked_project_element.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1016_linked_project_element.html"
---

### Access to Individual Elements in Linked Projects

Access to individual elements in linked projects is a relatively simple matter, but can still cause some confusion, as we can see below.

We looked at various aspects of linking files and accessing the linked element data and geometry in the past, e.g.

- [Linked files](http://thebuildingcoder.typepad.com/blog/2008/12/linked-files.html)
- [Hiding linked files](http://thebuildingcoder.typepad.com/blog/2009/01/hiding-linked-files.html)
- [List linked elements](http://thebuildingcoder.typepad.com/blog/2009/02/list-linked-elements.html)
- [Access to linked file geometry](http://thebuildingcoder.typepad.com/blog/2009/04/access-to-linked-file-geometry.html)
- [List linked files and TransmissionData](http://thebuildingcoder.typepad.com/blog/2011/05/list-linked-files-and-transmissiondata.html)
- [Linked element geometry access](http://thebuildingcoder.typepad.com/blog/2011/07/linked-element-geometry-access.html)
- [Selecting a face in a linked file](http://thebuildingcoder.typepad.com/blog/2012/05/selecting-a-face-in-a-linked-file.html)
- Arnošt Löbel's AU 2012 class
  [CP3455](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3455) on
  [managing Revit links](http://thebuildingcoder.typepad.com/blog/2012/11/au-classes-on-the-view-mep-and-link-apis.html#4)

Let's revisit this issue to hopefully achieve final clarity addressing the following points after a quick mention of my Saturday mountain hike:

- [Martinsloch and Piz Segnes](#1)
- [Executive summary](#2)
- [Detailed discussion](#3)
- [Sample code](#4)
- [Consider using a custom exporter](#5)

#### Martinsloch and Piz Segnes

I went on a beautiful hike to the Segnes pass and Piz Segnes with my friend
[Urs](https://www.facebook.com/urs.wiesendanger.3/posts/10200327965538712) this
weekend.

We spent the night outside, sleeping under the stars and occasional strong bursts of wind on the terrace in front of the
[mountain lodge](http://www.segnespass.ch) at
2627 m, right beside the
[Martinsloch](http://de.wikipedia.org/wiki/Martinsloch):

![Martinsloch](img/martinsloch.jpg)

From there, we climbed Piz Segnes 3098 m and went part of the way over the ridge to Piz Sardona, 3055 m.
I really feel my legs now!

#### Linked Element Access Executive Summary

We'll start by looking at a really short executive summary of the question and answer, followed by the lengthy discussion required to summarise it so succinctly:

**Question:** How can I retrieve the element geometry from all the different instances of a linked model?

For each copy of the linked model, I need to determine a different geometric instance and representation of the same element in it.

**Answer:** Get the elements from the linked document, get the transformation from each one of the linked document instances, and apply them to the elements.

#### Detailed Discussion

To see how hard and confusing it can be to achieve this understanding, here is a summary of the conversation that led up to the question and answer above:

**Question:** I would like to do is retrieve all copies of elements from a linked file.

In this sample case, I am dealing with walls and have a model containing two instances of a linked model.

The linked model contains one wall, so the containing model displays two walls.

I am using the code at [pastebin.com/XCC0ru8K](http://pastebin.com/XCC0ru8K).

In Revit 2014 it only returns the original wall coming in the linked model itself, but not the copies in the containing model that I am really interested in.

My questions are:

- Is there any way in the API to retrieve the walls from the copies of the linked models?
- If there is, are these copies going to have the same element ids as their originals?

If the second point is true, is there any way to still differentiate the originals from their copies by some other unique identifier?

For a concrete example, let's say you link a model a.rvt into the containing model c.rvt.

After linking it once, you create another copy of a.rvt in c.rvt, e.g. by Copy and Paste on the inserted linked a.rvt, placing it somewhere next to the original link.

Here is a picture illustrating this:

![Two copies of the same linked file](img/rs_linked_elements_1.jpeg)

This way you have two instances of a.rvt linked into c.rvt.
Each instance contains one wall.

I am trying to write code to retrieve both walls.

My attempts so far only return the wall from the linked model first inserted, not the wall from the second copy.

I need to have both walls, because I need to access their positions to display them both in the proper location.

**Answer:** Yes, that should be pretty easy.

You know that a.rvt is linked into c.rvt.

You can get the original wall element from a filtered element collector running on doc\_a.

This does not tell you where the wall is displayed in the C model.

For that, you need to get the instances of A in C.
These are link instances, represented by the Revit API class RevitLinkInstance.

You get those from a filtered element collector running on doc\_c.

Each link instance includes placement information.

That tells you where the different copies of the wall will appear in model C.

There will not be any "copies of the wall", just instances of the link.

You can easily explore the situation and see these relationships for yourself using RevitLookup.

Actually, there are several properties that provide access to the placement information.
The one most appropriate for your situation is probably the Instance.GetTotalTransform property.

The sample code provided to
[list linked elements](http://thebuildingcoder.typepad.com/blog/2009/02/list-linked-elements.html) shows
that this was even possible way back in Revit 2009.

The numerous other samples listed above really have said all that can be said on this topic.

#### Sample Code

Let's look at an ultimate minimal simple sample demonstrating the main relationships between, and how to access:

- The link types
- The link instances
- The linked documents
- Their elements

Here is the entire Execute method mainline of an external command accessing and listing all of these:

```python
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  FilteredElementCollector links
    = new FilteredElementCollector( doc )
      .OfCategory( BuiltInCategory.OST\_RvtLinks );

  Debug.Print( "\nLinks:\n" );

  string what;

  foreach( Element e in links )
  {
    if( e is RevitLinkInstance )
    {
      what = "instance";
    }
    else
    {
      Debug.Assert( e is RevitLinkType,
        "expected all RvtLinks elements to "
        + "be link type or link instance" );

      what = "type";
    }

    Debug.Print( string.Format(
      "Link {0} '{1}' <{2}>",
      what, e.Name, e.Id.IntegerValue ) );
  }

  Debug.Print( "\nDocuments and their elements:\n" );

  foreach( Document d in app.Documents )
  {
    if( d.IsLinked )
    {
      Debug.Print( string.Format(
        "Linked document '{0}':",
        d.Title ) );

      FilteredElementCollector walls
        = new FilteredElementCollector( d )
          .OfClass( typeof( Wall ) );

      foreach( Element e in walls )
      {
        Debug.Print( string.Format(
          "  Element '{0}' <{1}>",
          e.Name, e.Id.IntegerValue ) );
      }
    }
  }
  return Result.Succeeded;
```

This command can obviously use read-only transaction mode, since no database modifications are performed.

Note that the OST\_Links built-in category identifies both link types and link instances.

One link type is generated for each linked document.

Every copy of the linked document placed into the model generates a new link instance.

This is similar to the relationship between family types and instances.

We apply this command to the following model containing two linked files, a.rvt and b.rvt, each of them containing one single wall and inserted twice into the model:

![Sample model containing four link instances or two linked files](img/rs_linked_elements_2.png)

Just as expected, the output generated by this call lists the two link types, one for each document, the four instances, two for each document, the two linked documents, and one wall element in each:

```
Links:

Link type 'a.rvt' <193823>
Link instance 'a.rvt : 1 : location <Not Shared>' <193824>
Link type 'b.rvt' <193826>
Link instance 'b.rvt : 2 : location <Not Shared>' <193827>
Link instance 'a.rvt : 3 : location <Not Shared>' <193829>
Link instance 'b.rvt : 4 : location <Not Shared>' <193841>

Documents and their elements:

Linked document 'a.rvt':
  Element 'Generic - 200mm' <193834>
Linked document 'b.rvt':
  Element 'Generic - 200mm' <193842>
```

#### Consider Using a Custom Exporter

You say that you are interested only in "the element geometry from all the different instances of a linked model".

In that case, quite possibly, the easiest way to achieve what you want is to use the
[custom exporter framework](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html).

Here is a happy report in this vein by Paul Kinnane in his
[comment](http://thebuildingcoder.typepad.com/blog/2013/07/revit-2014-obj-exporter-and-new-sdk-samples.html?cid=6a00e553e168978833019aff1a9b0a970b#comment-6a00e553e168978833019aff1a9b0a970b) on it:

Yes, the 2014 version of OctaneRender for Revit uses the custom exporter framework.
The 2012 and 2013 versions use a much more complex geometry extraction system based on your originally posted code, but much more complex still, e.g. in order to try to get assigned materials, generate UV's, etc.
The custom exporter framework is vastly simpler – a lot faster to load, and about 1/50th of the coding time :-)

I've done some load time comparisons (i.e. the time taking to get the geometry from Revit and load into Octane).
Based on a 500k polygon scene, the C# based Revit 2014 version is one of the fastest of the plugins for the various architectural and design apps that I have written for Octane, which I was not expecting.
I expected C++ non-managed to be significantly faster, but this was not the case.

Not being able to obtain texture map and bump map filenames for Revit materials is still an issue...

The biggest challenge was the limitations placed by the API when in a 3D perspective view...

Keep up the good work with your posts and code samples – they are invaluable to plugin developers.

Many thanks to you, Paul, for the good news, and also for reporting on the problematic areas!

And whoever is interested in only "the element geometry from all the different instances of a linked model", please pay heed – making use of the custom exporter framework instead of the explicitly retrieving and traversing all kinds of links and relationships as described above may well prove "vastly simpler" and "1/50th of the coding time" for you too!