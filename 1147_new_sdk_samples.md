---
post_number: "1147"
title: "New Revit 2015 SDK Samples"
slug: "new_sdk_samples"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'parameters', 'revit-api']
source_file: "1147_new_sdk_samples.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1147_new_sdk_samples.html"
---

### New Revit 2015 SDK Samples

Rather belatedly, I went through my usual analysis of the Revit SDK to determine the [new samples added to the Revit 2015 SDK](#2").

While I was at it, I also migrated the [DevDays samples for Revit 2015 UR1](#3),
since some new SDK functionality is not yet covered by new SDK samples, and the ones posted with the
[DevDays online recording](http://thebuildingcoder.typepad.com/blog/2014/04/revit-2015-api-news-devdays-online-recording.html) were
still compiled for one of the early pre-release versions of Revit.

#### New Samples in the Revit 2015 SDK

To cut a long story short, the initial release of the Revit 2015 SDK includes four new samples demonstrating part of the most important
[new Revit 2015 API functionality](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html):

- ExternalResourceServer

- ExternalResourceDBServer – demonstrates a concrete class derived from the IExternalResourceServer interface. The class simulates the provision of keynote data and Revit link models from files in a remote storage location, as well as keynote data created from a database.
- ExternalResourceUIServer – provides messages and other UI in support of the SampleExternalResourceServer in the ExternalResourceDBServer project.

- FamilyParametersOrder – demonstrates how to define and sort family parameter order.
- GetSetDefaultTypes – demonstrates how to get and set default family and element types.
- Structural Analysis SDK/Examples/ASCE-7-10 – new RST sample.

Furthermore, the trusty old RevitLookup sample, an important database exploration and debugging tool in its own right, was removed from the Revit SDK and now lives all on its own in its cosy
[RevitLookup GitHub repository](https://github.com/jeremytammik/RevitLookup).

#### DevDays Online Samples Updated for Revit 2015 UR1

The new SDK samples listed above do not cover all the new Revit 2015 API features.

For instance, nothing to illustrate the important new DirectShape functionality is included above.

I therefore took another look at the samples that we used to demonstrate some of the new functionality at the DevDays Online conferences and posted with the
[DevDays online recording](http://thebuildingcoder.typepad.com/blog/2014/04/revit-2015-api-news-devdays-online-recording.html),
and migrated them to the current Revit 2015 UR1 release.

Here are the two updated archive files with and without the sample models included:

- [Sample code incl. RVT models](http://thebuildingcoder.typepad.com/devday/2013/online/Revit_2015_API_News/Revit_2015_API_Samples_UR1.zip) (45 MB)
  [^](file:////a/devdays/2013/online/Revit_2015_API_Samples_UR1.zip)
- [Sample code excl. RVT models](http://thebuildingcoder.typepad.com/devday/2013/online/Revit_2015_API_News/Revit_2015_API_Samples_No_RVT_UR1.zip) (1.5 MB)
  [^](zip/Revit_2015_API_Samples_No_RVT_UR1.zip)