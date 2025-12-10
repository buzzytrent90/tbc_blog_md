---
post_number: "0458"
title: "Model Review for Standards Compliance"
slug: "model_review"
author: "Jeremy Tammik"
tags: ['family', 'revit-api', 'views']
source_file: "0458_model_review.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0458_model_review.html"
---

### Model Review for Standards Compliance

We discussed Autodesk Revit
[Model Review](http://thebuildingcoder.typepad.com/blog/2009/11/model-review.html) when
it was originally introduced as a subscription pack add-in for Revit 2010.
In the last moments before leaving Cairo and my host Mohamed Yamani of
[Kemet Corporation](http://www.kemet.com.eg) yesterday,
we discussed some of their further Revit application development ideas and customisation needs.
One important item they mentioned was automated checking of building standards compliance.

This can obviously be implemented using the Revit API, but of course we wish to avoid reinventing the wheel where possible.

I am absolutely not a product or implementation expert, but I did remember that old post, largely because of frequent contact with its principal implementer
[Matt Mason](http://cadappdev.blogspot.com),
also a regular Autodesk University presenter.

We verified that Kemet already have the model review module installed and were simply not aware of its possible uses, especially its customisation facilities
which allow a .NET programmer to plug in her own custom checks into the system.

So, surprisingly, in the very last minutes of my visit, yet another issue was resolved.

This is just to point out once again that this extremely useful functionality is available, in case you have similar needs.

By the way, I asked Matt what new functionality has been added to the Model Review application for Revit 2011, to which he replies:

Yes – the Model Review application was updated for 2011 (with some help from Avatech). It is downloadable from the subscription site.

As before, the principal uses of the tool are to check Revit projects or Revit families to determine if they meet with firm-specific or broad standards.

The primary enhancements in 2011 are:

- Localized in all Revit languages- Family support – while the 2010 version could run on families standalone or in a folder, the 2011 version can run on all families embedded within a project (doing the equivalent of an 'Edit Family' to test each family individually).- Check Conditions – dynamic test to determine if a particular check is appropriate or not. For example, only run a particular check if a family is in the Furniture category, or only run a particular project check if the project is a 'Healthcare facility' type building.- Document Event-based Running (automatically running on document saves, etc.).- New Checks in 2011:
          - Model Type- Building Type- Family Category

Many thanks to Matt for the quick overview!