---
post_number: "1441"
title: "Accel Render Asset"
slug: "accel_render_asset"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'parameters', 'revit-api', 'rooms', 'selection', 'sheets', 'views', 'walls']
source_file: "1441_accel_render_asset.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1441_accel_render_asset.html"
---

### Roomedit3d Console Test and Rendering Assets
I returned from the trip to Barcelona last week for the [Forge](http://forge.autodesk.com) [Accelerator](http://autodeskcloudaccelerator.com) and was immediately inundated with overdue tasks.
One issue that I addressed has to do with [rendering assets](#3).
In Barcelona, I also just managed to finish reading the Spanish novel [\*La Tempestad\*](#4).
#### Roomedit3d Revit-Independent Implementation Aspects
Before getting to that, let me point out that I started an exciting new project dubbed `roomedit3d` connecting BIM and the cloud last week as well, demonstrating two cool possibilities to enhance interaction with
the [View and Data API](https://developer.autodesk.com/api/view-and-data-api):
- A viewer extension enabling interactive model modification, i.e., translation of selected elements, based on Philippe Leefsma's [TransformTool](http://adndevblog.typepad.com/cloud_and_mobile/2015/08/moving-visually-your-components-in-the-viewer-using-the-transformtool.html)
- Real-time communication of the modification back to the source CAD system using:
- A REST API POST call from the viewer extension to the node.js web server
- A direct [socket.io](http://socket.io) connection to broadcast from the web server to any number of desktop clients
[The 3D Web Coder](http://the3dwebcoder.typepad.com) already described all its Revit-independent implementation aspects in full detail:
- [Roomedit3d](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#3)
- [It's Read-Only! How can it be Read-Write?](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#4)
- [Genealogy or Where to Start?](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#5)
- [Capturing the TransformTool Selection](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#6)
- [Determining the TransformTool Translation Vector](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#7)
- [POST From Viewer to Server](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#8)
- [Broadcast via Socket.io](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#9)
- [Desktop Notification Connection and Subscription](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#10)
- [Demo Recording](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#11)
- [Download](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#12)
- [To Do](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#13)
#### Rendering Assets
Back to the pure Revit API, I have been asked repeatedly about the possibilities to access and manipulate textures and other rendering assets.
Here is the latest incarnation of such a question:
\*\*Question:\*\* The Revit API enables creating structural and thermal assets and setting their properties with the help of the `StructuralAsset` and `ThermalAsset` classes.
However, I don't see any similar classes for appearance assets.
The only related class I can find is the `AppearanceAssetElement`, and this class seems to work completely differently.
Can I create a new `AppereanceAsset` or duplicate an existing one and edit its properties?
Or is there another way to edit the appearance asset of a material?
Furthermore, it appears that the Render Appearance is stored in an `.adsklib` file, and this is a shared format between different Autodesk products.
Is it not possible to edit the Revit render appearance asset in Revit itself, would it be possible to do so in some other program and then import it into Revit?
\*\*Answer:\*\* Yes, you can edit and create AppearanceAssetElement objects.
Look at The Building Coder discussion of [What's New in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html):
#### Materials API changes
\*\*Applying visual materials\*\*
The method
- Material.SetRenderAppearance()
has been deprecated. The Render appearance properties should be set via the related `ApperanceAssetElement`.
Use the new property:
- Material.AppearanceAssetId
to assign the element to the material.
AppearanceAssetElements can be found via element filtering – they expose the following members:
- AppearanceAssetElement.Create() – creates a new asset element for a given rendering Asset and name.
- AppearanceAssetElement.GetAppearanceAssetElementByName() – gets an asset element handle given the name.
- AppearanceAssetElement.SetRenderingAsset() – Sets the rendering Asset to the element
Afaik, the only way to obtain full coverage to edit all the aspects of an appearance asset from within Revit is via the user interface.
From the Revit API point of view, it is pretty much a black box.
Regarding the use of the `.adsklib` file to exchange assets between different Autodesk products:
Yes, this is indeed possible, as described in some of the following suggestions:
A. You might want to take a look at Aaron Lu's explanation and sample code showing [how to access a material's asset properties](http://adndevblog.typepad.com/aec/2015/03/revitapi-how-to-get-asset-properties-of-material-i-want.html).
B. Unfortunately, as said, we do not have full access to all asset properties, e.g., here is a request that we cannot currently satisfy:
[Q] My family has a parameter that defines a colour for this component (by default, the parameter is assigned a material with a default colour). I want to create a new material with a specific colour. This works fine. The problem comes when I switch to realistic. The component takes the colour from an Appearance Asset. How can I create and assign a new Asset for a Material? Probably, the fix will come by checking the option 'Use Render Appearance' from the Shading properties and assigning the colour to the created Asset. Does the Revit API provide support for this?
[A] Unfortunately the API currently does not provide full access to all assets.
C. You might also want to check out The Building Coder topic group on [Material Management and Libraries](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.5).
D. If your workflow requires adding a material that is not currently available, one approach might be to copy and paste it from some other Revit file, or loading an asset from some other product like Inventor via an ADSK package:
- [Loading an Inventor ADSK component](http://thebuildingcoder.typepad.com/blog/2011/12/loading-an-inventor-adsk-component.html)
- [ADSK file import](http://thebuildingcoder.typepad.com/blog/2012/08/adsk-file-import-and-phase-of-room.html)
E. Furthermore, you might want to check the Revit API discussion forum thread on [Revit API materials and textures](http://forums.autodesk.com/t5/revit-api/revit-api-materials-and-textures/m-p/6319370).
The latest comment there by Scott Wilson is especially interesting:
> I have a finite set of actual textures the will be used which doesn't change very often (Basically it's just the set of standard steel roofing colours but the specified sheet thickness and coating type will change for each situation.) I also have control over the project templates that are used so I created a set of template materials with structured naming that identify the texture applied. When I need a new textured material I find the appropriate template material, duplicate it and set the other properties as needed. I have built a little framework around this that allows the user to add new template materials also if they follow the same structured naming scheme. I know this won't work for many, but I just thought I'd share my limited workaround for those it might help.
Finally, let me point out an existing wish list item to enhance this access, CF-234 [API wish: apply photo texturing -- 08963731].
#### La Tempestad
Talking about something unrelated to Revit and its API, my reading has radically decreased lately.
I just finished the first book in months, though, \*La Tempestad\* by [Juan Manuel de Prada](https://es.wikipedia.org/wiki/Juan_Manuel_de_Prada).
It won the Spanish [Premio Planeta](https://en.wikipedia.org/wiki/Premio_Planeta_de_Novela) prize
in 1997 and came highly recommended by
Jose Ignacio [@montesherraiz](https://github.com/Montesherraiz) Montes,
of [Avatar BIM](http://avatarbim.com) in Madrid, during
the [BIM Programming Workshop](http://www.bimprogramming.com) in January:
I like it a lot.
It is a bit strange, beautiful, naive, poetical, melancholy, about an Spanish art expert visiting and learning to hate the city of Venice, falling in love rather precipitously, deeply and desperately, then returning to continue a wasted academic life back in Spain.
Here are two quotes that I enjoyed from the last few pages:
Pero yo sabia que la verdad no existe, se compone de falsedades parciales o piadosas, la verdad depende de quien la formula, nadie es infalible ni omnisciente ni bienintencionado.
\*But I knew that truth does not exist, it consists of partial or merciful falsehoods, truth depends on the speaker, no one is infallible or omniscient nor well-intentioned.\*
Otros rostros se alejan y precipitan en la común argamasa del olvido, pero no el de Chiara.
A veces mientras me lavo y me enfrente conmigo mismo ante el espejo del lavabo, me pregunto que seria de mi si me faltase ese consuelo y esa condena.
Como aun no estoy despejado del todo y las legañas y el abotargamiento embrutecen mi expresión, encuentro en seguida la respuesta: seria un animal que no se comprende a si mismo, atrapado entre un presente mostrenco y un futuro que no existe.
Así, por lo menos, tengo un pasado, y lo rememoro, y lo habito.
\*Other faces fade away and fall into the general mud of oblivion, but not Chiara's.
Sometimes when washing in the morning I face myself in the mirror above the basin and I wonder what would have become of me without this consolation and condemnation.
Still gummy and puffy, my expression is brutalized and the answer clear: I would be an animal that does not understand itself, caught between a straying present and a non-existent future.
As it is, at least, I have a past, and re-remember it, and inhabit it.\*