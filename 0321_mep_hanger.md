---
post_number: "0321"
title: "MEP Hangers"
slug: "mep_hanger"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'revit-api']
source_file: "0321_mep_hanger.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0321_mep_hanger.html"
---

### MEP Hangers

We repeatedly touched on various topics related to the
[Revit MEP API](http://thebuildingcoder.typepad.com/blog/2009/09/the-revit-mep-api.html).
Here is another little item that is a bit more out of the way, some background information on MEP duct and pipe supports, also known as hangers.

**Question:** In the description of the MechanicalEquipment class, the Revit MEP API help says that "The Mechanical Equipment object can only be queried in Autodesk Revit MEP."

I would like to add supports to ducts and pipes.
Could you please tell me whether this is possible with mechanical equipment elements?
If not, please explain which category should be used for programmatic addition of duct and pipe supports.

**Answer:** That remark in the documentation is possibly slightly misleading.
The MechanicalEquipment class itself is indeed read-only.
However, it is a specialisation of FamilyInstance, which is writeable and creatable.
I should think that the supports can be constructed with an appropriate NewFamilyInstance overload and the correct family.

Revit does currently not provide any functionality specific to hangers.
The most logical alternative would probably be to use a combination of small diameter round columns and small profile beams.
The mechanical equipment category is primarily for transferring heat (i.e., chillers, boilers), and transporting it (pumps, fans), not for support elements.

In AutoCAD MEP, the Hanger object is made up of two structural member shapes which are aggregated into a Hanger style.
Revit currently has no such functionality, and of course, no ability to add a category such as Hanger.
However, modelling the elements as small structural members for the purposes of design, coordination, and quantification may be feasible.

Many thanks to Saikat, Scott Conover, and Martin Schmid for this explanation!