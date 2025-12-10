---
post_number: "0758"
title: "Edit Family Requires No Transaction"
slug: "edit_family"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'revit-api', 'transactions']
source_file: "0758_edit_family.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0758_edit_family.html"
---

### Edit Family Requires No Transaction

I recently received a note from Matt Mason pointing out that the behaviour of the EditFamily method changed in Revit 2013, and a suggestion to highlight this fact.
The suggestion is very valid, since his note was quickly followed up by a matching
[question](http://thebuildingcoder.typepad.com/blog/2011/06/reloading-a-family.html?cid=6a00e553e1689788330168eaefdf61970c#comment-6a00e553e1689788330168eaefdf61970c) by
[Woong Ki Sung](mailto:noclew@mit.edu) on
[reloading a family](http://thebuildingcoder.typepad.com/blog/2011/06/reloading-a-family.html?cid=6a00e553e1689788330168eaefdf61970c#comment-6a00e553e1689788330168eaefdf61970c),
who ran into the very same issue:

**Question:** I am testing some code to edit family types in Revit 2013, and I have a transaction problem even though I set up manual transactions properly.

[The code](http://dl.dropbox.com/u/39185526/familyEdit.cs) is
very simple, so could you please check? In advance, thank for your help.

Here is the answer by Matt:

**Answer:** A quick note after I saw your recent post on upgrading add-ins.

I had an interesting case the other day:
there is a new API requirement in 2013 that all transactions must be closed before calling EditFamily
– it throws an exception to that effect.

I hadn't really focused on this before, and the add-in in question was actually still set up for Automatic mode
– I had to convert my add-in to manual transaction mode so that I could control the transactions and close them all prior to calling EditFamily.

The remarks on the EditFamily method in the Revit 2013 API help file RevitAPI.chm now clearly state:

Gets the document of a loaded family to edit. This creates an independent copy of the family for editing. To apply the changes back to the family stored in the document, use the LoadFamily overload accepting IFamilyLoadOptions.

This method may not be called if the document is currently modifiable (has an open transaction) or is in a read-only state. The method may not be called during dynamic updates. To test the document's current status, check the values of IsModifiable and IsReadOnly properties.

Many thanks to Matt for pointing this out!