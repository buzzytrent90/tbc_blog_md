---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.740438'
original_url: https://thebuildingcoder.typepad.com/blog/0854_temp_transact.html
post_number: 0854
reading_time_minutes: 2
series: general
slug: temp_transact
source_file: 0854_temp_transact.htm
tags:
- elements
- geometry
- revit-api
- transactions
- views
title: Temporary Transaction Trick Touchup
word_count: 431
---

﻿

### Temporary Transaction Trick Touchup

I mentioned a number of uses of the
[temporary transaction trick](http://thebuildingcoder.typepad.com/blog/2012/10/the-temporary-transaction-trick-for-gross-slab-data.html) a
few days back.

Autodesk's own Revit API transaction expert Arnošt Löbel has a very important point to add to that discussion:

The procedure as described is not always going to work.

First of all, you need to regenerate manually before retrieving any modified geometry; even that is not always guaranteed to give you the accurate and proper geometry.

Many times you will need to actually commit the 'temporary' transaction, since that is the only way to guarantee that all changes propagate though the model.

In order to undo the temporary transaction, you need to enclose it within a transaction group and roll back that group at the end.
It goes as follows:

1. Start a transaction group.- Start a transaction- Make changes.- Commit the transaction- Retrieve changed geometry- Roll back the transaction group

Unfortunately, all too many are unaware of the fact that it is safe to read a model only after regeneration, and sometimes only after committing the open transaction.
This is not only related to the 'temporary transaction' trick, it is simply a matter of fact: one should query model geometry only between transactions, or at least after regeneration (and auto-joining, if appropriate).

Many thanks to Arnošt for this important enhancement!

#### Cloud Based Team Foundation Service

We have seen how
[Git and Github](http://thebuildingcoder.typepad.com/blog/2012/09/divideparts-in-f.html#4) can
simplify global source code management and sharing, and Victor demonstrates one aspect of this by regularly providing his samples such as the
[Revit external access demo](http://thebuildingcoder.typepad.com/blog/2012/11/drive-revit-through-a-wcf-service.html) via
that platform.

As you obviously know, Autodesk is also driving hard and moving fast towards providing innovative and empowering cloud-based solutions, e.g.
[Autodesk 360](https://360.autodesk.com),
[PLM 360](http://www.autodeskplm360.com) and
[BIM 360 Glue](https://bim360.autodesk.com).

Now Microsoft joined the club, announcing its own cloud-based software project management platform, e.g. for Visual Studio, the
[Team Foundation Service](http://tfs.visualstudio.com).

You can sign up for a free preview account right away, if you are interested.

#### Rotating Annotation Symbol

Saikat Bhattacharya discusses using the AnnotationSymbol.Location.Rotate and ElementTransformUtils.RotateElement methods to
[rotate an annotation symbol](http://adndevblog.typepad.com/aec/2012/11/rotating-annotationsymbols-using-the-revit-api.html),
and presents a sample code snippet implementing it.