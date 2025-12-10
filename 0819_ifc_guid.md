---
post_number: "0819"
title: "IFC GUID Generation and Uniqueness"
slug: "ifc_guid"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'levels', 'revit-api', 'rooms', 'walls']
source_file: "0819_ifc_guid.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0819_ifc_guid.html"
---

### IFC GUID Generation and Uniqueness

We looked at IFC GUIDs a couple of times already, e.g. to discuss the relationship between
[UniqueId, DWF and IFC GUID](http://thebuildingcoder.typepad.com/blog/2009/02/uniqueid-dwf-and-ifc-guid.html),
the
[ExportUtils.GetExportId method](http://thebuildingcoder.typepad.com/blog/2010/06/ifc-guid-algorithm-in-c.html),
and the
[IFC GUID representation algorithm](http://thebuildingcoder.typepad.com/blog/2010/06/ifc-guid-algorithm-in-c.html).

The Revit 2012 API introduced the ExporterIFCUtils class for customizing IFC export, and the
[IFC exporter was released as open source](http://thebuildingcoder.typepad.com/blog/2011/09/revit-ifc-exporter-released-as-open-source.html#2).

In Revit 2013, the IFC export implementation switched to a more generic toolkit, many of the APIs introduced in Revit 2012 were removed and replaced, and new interfaces introduced that allow more flexibility in the data which is written to the IFC file.

Here is some clarification on some of the methods associated with IFC GUIDs:

**Question:** Can you please tell me how these ExporterIFCUtils functions should work:

- CreateGUID- CreateGUID(Element)- CreateAlternateGUID(Element)- CreateSubElementGUID(Element, int)

I was hoping that at least one of them would generate a consistent GUID, but it seems that they are different in every run.
How can I generate a consistent IFC GUID for a given element?

**Answer:** The one and only really unique id maintained by Revit and guaranteed to be unique and stable for elements across worksharing environments is the UniqueId, which is a combination of a GUID and the history signature identifying the history of a workshared file.

That said, the IFC GUID is also unique across sessions.
This is one of the requirements of IFC.
If I export a wall in one session via IFC, close the session, reopen the document, and reexport, the GUID remains the same for that wall, and should be different from any other wall's GUID.

The method CreateGUID with no arguments however does indeed create a random GUID every time.

The other routines are intended to be stable.

For a given Element with an Element Id in a Document, the two methods CreateGUID(Element) and CreateAlternateGUID(Element) should create two consistent GUIDs.

For a given Element with an Element Id and an index (the int) in a Document, the method CreateSubElementGUID(Element, int) should create a consistent GUID, based on the GUID created with CreateGUID above.

For more information, please refer to the description of the ExportUtils.GetExportId method.

**Response:** That makes sense.
Unfortunately, the GUID is different in every run for me.

The CreateProjectLevelGUID method also returns a different GUID in every run.
I guess it should also be consistent, shouldn't it?

**Answer:** After some investigation we now understand the source of the issue.
The GUID generation routines have an internal cache that prevents duplicate GUIDs from being generated.
In IFC, these functions are called once per element, and then a call to the ExporterIFCUtils.EndExportInternal method clears this cache for the next IFC export.
Please note that the first time you call these routines, they have consistent values from previous sessions.

For now, there are two workarounds:

1. Only call the functions once per session, and store their values.- Call ExporterIFCUtils.EndExportInternal to reset the internal cache.

I would recommend #1 over #2, as the EndExportInternal call does more than just clear the cache, and may have unintended consequences.

**Response:** Thank you very much for your explanation.
It makes perfectly sense now.

The workarounds you mention may solve this but have some drawbacks:

- I can't use the EndExportInternal method because I'm not using the Revit IFC API and therefore I don't have access to an ExporterIFC object.- I don't want to start building my own cache, because it would not be 100% water proof, and who knows how much memory it would eventually consume with really huge projects.

Fortunately, there's a third workaround which I decided to implement.
I combined the
[IFC GUID representation algorithm](http://thebuildingcoder.typepad.com/blog/2010/06/ifc-guid-algorithm-in-c.html) showing
how to construct an IFC GUID from a 'normal GUID' with the ExportUtils.GetExportId functionality, which produces a consistent GUID between runs.

Can you see any problems in this approach?

**Answer:** A couple of comments:

First of all, the approach you ended up using is the one that I was going to suggest (or close enough!)
You should indeed go for that.

Secondly, just to clear up one point, the Revit IFC API is part of the Revit API – if you have the Revit API, you also have the Revit IFC API.

**Question:** In another case, we are exporting models from Revit Architecture to IFC and observe that the GUIDs of some entities are not consistent, e.g. of IFCRelationship, IFCTypeObject and IFCPropertySetDefinition.

We are not making any changes to the model, just exporting it twice.
Still, the exported IFC files contain different GUIDs generated for those entity types.

The GUID for geometric entities like IFCDOOR, IFCWalllStantdardCase etc. do not change.

What might be causing this behaviour?

**Answer:** The algorithm for generating a consistent IfcGUID depends on the IFC entity in question having a constant and predictable relationship to a Revit element. For something like an IFCDOOR, there is a one-to-one relationship between that and a Revit door element, so we can guarantee a consistent IfcGUID.
For things like relationships, we either

1. Don't have a way to make this 1-1 relationship,- Don't have enough bits to make a GUID that is unique, or- Haven't come up with or implemented the algorithm yet.

For example, for IFCRELSPACEBOUNDARY, how do you uniquely define one or more connections between a room and a nearby wall?
This is an example of case 1.

For IFCPROPERTYSET, there is a mechanism in place for some elements that allows for a consistent GUID, so this is partially done and partially case 3.

IFCDOORSTYLE should be handled reliably.
As with most things in IFC, it is a combination of being able to do it and having time to do it.