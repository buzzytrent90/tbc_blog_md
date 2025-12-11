---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.8
content_type: code_example
optimization_date: '2025-12-11T11:44:15.819575'
original_url: https://thebuildingcoder.typepad.com/blog/1348_batch_dwf_futureproof.html
post_number: '1348'
reading_time_minutes: 6
series: general
slug: batch_dwf_futureproof
source_file: 1348_batch_dwf_futureproof.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- parameters
- python
- revit-api
- views
- walls
title: Batch Processing, DWFx Links, Future-Proofing
word_count: 1247
---

### Batch Processing, DWFx Links, Future-Proofing

The never-ending stream of Revit API topics continues.

Today, let's look at:

- [Unrestricted VendorId](#2).
- [Retrieving DWFx links](#3).
- [Batch processing Revit documents](#4).
- [Future-proofing The Building Coder samples](#5).
- Recent Revit API AEC DevBlog posts:

- [Delete custom family parameter](http://adndevblog.typepad.com/aec/2015/08/delete-custom-familyparameter.html).
- [JoinGeometryUtils.JoinGeometry for walls and columns](http://adndevblog.typepad.com/aec/2015/08/joingeometry-for-walls-and-columns.html).
- [Reading gross and rentable area elements](http://adndevblog.typepad.com/aec/2015/07/reading-gross-and-rentable-area-elements.html).

#### Unrestricted VendorId

In the past, we recommended using an Autodesk registered developer symbol or RDS as your vendor id, stored in the add-in manifest `VendorId` tag
(starting from the [Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/03/revit-2012-api-features.html)
[extensible storage](http://thebuildingcoder.typepad.com/blog/2011/04/extensible-storage.html) and
[add-in wizard](http://thebuildingcoder.typepad.com/blog/2011/04/visual-studio-add-in-wizards-for-revit-2012.html)).

Autodesk registered developer symbols are very limited, though, consisting of only four characters from a restricted set, [A-Z], [0-9], minus '-' and underscore '\_'.

The Autodesk registered developer symbol usage is now deprecated, so this restriction has been lifted for the Revit VendorId specification.

This liberalisation is already documented in the developer guide, in the
[Revit 2016 online help](http://help.autodesk.com/view/RVT/2016/ENU) >
Developers > Revit API Developers Guide > Introduction > Add-In Integration >
[Add-in Registration](http://help.autodesk.com/view/RVT/2016/ENU/?guid=GUID-4FFDB03E-6936-417C-9772-8FC258A261F7):

- **VendorId** –
  A unique vendor identifier that may be used by some operations in Revit (such as identification of extensible storage). This must be unique, and thus we recommend using reversed version of your domain name, for example, com.autodesk or uk.co.autodesk.

Accordingly, I should update my [Visual Studio Revit add-in wizard](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.20) to use `com.typepad.thebuildingcoder` instead of the current RDS `TBC_`... to be done asap.

#### Retrieving DWFx Links

Let's look at how to retrieve DWFx links from the current Revit project.

This question was raised and solved by Daniel in the Revit API discussion forum thread on
[getting all linked DWFx files in the Revit document](http://forums.autodesk.com/t5/revit-api/get-all-linked-dwfx-files-from-in-revit-document/m-p/5769622):

**Question:**
I'm trying to list all links in a document using filtered element collectors.

I get the RVT links (Revit and IFC) through the built-in category; DWG links also work fine using the `ofClass(typeof(importInstance))` filter.

I thought that DWFx and point clouds also should be listed under this filter?

It seems not, though.
I can't get them through that filter, and I can't see any other type to use.

**Answer:**
While working on creating a minimal reproducible sample, I found the error.

It's kind of simple. DWFx files are not linked, even though they appear to be so.

When I changed from:

```csharp
var dwgLinks = fecDwgLinks
.OfClass(typeof(ImportInstance))
.ToElements()
.Cast<ImportInstance>()
.Where(f=> f.IsLinked);
```

To:

```csharp
var dwgLinks = fecDwgLinks
.OfClass(typeof(ImportInstance))
.ToElements()
.Cast<ImportInstance>();
```

Things worked fine.

This code returns other links as well, but that is how I want it.

I am creating a tool that lists all the links.

When I use the collected elements, I use the category name, which contains the filename extension, to group the different types of links:

```csharp
var namearray = importlink.Category.Name.Split('.');
var name = namearray.FirstOrDefault();
var extension = string.Empty;
if (namearray.LastOrDefault() != null)
extension = namearray.Last();
```

I have created an Enum to group a list on:

```csharp
GroupingEnum lType = GroupingEnum.Unknown;
if (extension.ToLower().Contains("dwg"))
lType = GroupingEnum.Dwg;
else if (extension.ToLower().Contains("dwf"))
lType = GroupingEnum.Dwf;
```

This is used to group a list so that it is easy to choose the wanted elements.

#### Batch Processing Revit Documents

**Question:**
I'm looking at building a Revit service that processes Revit families on demand and extracts Revit family information.

Do you know of anyone who has done this before?

I have been reading the discussion on [driving Revit through a WCF service](http://thebuildingcoder.typepad.com/blog/2012/11/drive-revit-through-a-wcf-service.html).

I'm stuck on the problem that I will run into Revit errors that can't be handled through the API or Revit will eventually crash.

I will have to kill the Revit process, skip the family and start another Revit process.

How will I detect that Revit has stopped working?

Here is my proposed architecture:

- Install Revit on a server.
- Build a Revit plugin that extracts Revit family information, also the plugin subscribe to the Idling event and checks a WCF queue.
- Build some queue managing software on the server that receives the Revit family and add it to the WCF queue.
- Revit Idling event processes the families in the queue and adds the results to a database.

Easy so far.

But how do I catch in the queue managing software if Revit crashes or a warning dialogue pops up and stops Revit?

I could use time if Revit takes more than 2 minutes to process a family than it is not responding end the Revit process (crash it), skip this family and start up another Revit session.

How would you detect if Revit is not responding?

**Answer:**
I think the blog post you point to is a good starting point.

You might also want to check out Daren Thomas' recent contributions on this topic, discussing the
[Revit Python shell in the cloud as a web server](http://thebuildingcoder.typepad.com/blog/2015/07/firerating-and-the-revit-python-shell-in-the-cloud-as-web-servers.html#5) and explaining the
[Revit API context](http://thebuildingcoder.typepad.com/blog/2015/08/revit-api-context-and-form-creation-errors.html#2).

I have summarised several other related discussions in The Building Coder topic group on
[Idling and external events for modeless access and driving Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28).

By the way, I recommend never using the Idling event, if it can possibly be avoided.
External events are more flexible and do not block the entire system, like the Idling event sometimes does.

Otherwise, your architecture sounds absolutely fine to me.

Yes, you should assume that Revit will stop working after a while when driven in this manner.

That is to be expected, since it is implemented to be driven in a completely different manner, interactively, by a human user.

How to check when it stops working?

Probably the best approach is indeed to implement a timer and see how long it takes to complete the current request.

Once the timeout is exceeded, assume it died, kill it properly, and restart at that point.

This is a standard approach for
[driving lengthy complex batch processes](http://thebuildingcoder.typepad.com/blog/2014/12/au-ends-and-batch-rendering-across-several-projects.html).

#### Future-Proofing The Building Coder Samples

I continued in my perpetual struggle to future-proof
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) by
eliminating all deprecated API usage.

The result of my latest effort is
[release 2016.0.120.9](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.120.9),
which compiles once again with zero warnings.