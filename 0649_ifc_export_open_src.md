---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.329269'
original_url: https://thebuildingcoder.typepad.com/blog/0649_ifc_export_open_src.html
post_number: 0649
reading_time_minutes: 4
series: general
slug: ifc_export_open_src
source_file: 0649_ifc_export_open_src.htm
tags:
- elements
- parameters
- revit-api
title: Revit IFC Exporter Released as Open Source
word_count: 777
---

### Revit IFC Exporter Released as Open Source

In June, Autodesk released the
[Revit STL exporter as open source](http://thebuildingcoder.typepad.com/blog/2011/06/revit-2012-stl-exporter-released-as-open-source.html).
Now we are kicking of another even more exciting new open source project, really hot news for all Revit developers, especially if you are interested in IFC.
Before we get to that, here is another little snippet on my weekend activities.

#### Clariden Mountain

I spent last Saturday in the Swiss Alps, climbing the
[Clariden Mountain](http://www.summitpost.org/clariden/152997) with
my friend Martin.
It was not really a climb, just a hike up the mountainside for one thousand metres, easily doable in trekking sandals, followed by another couple of hundred after putting on boots and crampons to climb up the Iswändli and cross the glacier:

![On the Chammlijoch glacier](file:////j/photo/jeremy/2011/2011-09-09_hochtour_clariden/046.jpg)

After the glacier comes a final scramble over the Clariden Vorgipfel and the ridge up to the summit at 3268 m:

![Martin and Jeremy on the Clariden summit](file:////j/photo/jeremy/2011/2011-09-09_hochtour_clariden/043.jpg)

We returned back down the same way we came up, all in the most beautiful weather imaginable, possibly but hopefully not the last weekend this year with such fantastic mountain conditions.
Here are some
[more photos](https://picasaweb.google.com/104316998805199988071/Clariden3267mMitJeremy?authuser=0&authkey=Gv1sRgCPetsoCsmrfxvgE&feat=directlink) and
the GPS track of our route on a map.

Returning to Revit programming, let's have a look at this exciting new open source project:

#### Drive Data Interoperability in AEC Industries

*Autodesk Revit Industry Foundation Classes (IFC) Exporter Code Now Available as Open Source Software; Supporting Greater Interoperability with Compliant Software*

Autodesk announced that the Industry Foundation Classes (IFC) exporter for its Revit products is now accessible as open source code, licensed through a LGPL v. 2.1 licensing agreement.
Since 2005, Autodesk Revit products have provided IFC file export, making it possible to export replicas of project models to the standard IFC file format.
IFC files can then be imported into any design program that is compliant with the same standard, helping to support greater interoperability in the architecture, engineering and construction (AEC) industry.

Enabling the Revit IFC exporter code to be licensed as open source software provides users of Autodesk Revit products, including Autodesk Revit Architecture 2012, Autodesk Revit MEP 2012 and Autodesk Revit Structure 2012, with greater flexibility to customize their Revit IFC file output to help them better meet the needs of specific project or government IFC file input requirements.
With access to the Revit IFC exporter source code, users can add custom parameter sets to elements exported to IFCs, or custom quantities to the elements exported.
Users may also, for example, change the representation of the exported elements, should they find another, more useful encoding.

"For several years now, our customers have been asking for greater flexibility with the Revit IFC file format output," said Jim Lynch, vice president, Building and Strategic Technology Group, Architecture, Engineering and Construction Solutions, Autodesk.
"The decision to release the Revit IFC exporter code as open source furthers our ongoing commitment to support the IFC standard, and marks the latest demonstration of the Autodesk drive to encourage full data exchange within a Building Information Modelling workflow."

The Revit IFC exporter open source code is managed by a five-member steering committee composed of one Autodesk employee and four members of the AEC Building Information Modelling (BIM) community. The Revit IFC Exporter Open Source Committee is chaired by Emile Kfouri, BIM application development manager, Architecture, Engineering and Construction Solutions, Autodesk.

#### Availability and More Information

The Revit IFC exporter open source code is now available.
Access to the Revit IFC exporter open source code, as well as information regarding the Revit IFC Exporter Open Source Committee, can be found at the SourceForge repository at
[sourceforge.net/projects/ifcexporter](http://sourceforge.net/projects/ifcexporter).
For more information about the IFC standard visit the buildingSMART organization site at
[www.buildingsmart.org](http://www.buildingsmart.org).

For some more background information on this project and the reasoning behind it, please have a look at
[BIM Apps](http://bimapps.typepad.com/bim-apps/2011/09/ifc-exporter-for-revit-2012-is-released-as-open-source.html).

#### Book Recommendation: The Cathedral and the Bazaar

For a fascinating background on open source development and to understand in depth both the motivation of open source contributors and the advantages of this approach over a closed shop, I can highly recommend
[The Cathedral and the Bazaar](http://en.wikipedia.org/wiki/The_Cathedral_and_the_Bazaar) by
Eric Raymond.