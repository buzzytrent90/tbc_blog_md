---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:16.284564'
original_url: https://thebuildingcoder.typepad.com/blog/1557_family_inst_refs.html
post_number: '1557'
reading_time_minutes: 3
series: family
slug: family_inst_refs
source_file: 1557_family_inst_refs.md
tags:
- family
- geometry
- parameters
- references
- revit-api
- sheets
- views
title: Family Inst Refs
word_count: 557
---

### API Access to Family Instance References
Here is another Revit 2018 API enhancement that is obviously worth highlighting, since it immediately answers the question
on [getting a specific reference from a family instance](https://forums.autodesk.com/t5/revit-api-forum/get-specific-reference-from-family/m-p/7085619) raised in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160):
#### Question
Is it possible to get a named reference plane from a family without going into the family editor API?
For example, I would like to have users be able to name a reference plane '\*my arbitrary location\*' when they are creating a family in the family editor.
Then, when it's loaded in the project, I would like to find an instance of it, be able to find the '\*my arbitrary location\*' in the project, and use that location data for other things in relation to the placement in the project.
I've seen the post
on [how to retrieve dimensioning references](http://thebuildingcoder.typepad.com/blog/2015/05/how-to-retrieve-dimensioning-references.html),
but that basically says you would have to use some type of geometry analysis to figure out if it's the correct one or not.
Unfortunately, it's user (or at least company) specific and varied so there is no way to identify it geometrically, but it would be easy for the user to label the reference plane with a name if I could get that data out...
#### Answer
You are in luck.
A similar question was asked in a previous case, 12730663 \*Create dimension for detail items in drafting view\*:
[Q] I want to dimension the detail item in drafting view. But the geometry of the family instance only can be retrieved by GetOriginalGeometry method, and there is any reference can be got. How can I add the dimension to detail items by API?
[A] For Revit 2018, we have new capabilities to get the standard references for family instances. I haven’t tested specifically with detail items, but I believe this should work for them as well:
Check out the
section [API access to `FamilyInstance` references](http://thebuildingcoder.typepad.com/blog/2017/04/whats-new-in-the-revit-2018-api.html#3.19)
in [What's New in the Revit 2018 API](http://thebuildingcoder.typepad.com/blog/2017/04/whats-new-in-the-revit-2018-api.html):
> The following new methods have been added to enable easy access to FamilyInstance references that correspond to reference planes and reference lines in the family. Some use the options in the new enumeration FamilyInstanceReferenceType as input to identify "Strong" or "Weak" references or specific positional references in each of the 3 coordinate directions (as determined by the possible values of parameter "Is Reference" of reference planes and parameter "Reference" of reference lines in families).
- FamilyInstance.GetReferences()
- FamilyInstance.GetReferenceByName()
- FamilyInstance.GetReferenceType()
- FamilyInstance.GetReferenceName()
Prior to Revit 2018, I don’t believe there is a technique to get at the available references.
This also resolves the older case 11909355 \*Parametric family instance placement\* and the wish list item CF-1271 \*API wish: need a way to relate view specific geometry in project with particular ref plane in family\* that it gave rise to, which has now been closed as fixed.
![Family instance reference plane](img/family_instance_ref_plane.png)