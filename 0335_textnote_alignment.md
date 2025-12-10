---
post_number: "0335"
title: "RevitLookup and TextNote Alignment"
slug: "textnote_alignment"
author: "Jeremy Tammik"
tags: ['elements', 'parameters', 'revit-api']
source_file: "0335_textnote_alignment.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0335_textnote_alignment.html"
---

### RevitLookup and TextNote Alignment

To take a break from all the exciting Revit 2011 oriented news and dash off a quick post while continuing my preparations for the Revit programming introduction workshop here in Warsaw, I thought I would simply present a recent case handled by my colleague Saikat Bhattacharya.
While doing so, however, I ran into yet another 2011 related issue concerning RevitLookup, previously known as RvtMgdDbg, which I thought I should mention before diving into the details of the case.

#### RevitLookup

Both I and many others have repeatedly extolled the virtues of
[RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html),
the Revit database exploration and snooping tool.
The in-depth description above is based on the version for Revit 2009.
A
[version for Revit 2010](http://thebuildingcoder.typepad.com/blog/2009/10/rvtmgddbg-for-revit-2010.html) is
also available.
One issue with this tool in the past has been that although it is an invaluable and irreplaceable debugging and exploration tool, it has been developed and managed by a separate team.
That has now happily changed.
From Revit 2011 onwards, this tool has been taken over by the team that manages the Revit SDK and is thus included there.
Since the Revit SDK is included in the Revit setup files, RevitLookup is now available with every installation of Revit.
At the same time, the old name RvtMgdDbg was deemed unpronounceable and it has therefore been renamed to RevitLookup.
With that small but important and happy bit of information out of the way, let us look at an example of using it.

#### TextNote Alignment

**Question:** I am trying to copy a TextNote from one Revit document to another.
I need to access the TextAlignment value in order to pass it to the Create.NewTextNote method in the target document.
Using RevitLookup, I can see that the TextNote object has a HORIZ\_ALIGN parameter which defines the horizontal alignment, but I cannot find a corresponding parameter for the vertical alignment within the TextNote or the TextNoteType object.
Which object contains the vertical alignment information, so I can combine both the horizontal and vertical alignment for the call to the Create.NewTextNote method?
Here is a screenshot of RevitLookup displaying the HORIZ\_ALIGN parameter:

![HORIZ_ALIGN parameter](img/text_align_horiz.png)

**Answer:** Your screen snapshot above shows RevitLookup exploring the officially available parameters on the TextNote object.
Many Revit elements have more parameters attached to them than the ones displayed in the official parameters collection.
The additional ones can be displayed by iterating over all possible values of the built-in parameters enumeration and trying to obtain a corresponding parameter for each.
This iteration is performed by RevitLookup if you click the 'Built-in Enums Snoop' button.
Doing that, you will discover that the text note vertical alignment information is stored in the built-in parameter TEXT\_ALIGN\_VERT, which is also located on the TextNote object.
Here is a screen snapshot showing how to access the value in RevitLookup:

![TEXT_ALIGN_VERT parameter](img/text_align_vert.png)

To explore the meanings of the integer values stored in the TEXT\_ALIGN\_VERT parameter, I created three different text notes with varying vertical alignments using the API and looked at each of them in RevitLookup.
That showed that the internal values for the built-in parameter for the different vertical alignments correspond to the following TextAlignFlags:

```
TextAlignFlags.TEF_ALIGN_TOP = 512
TextAlignFlags.TEF_ALIGN_MIDDLE = 1024
TextAlignFlags.TEF_ALIGN_BOTTOM = 2048
```

You can thus extract the integer value of the TEXT\_ALIGN\_VERT parameter on the TextNote object and use it to recreate an equivalent new object with the same vertical alignment via the NewTextNote method.