---
post_number: "1767"
title: "Ifc Geo Inst"
slug: "ifc_geo_inst"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'levels', 'parameters', 'revit-api', 'sheets', 'windows']
source_file: "1767_ifc_geo_inst.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1767_ifc_geo_inst.html"
---

### IFC Update, Instance Geometry, Parameters
I completed the move to my new computer, and happily all systems go now.
Here are some other topics that came up in the past few days:
- [Revit-IFC Release 20.1.0](#2)
- [Community discussion on Revit](#3)
- [Solid or instance, depending](#4)
- [Exporting parameters to Access](#5)
- [Store globals on custom `DataStorage`, not `ProjectInfo`](#6)
![Mahab Ghodss civil engineering app](img/mahabghodss_video.png)

Mahab Ghodss civil engineering project ([video](https://thebuildingcoder.typepad.com/files/mahabghodss_video.mp4))

- [Iranian civil engineering project video](#7)
#### Revit-IFC Release 20.1.0
[Release 20.1.0 for Revit 2020](https://github.com/Autodesk/revit-ifc/releases/tag/IFC_v20.1.0) has been released.
It includes 20.0.0.0 plus the out-of-the-box Revit 2020 IFC functionalities, bug fixes and various regression fixes.
Says Angel [@avelezsosa](https://twitter.com/avelezsosa) Velez:
> Finally! [#Revit](https://twitter.com/hashtag/Revit?src=hash&ref_src=twsrc%5Etfw) [#IFC](https://twitter.com/hashtag/IFC?src=hash&ref_src=twsrc%5Etfw) [#OpenSource](https://twitter.com/hashtag/OpenSource?src=hash&ref_src=twsrc%5Etfw) [#GitHub](https://twitter.com/hashtag/GitHub?src=hash&ref_src=twsrc%5Etfw) v20.1 is out! <https://t.co/TmCc62uDjA>. Up next: App Store version, v19.3.
>
> — Angel Velez (@avelezsosa) [August 8, 2019](https://twitter.com/avelezsosa/status/1159476876538187777?ref_src=twsrc%5Etfw)

#### Community Discussion on Revit
In the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
Daniel Alvarado invites all Revit API users to
a [community discussion on Revit](https://forums.autodesk.com/t5/revit-api-forum/community-discussion-revit/m-p/8962297), saying:
> I found an interesting article on the Revit Blog.
I am sharing this article by Amie Vaccaro to discuss
> - [How ongoing innovations have made Revit more automated than ever](https://blogs.autodesk.com/revit/2019/08/12/how-ongoing-innovations-have-made-revit-more-automated-than-ever)
> To participate in the discussion, write your comments, thoughts, and questions and feel free to agree or disagree with the ideas presented in the article. I would like to hear all kinds of feedback :-)
> Discussion questions:
> - What features of Revit do you wish were automated?
> - Now that you know what features are automated, which one is most useful for you?
> We invite anyone to participate in these discussions and look forward to animated conversations!
#### Solid or Instance, Depending
Also in the discussion forum, in the thread asking why
the [solid element is missing](https://forums.autodesk.com/t5/revit-api-forum/solid-element-is-missing/m-p/8950786),
Bobby C Jones points out two important basic knowledgebase articles clarifying how to retrieve geometry from family instances:
First, the article
on [GeometryInstances](https://knowledge.autodesk.com/search-result/caas/CloudHelp/cloudhelp/2016/ENU/Revit-API/files/GUID-B4F83374-0DF6-4737-91EB-900E676E862B-htm.html) explains:
> Note that not all Family instances will include GeometryInstances.
When Revit needs to make a unique copy of the family geometry for a given instance (because of the effect of local joins, intersections, and other factors related to the instance placement), no GeometryInstance will be encountered; instead, the Solid geometry will be found at the top level of the hierarchy.
It also points to the helpful [Example: Retrieve Geometry Data from a Beam](https://knowledge.autodesk.com/search-result/caas/CloudHelp/cloudhelp/2016/ENU/Revit-API/files/GUID-F092BCCC-77E9-4DA9-9264-10F0DB354BF5-htm.html), which reiterates:
> The `GeometryElement` may contain the desired geometry as a `Solid` or `GeometryInstance`, depending on whether a beam is joined or standalone, and this code covers both cases...
If this is unclear to you, please refer to the detailed knowledgebase explanations.
#### Exporting Parameters to Access
Another short note from the discussion forum
on [how to export selected parameters to MS access using my own ribbon](https://forums.autodesk.com/t5/revit-api-forum/how-to-export-selected-parameters-to-ms-access-using-my-own/m-p/8960356):
\*\*Question:\*\* If I want to create my own ribbon within add-ins menu of Revit 2019. And when I click the ribbon, it will show the window that asks me which parameters I want to export to MS access file. (I want to be able to select the required parameters of each element.) Can you suggest any API sample code? Thank you.
\*\*Answer:\*\* Look at the [ADN Xtra labs](https://github.com/jeremytammik/AdnRevitApiLabsXtra),
in the `XtraCs` project, module Lab4.cs, especially the external
commands [Lab4_1_ElementParameters](https://github.com/jeremytammik/AdnRevitApiLabsXtra/blob/master/XtraCs/Labs4.cs#L45-L266)
and [Lab4_2_ExportParametersToExcel](https://github.com/jeremytammik/AdnRevitApiLabsXtra/blob/master/XtraCs/Labs4.cs#L268-L509).
#### Store Globals on Custom DataStorage, not ProjectInfo
Talking about parameters, I implemented an add-in to round-trip Revit element parameters to Forge and back, enabling
the [use of Forge or an external spreadsheet to create shared parameters](https://thebuildingcoder.typepad.com/blog/2017/09/use-forge-or-spreadsheet-to-create-shared-parameters.html).
A recent [comment](https://thebuildingcoder.typepad.com/blog/2017/09/use-forge-or-spreadsheet-to-create-shared-parameters.html#comment-4568543582) on that once again brought up the topic of storing global information in the model.
If you wish to do so, do not store it on the `ProjectInfo` singleton element; rather, you can and should create your own `DataStorage` element for this purpose, as explained in the note
on [storing a dictionary – use `DataStorage`, not `ProjectInfo`](https://thebuildingcoder.typepad.com/blog/2016/11/1500-posts-devday-and-storing-a-dictionary.html#5).