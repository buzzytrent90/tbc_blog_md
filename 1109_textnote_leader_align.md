---
post_number: "1109"
title: "TextNote Leader Alignment"
slug: "textnote_leader_align"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'selection', 'transactions', 'views']
source_file: "1109_textnote_leader_align.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1109_textnote_leader_align.html"
---

### TextNote Leader Alignment

Let us explore [TextNote leader alignment](#2).

Before getting to that, I'll just briefly mention that I still have unused vacation days from last year to use up, so I am going away again, this time on a desert hike near
[Zagora](http://en.wikipedia.org/wiki/Zagora,_Morocco) in Morocco.

![Desert dune](img/220px_Maroc_dune_de_Chegaga.jpg)

No computer, no Internet, no blog...

#### TextNote Leader Alignment

Here is one last topic before I leave, contributed by my colleague
[Katsuaki Takamizawa](http://adndevblog.typepad.com/aec/katsuaki-takamizawa.html), with help from Jim Jia and Pawel Madej of the Revit development team:

**Question:** I am calling the NewTextNote method and specifying the leader elbow and text origin at the same height in an attempt to make the second leader segment horizontal.

However, the text is always shifted down a bit like this:

![TextNote leader shifted downwards](img/textnote_leader_1.jpeg)

As you can see, the text is shifted downward in comparison to the horizontal detail line segment.

I produced this test by drawing a detail line vertically and then horizontally, running my add-in and picking the points for the leader end, leader elbow, and text position along this detail line.

How can I make the second leader segment horizontal?

**Answer 1:** A workaround to make the second segment horizontal is provided by this code snippet:
```csharp
  // Create text note without leader first

  TextNote text = doc.Create.NewTextNote(
    view, ptTextOrigin, baseVec, ptVec,
    lineWidth, textAlign, strText);

  // Create leader for the new text:
  // change the leader elbow and end;
  // note that the elbow's Y should not be
  // changed to keep horizontal!

  if (bLeader)
  {
    Leader noteLeader = text.AddLeader(
      TextNoteLeaderTypes.TNLT\_STRAIGHT\_R);

    // Use default Y!

    noteLeader.Elbow = new XYZ (point1.X,
      noteLeader.Elbow.Y, point1.Z );

    noteLeader.End = point1;
  }
```

Note that the workaround unfortunately introduces another issue: the specified elbow end Y doesn’t take effect anymore.

**Answer 2:** You can combine the desired text alignment flags using bit-wise Boolean operations.

In your original code, you are using

```csharp
  TextAlignFlags textAlign
    = TextAlignFlags.TEF\_ALIGN\_CENTER;
```

Try using this instead:

```csharp
  TextAlignFlags textAlign
    = TextAlignFlags.TEF\_ALIGN\_MIDDLE
    | TextAlignFlags.TEF\_ALIGN\_CENTER;
```

The result is close to horizontal:

![TextNote leader almost horizontal](img/textnote_leader_2.png)

**Response:** I combined the two suggested approaches to obtain a very good result.

Here are some resulting samples using different TextAlignFlags values:

![TextNote leader alignment](img/textnote_leader_3.jpeg)

Here is the code that I used to generate these:

```csharp
  XYZ leaderOrigin = uidoc.Selection.PickPoint(
    "Pick Leader End" );

  //XYZ leaderElbow = uidoc.Selection.PickPoint(
  //  "Pick Leader Elbow");

  XYZ ptTextOrigin = uidoc.Selection.PickPoint(
    "Pick Text Origin" );

  using( Transaction tr = new Transaction( doc ) )
  {
    tr.Start( "NewTextNote" );

    XYZ baseVec = XYZ.BasisX;
    XYZ upVec = XYZ.BasisY;

    double lineWidth = 0.06;

    //TextAlignFlags textAlign
    //  = TextAlignFlags.TEF\_ALIGN\_CENTER;

    //TextAlignFlags textAlign
    //  = TextAlignFlags.TEF\_ALIGN\_MIDDLE
    //  | TextAlignFlags.TEF\_ALIGN\_CENTER;

    TextAlignFlags textAlign
      = TextAlignFlags.TEF\_ALIGN\_MIDDLE
      | TextAlignFlags.TEF\_ALIGN\_RIGHT;

    string strText = "ABCDEFGH";

    // Create text note without leader first

    TextNote text = doc.Create.NewTextNote(
      view, ptTextOrigin, baseVec, baseVec,
      lineWidth, textAlign, strText );

    // Create leader for the new text:
    // change the leader elbow and end;
    // note that the elbow's Y should not be
    // changed to keep horizontal!

    Leader noteLeader = text.AddLeader(
      TextNoteLeaderTypes.TNLT\_STRAIGHT\_R );

    // Use default Y!

    noteLeader.Elbow = new XYZ( leaderOrigin.X,
      noteLeader.Elbow.Y, leaderOrigin.Z );

    noteLeader.End = leaderOrigin;

    tr.Commit();
  }
```

Many thanks to Katsu-san, Jim and Pawel for this useful research and solution!