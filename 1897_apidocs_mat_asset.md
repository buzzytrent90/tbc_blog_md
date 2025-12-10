---
post_number: "1897"
title: "Apidocs Mat Asset"
slug: "apidocs_mat_asset"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'parameters', 'revit-api', 'sheets', 'views', 'windows']
source_file: "1897_apidocs_mat_asset.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1897_apidocs_mat_asset.html"
---

### Forge, RevitApiDocs, Materials and Stacked Items
Here are some exciting Forge, Revit API and other topics for this week:
- [Forge online training in April 2021](#2)
- [RevitApiDocs support for Revit 2021](#3)
- [Welcome, AEC BIM Tools](#4)
- [Visual materials API in Dynamo](#5)
- [24x24 stacked ribbon items](#6)
- [Innovative drone fly-through film](#7)
#### Forge Online Training in April 2021
Are you interested in getting started with the Autodesk Forge platform development, perhaps in a more interactive, guided way?
Or maybe you already have experience with our platform, and are just interested in honing your skills?
If so, come and join us for another series of Forge Training webinars from April 13th until April 16th.
During these days, our dev advocates will guide your through the development of sample applications (using Node.js or .NET) leveraging different parts of Forge, and answer your questions along the way as you develop these applications yourself.
You can refer to Petr Broz's blog post for full details and register
either [here](https://www.eventbrite.com/e/forge-online-training-april-13-16-2021-registration-145580133097) or
there:
[Forge Online Training: April 2021](https://forge.autodesk.com/blog/forge-online-training-april-2021)
#### RevitApiDocs Support for Revit 2021
Back to the Revit API, Gui Talarico just announced an update to the online Revit API documentation for Revit 2021.1:
> [ApiDocs.co](https://apidocs.co) was updated last month and [RevitApiDocs.com](https://www.revitapidocs.com) last Friday!
Very many thanks to Gui for the great news and all his work on this invaluable resource.
We hope to reduce the turn-around time for the next release :-)
![RevitApiDocs for Revit 2021](img/revitapidocs_2021.png "RevitApiDocs for Revit 2021")
#### Welcome, AEC BIM Tools
Simon Jones was one of the first and foremost AEC oriented people at Autodesk for a couple of decades in the previous millennium.
Simon recently left Autodesk after over 35 years with us and launched [AEC BIM Tools](https://www.aecbimtools.com).
He now published his first own Revit add-in, a [Shared Parameter Inspector for Revit](https://www.aecbimtools.com/sharedparameterinspector).
You can download and test run a trial version.
If you find it useful, you can donate what you think it is worth to you.
Good luck and much success in your new adventure, Simon!
#### Visual Materials API in Dynamo
Konrad K Sobon published
a [few more comments about materials in Revit, Dynamo, APIs etc](https://archi-lab.net/few-more-comments-about-materials-in-revit-dynamo-apis-etc).
It explores the use of Dynamo to create materials in Revit from something like an Excel spreadsheet, presents several useful solutions and also points out weaknesses in the current API functionality.
The material appearance asset property mapping image at the end is especially useful and valuable:
![Material appearance asset property mapping](img/archi_lab_material_appearance_asset_properties.jpg "Material appearance asset property mapping")
#### 24x24 Stacked Ribbon Items
Diving deeper into some practical coding, let's look at
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [24x24 StackedItems](https://forums.autodesk.com/t5/revit-api-forum/24x24-stackeditems/m-p/10169950):
\*\*Question:\*\* This may be an easy one, but so far I am struggling to find anything specific about it.
How do you make a `StackedItem` where the icons are 24x24 when there are only 2 in the stack?
It seems like it should be possible as it is used multiple times in the modify tab, cf. this example:
![2-stack 24x24 icons](img/24x24_icon_sizes_1.png "2-stack 24x24 icons")
I have been able to set the `ShowText` property to `false` to get the 3 stacked icons, but, when I use the same method with the 2 icon stack, it remains 16x16, regardless of the icon resolution.
I have tried to obtain and change the button's height and width, minWidth and minHeight through the Autodesk.Window.RibbonItem object to no avail.
Has anyone had any success in creating these icons?
Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) [@CADBIMDeveloper](https://github.com/CADBIMDeveloper) Ignatovich, aka Александр Игнатович, presents an elegant solution with some very pretty code indeed:
\*\*Answer:\*\* I succeeded :-)
With text:
![2-stack 24x24 icons](img/ai_24x24_icon_sizes_2.png "2-stack 24x24 icons")
Without text:
![2-stack 24x24 icons](img/ai_24x24_icon_sizes_3.png "2-stack 24x24 icons")
```csharp
var revitRibbonItem = UIFramework.RevitRibbonControl
.RibbonControl.findRibbonItemById(
ribbonItem.GetId() );
if( useMediumIconSize )
revitRibbonItem.Size = RibbonItemSize.Large;
if( hideButtonCaption )
revitRibbonItem.ShowText = false;
```
`GetId` is an extension method:
```csharp
internal static class RibbonItemExtensions
{
public static string GetId(
this RibbonItem ribbonItem )
{
var type = typeof( RibbonItem );
var parentId = type
.GetField( "m_parentId",
BindingFlags.Instance
| BindingFlags.NonPublic )
?.GetValue( ribbonItem )
?? string.Empty;
var generateIdMethod = type
.GetMethod( "generateId",
BindingFlags.Static
| BindingFlags.NonPublic );
return (string) generateIdMethod?.Invoke(
ribbonItem, new[] {
parentId, ribbonItem.Name } );
}
}
```
Many thanks to Alexander for the nice solution and the succinct, instructive and inspiring coding.
#### Innovative Drone Fly-Through Film
The one-and-a-half-minute drone video [Right Up Our Alley](https://youtu.be/VgS54fqKxf0) may
help drive film making to new heights, lengths, curves and other experiences
– [Hollywood bigwigs shower praise on creators of Minnesota bowling alley drone video](https://www.abc.net.au/news/2021-03-12/hollywood-drone-video-minnesota-bowling-alley/13241718):
> A single-take video shot with a drone flying through a Minnesota bowling alley has been hailed as "stupendous" by a string of celebrities and big-name film-makers.
From there, the drone flies in and around bowlers in the lanes and drinkers at the bar, going in between legs and into the back compartment where the bowling pins are swept up and set up and all around – all in one shot.
It finishes with something of a cliffhanger (SPOILER: No drones were seriously harmed in the making of the video).
Key points:
> - The video is all one-take
- It took about 10 to 12 attempts
- The only thing added in post-production is the audio
- The filmmakers are trying to encourage people to return to bars, restaurants and bowling alleys
- The near 90-second video titled [Right Up Our Alley](https://youtu.be/VgS54fqKxf0) begins outside, the drone swoops in from across the street, through the doors, all around the bowling alley, bar, between people, next to the rolling bowling ball, ...