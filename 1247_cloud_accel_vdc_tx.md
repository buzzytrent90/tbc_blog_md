---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.600878'
original_url: https://thebuildingcoder.typepad.com/blog/1247_cloud_accel_vdc_tx.html
post_number: '1247'
reading_time_minutes: 4
series: general
slug: cloud_accel_vdc_tx
source_file: 1247_cloud_accel_vdc_tx.htm
tags:
- elements
- revit-api
- transactions
title: Cloud Accelerator, VDC and Transaction Groups
word_count: 781
---

### Cloud Accelerator, VDC and Transaction Groups

Wow, hark these cool topics for today:

- [Autodesk Cloud Accelerator workshop invitation](#2)
- [BIM becomes VDC](#3)
- [Using Transaction Groups to Relinquish Elements Without Saving or Syncing](#4)

#### Autodesk Cloud Accelerator Workshop Invitation

Autodesk just announced the
[2015 Autodesk Cloud Accelerator](http://adndevblog.typepad.com/cloud_and_mobile/2014/11/get-ready-for-the-autodesk-cloud-accelerator-program.html),
a two-week workshop in March 2015 in our San Francisco office for up to 14 creative software developers.

For two weeks, March 9 to March 20, 2015, participants will work intensively on their chosen project with the help, support, and training of the Autodesk Cloud engineering teams. This includes daily classes by industry experts on important aspects of cloud development and 1 on 1 help from these experts in real time. At the end of the Accelerator, you will also have the opportunity to present your prototype to senior Autodesk Executives.

![Autodesk Cloud Accelerator](img/cloud_accelerator.jpg)

If you are interested in joining the adventure, please look at Stephen Preston's more detailed invitation to
[get ready for the Autodesk cloud accelerator program](http://adndevblog.typepad.com/cloud_and_mobile/2014/11/get-ready-for-the-autodesk-cloud-accelerator-program.html).

#### BIM Becomes VDC

Some people are still struggling to contend that BIM is hype.

Whatever are they going to say about VDC, virtual design and construction?

That seems to be the direction the industry is heading right now.

To learn more, take a look at
[BIM becomes VDC](http://www.bdcnetwork.com/bim-becomes-vdc),
*A case study in disruption*, by John Tobin, LEED AP.

#### Using Transaction Groups to Relinquish Elements Without Saving or Syncing

**Question:** I have a valid local file open in Revit which has been modified by my add-in.
Can I programmatically close this file without syncing, saving, or leaving any elements checked out by the current user?

It's possible to relinquish un-modified elements using the WorksharingUtils.RelinquishOwnership method.
However, this will not relinquish the modified elements.

The documentation for Document.Close also doesn't mention anything about relinquishing checked out elements.

So, is the only way to do this to close without saving, re-open the same model, and then call RelinquishOwnership?

**Answer:** Thank you for your interesting query.

One obvious approach would be to encapsulate all of the changes performed by your add-in in a transaction group and roll that back again before terminating your add-in command sequence.

**Response:** Yeah, rolling back a transaction is the obvious things to do.
Unfortunately with the tool I'm working on a number of transactions will have been committed before it decides that it needs to discard the changes.

What about a TransactionGroup?

If I commit transactions that check out elements inside a transaction group and then roll back the transaction group will this also relinquish the checked out elements?

At least at then those elements would not longer be modified, so Relinquish should work.

**Answer:** Yes, exactly.
I was not sure whether that was obvious to you, which is why I pointed it out.

Some Revit API interactions do in fact require a committed transaction, or a whole series of sequential committed transactions, in order to fulfil certain tasks.

All of these transactions can be fully committed, and will still be rolled back if the entire series is encapsulated in a transaction group.

Here is the description of the TransactionGroup class in the Revit API help file RevitAPI.chm:

Transaction groups aggregate a number of transactions.

A transaction group controls whether transactions committed inside the group should stay committed or should be all discarded. If the group is committed, all the transactions remain committed, but if the transaction group is rolled back instead, all the inner, already committed transactions will be undone (and removed).

There are two ways of committing a group – Commit and Assimilate. By committing, all transactions committed inside a group stay as they are, while by assimilating, all inner transactions will be merged into a single transaction.

A transaction group can only be started when no transaction is active, and must be closed only after the last transaction started inside the group is finished, i.e. after it was either committed or rolled back.

Transaction groups may be nested inside each other with the restriction that every nested transaction group is entirely contained (opened and closed) in the parent transaction group.

Just like transactions themselves, the simplest, most effective and fool proof way of handling transaction groups in .NET code is to
[encapsulate them in a 'using' statement](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html).

I hope this helps.