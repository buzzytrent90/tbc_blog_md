---
post_number: "0181"
title: "Practical Notes on Impractical Things"
slug: "potpourri"
author: "Jeremy Tammik"
tags: ['family', 'revit-api']
source_file: "0181_potpourri.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0181_potpourri.html"
---

### Practical Notes on Impractical Things

I just learned that the two introductory images presented in the article on the
[Revit form creation API](http://thebuildingcoder.typepad.com/blog/2009/07/revit-form-creation-api.html)
came from Zach Kron, whose blog
[Buildz](http://buildz.blogspot.com)
presents lots of interesting 'practical notes on making impractical things' in Revit.

Here is a pointer to a completely different kind of practical note on another blog:

#### Disable Auto Setback in Framing Template

Several developers have been struggling with some of the hard-coded Revit behaviour.
Miroslav Schonauer just pointed out that he and Sarah Capes found a solution for one specific problem on an Autodesk Consulting project, which is documented in this article on
[how to disable auto setback in a framing template](http://bimandbeam.typepad.com/bim_beam/2009/07/framing-templatehow-to-disable-auto-setback.html).

And to round off today's
[potpourri](http://en.wikipedia.org/wiki/Potpourri),
here is another issue around the topic of form creation:

#### NewSweptBlend Failure

**Question:**
I am trying to use the NewSweptBlend method, but it throws an exception saying The attempted operation is not permitted in this type of family. What can I do to solve this?

**Answer:**
This indicates that swept blends are not supported in the category type of the family where you are attempting to create it.

If it is a 2D family type, you would get this exception, and maybe also on a few other types as well.

In your specific case, the code uses a Conceptual Mass template and tries to create a SweptBlend in it.
If you want to work in a Conceptual Mass template, you must use the NewSweptBlendForm method instead.
If you want to use NewSweptBlend, then use a non-mass template.
Note the difference between a generic model and conceptual mass.