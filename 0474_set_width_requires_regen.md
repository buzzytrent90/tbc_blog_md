---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.009038'
original_url: https://thebuildingcoder.typepad.com/blog/0474_set_width_requires_regen.html
post_number: '0474'
reading_time_minutes: 3
series: general
slug: set_width_requires_regen
source_file: 0474_set_width_requires_regen.htm
tags:
- csharp
- elements
- family
- filtering
- revit-api
- transactions
- views
title: Setting Text Width Requires Regen
word_count: 558
---

### Setting Text Width Requires Regen

I presented a general introduction to the topic of regeneration in the discussion of my first Revit 2011 add-in, the
[pipe to conduit converter](http://thebuildingcoder.typepad.com/blog/2010/05/pipe-to-conduit-converter.html#6), and
in Kevin Vandecar's
[performance tips and tricks](http://thebuildingcoder.typepad.com/blog/2010/10/filtered-element-collectors.html).

We discussed the
[danger of lacking regeneration calls](http://thebuildingcoder.typepad.com/blog/2010/04/manual-regeneration-mode-danger.html),
[best regen practices](http://thebuildingcoder.typepad.com/blog/2010/04/regeneration-option-best-practices.html) and
[some performance issues](http://thebuildingcoder.typepad.com/blog/2010/04/regeneration-option-best-practices.html) quite
a while ago, and an example of problems resulting from accessing stale data in the
[location curve of a newly inserted family instance](http://thebuildingcoder.typepad.com/blog/2010/06/to-regenerate-or-not-to-regenerate.html).

Here is another interesting and very clear little example of one of the many problems that you can run into if you forget to regenerate appropriately, raised and resolved by Armand Marshall of
[EIC Engineers, Inc.](http://www.eicengineers.com):

**Question:** I've been working on reading values from an Excel file and creating text in Revit from that.
I read the discussion on
[TextNote rotation](http://thebuildingcoder.typepad.com/blog/2010/06/textnote-rotation.html) and
notice that you read the TextNote.Width value.
When I create a TextNote myself and try to set the TextNote.Width, it doesn't seem to work:
```csharp
  Document activeDoc = commandData.Application
    .ActiveUIDocument.Document;

  View currentView = activeDoc.ActiveView;

  XYZ origin = new XYZ( 0, 0, 0 );
  XYZ baseVec = new XYZ( 0, 0, 0 );
  XYZ upVec = new XYZ( 0, 0, 1 );

  double lineWid = 0.125;
  string strText = string1;

  TextNote textNote
    = activeDoc.Create.NewTextNote(
      currentView, origin, baseVec, upVec, lineWid,
      TextAlignFlags.TEF\_ALIGN\_CENTER
      | TextAlignFlags.TEF\_ALIGN\_MIDDLE, strText );

  textNote.Width = 21.2;
```

Setting the width seems to have no effect.
I am trying to set the width wide enough to avoid the text wrapping.

If I display the value in a task dialog, i.e. TaskDialog.Show("Test", textNote.Width.ToString()), it comes up and says that the width is zero.

Any idea what might be going on?

**Answer:** Your code looks ok to me, and I am not aware of any issues with text notes and width at all.
I am sure others are using this as well, so I cannot imagine what the problem might be.

The API documentation says:
With this method, the width of the TextNote object can be retrieved and changed.
The set method will calculate the real width of the TextNote.
If the requested width is less than the real width, the real width will be set instead of the requested width.

Have you tried using different alignment flags?
Different widths?

**Response:** I figured out the problem!

After you create the TextNote, it is as if it doesn't exist until there is a regeneration.

So I added a call to activeDoc.Regenerate after creating it and before setting its width, and now it works fine.

**Answer:** Yes, this is always a likely culprit, and I should definitely make a habit of always asking people what their regeneration option and transaction mode are before thinking of anything else.

Many thanks to Armand for this helpful example!