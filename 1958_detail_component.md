---
post_number: "1958"
title: "Detail Component"
slug: "detail_component"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'python', 'references', 'revit-api', 'sheets', 'transactions', 'views']
source_file: "1958_detail_component.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1958_detail_component.html"
---

### Tag Extents and Lazy Detail Components
Today, let's highlight two really nice contributions from the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160):
- [Determining tag extents](#2)
- [Unrotate rotated tags](#2.2)
- [One-click detail family generator](#3)
First, though, a little aphorism to ponder:

Yesterday, I was clever and tried to change the world.

Today, I am wise and try to change myself.

– Rumi

#### Determining Tag Extents
We repeatedly discussed how to ensure that tags do not overlap, both here and in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
e.g., in the threads
on [tags without overlapping](https://forums.autodesk.com/t5/revit-api-forum/tags-without-overlapping/m-p/11275579)
and [auto tagging without overlap](https://forums.autodesk.com/t5/revit-api-forum/auto-tagging-without-overlap/td-p/9996808).
A hard-coded algorithm to achieve partial success was presented in the latter and reproduced
in [Python and Dynamo autotag without overlap](https://thebuildingcoder.typepad.com/blog/2021/02/splits-persona-collector-region-tag-modification.html#5).
A more complete solution using a more advanced algorithm is now available commercially,
called [Smart Annotation](https://bimlogiq.com/products/smart-annotataion) by [BIMLOGiQ](https://bimlogiq.com).
One prerequisite for achieving this task is determining the extents of a tag.
[AmitMetz](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/9455666) very
kindly shares sample code for a method to achieve this in the thread
on [tag width/height or accurate `BoundingBox` of `IndependentTag`](https://forums.autodesk.com/t5/revit-api-forum/tag-width-height-or-accurate-boundingbox-of-independenttag/m-p/11274095).
Says he:
Following the helpful comments above, here is a method that returns tag dimensions.
A few comments on the implementation:
- First, we need to make sure the LeaderEndCondition is free in order to find the LeaderEndPoint.
- Move the tag and it's elbow to LeaderEndPoint.
- We get the correct BoundingBox only after moving the tag and it's elbow, and committing the Transaction.
- I tried to use an unwrapped `transaction.rollback` without `TransactionGroup`, but it didn't work.
So, if we want to keep the tag in its original location, we have to commit the transaction and then roll back the transaction group.
```csharp
///
/// Determine tag extents, width and height
/// summary>
public static Tuple GetTagExtents(
IndependentTag tag)
{
Document doc = tag.Document;
//Dimension to return
double tagWidth;
double tagHeight;
//Tag's View and Element
View sec = doc.GetElement(tag.OwnerViewId) as View;
XYZ rightDirection = sec.RightDirection;
XYZ upDirection = sec.UpDirection;
Reference pipeReference = tag.GetTaggedReferences().First();
//Reference pipeReference = tag.GetTaggedReference(); //Older Revit Version
using (TransactionGroup transG = new TransactionGroup(doc))
{
transG.Start("Determine Tag Dimension");
using (Transaction trans = new Transaction(doc))
{
trans.Start("Determine Tag Dimension");
tag.LeaderEndCondition = LeaderEndCondition.Free;
XYZ leaderEndPoint = tag.GetLeaderEnd(pipeReference);
tag.TagHeadPosition = leaderEndPoint;
tag.SetLeaderElbow(pipeReference, leaderEndPoint);
trans.Commit();
}
//Tag Dimension
BoundingBoxXYZ tagBox = tag.get_BoundingBox(sec);
tagWidth = (tagBox.Max - tagBox.Min).DotProduct(rightDirection);
tagHeight = (tagBox.Max - tagBox.Min).DotProduct(upDirection);
transG.RollBack();
}
return Tuple.Create(tagWidth, tagHeight);
}
```
Many thanks to Amit for this nice implementation!
#### Unrotate Rotated Tags
Steven Micaletti, VDC Software & Technology Developer added
a [comment on this method](https://www.linkedin.com/feed/update/urn:li:activity:6952994276876169216?commentUrn=urn%3Ali%3Acomment%3A%28activity%3A6952994276876169216%2C6953115731991433216%29), saying:
> Good stuff; however, the GetTagExtents() implementation is missing a critical step – rotating the Pipe.
The Pipe in the thread is at an angle, while the tag is not, and this is not generally how MEP elements are tagged.
MEP tag families usually have the "rotate with component" option enabled, and in that scenario the bounding box returned is unusable.
We must first disconnect and rotate the pipe to be model axis aligned before we set the TagHeadPosition, get the BoundingBox, and RollBack() the Transaction.
Thank you, Steven, for pointing this out!
#### One-Click Detail Family Generator
Another nice solution and entire open source sample add-in is shared by
Peter [PitPaf](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/12564927) of [Piotr Żuraw Architekt](https://www.zurawarchitekt.pl)
presenting [one click convert detail elements to detail family](https://forums.autodesk.com/t5/revit-api-forum/one-click-convert-detail-elements-to-detail-family/td-p/11230155):
> I'm working on a Revit add-in to automate and simplify creation of detail Components families.
> This is helps create detail components on the fly just in model view.
> It allows the user to draw parts of a detail with lines and fill regions in model view and change it to a component without opening the family editor.
> Here I want to share with you the first version of this add-in, including source code and compiled install files:
>

[github.com/PitPaf/LazyDetailComponent](https://github.com/PitPaf/LazyDetailComponent)

> Feel free to use it if you find it interesting. I appreciate all your comments.
![Lazy detail component](img/lazy_detail_component.png "Lazy detail component")
Many thanks to Peter for implementing, documenting and sharing this nice solution!