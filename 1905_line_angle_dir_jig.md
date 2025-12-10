---
post_number: "1905"
title: "Line Angle Dir Jig"
slug: "line_angle_dir_jig"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'references', 'revit-api', 'sheets', 'transactions', 'views', 'windows']
source_file: "1905_line_angle_dir_jig.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1905_line_angle_dir_jig.html"
---

### Refreshment, Cloud Model Path, Angle and Direction
An example of real-time live graphical user input feedback, cloud model functionality, the ever important need to regenerate and other important topics for this holiday day:
- [Line angle and direction jig](#2)
- [Determine cloud model local file path](#3)
- [How to refresh GroupType.Groups](#4)
- [Online access to RevitAPI.chm help files](#5)
- [C++/C# frontend engineer](#6)
#### Line Angle and Direction Jig
The Revit API does not provide much support for real-time feedback on graphical user input, so this original idea
by Maarten van der Linden of [Wagemaker](https://www.wagemaker.nl) may
come in handy, using `PromptForFamilyInstancePlacement` as a workaround, in answering
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to draw the detail line to let the information of direction and angle emerge before finishing the command](https://forums.autodesk.com/t5/revit-api-forum/how-can-i-draw-the-detail-line-to-let-the-information-of/m-p/10291715):
\*\*Question:\*\* How can I draw the Detail Line to let the information of direction and angle emerge before finishing the command?
Currently, I am using `PickPoint` to prompt the use to select line endpoints and creating a detail curve from that information.
However, I see no way to display any useful graphical feedback while picking these points like Revit itself does internally:
![Feedback on pick point angle and direction](img/line_based_detail_item_angle_direction_1.png "Feedback on pick point angle and direction")
![Feedback on pick point angle and direction](img/line_based_detail_item_angle_direction_2.png "Feedback on pick point angle and direction")
\*\*Answer:\*\* You could try to launch the built-in Revit command together with the built-in user interface by using `PostCommand`.
Another option might be to implement some kind of
own [jig, e.g., using the `IDirectContext3DServer`](https://thebuildingcoder.typepad.com/blog/2020/10/onbox-directcontext-jig-and-no-cdn.html#3).,
Maybe you could also use some kind of Windows tooltip to display the required real-time information.
Unfortunately, since this functionality is not supported by the Revit API out of the box, I'm afraid it may be hard to implement anything that is both useful and nice to use.
However, here Maarten comes to the rescue, saying:
> In my command to rotate the crop region of plan view, I use a line-based detail item in combination with `PromptForFamilyInstancePlacement` to mimic that kind of behaviour.
Here is a video of the functionality:
[

](img/line_based_detail_item_direction_arrow_jig.mp4)
Many thanks to Maarten for the great suggestion.
#### Determine Cloud Model Local File Path
We recently discussed the question
of [determining a BIM 360 model's local "absolute" path](https://forums.autodesk.com/t5/revit-api-forum/get-bim-360-model-s-quot-absolute-quot-path/m-p/10292538).
Unfortunately, the initial solution was limited to workshared cloud models.
Now Simon Jones shared a more general method that works with non-workshared cloud models as well, saying:
> Thanks for the original code to determine the file in the local cache for cloud models.
> However, I found it only worked for workshared cloud models.
> This is a revision to work also with non-workshared cloud models:
```csharp
string GetCloudModelLocalCacheFilepath(
Document doc,
string version_number )
{
string title = doc.Title;
string ext = Path.GetExtension( doc.PathName );
string localRevitFile = null;
if( doc.IsModelInCloud )
{
ModelPath modelPath = doc.GetCloudModelPath();
string guid = modelPath.GetModelGUID().ToString();
string folder = "C:\Users\" + Environment.UserName
+ "\AppData\Local\Autodesk\Revit\Autodesk Revit "
+ version_number + "\CollaborationCache";
string revitFile = guid + ext;
string[] files = Directory
.GetFiles( folder, revitFile, SearchOption.AllDirectories )
.Where( c => !c.Contains( "CentralCache" ) )
.ToArray();
if( 0 < files.Length )
{
localRevitFile = files[ 0 ];
}
else
{
Debug.Print( "Unable to find local rvt for: " + doc.PathName );
}
}
else
{
localRevitFile = doc.PathName;
}
return localRevitFile;
}
```
> The key is to use `modelPath.GetModelGUID` rather than `doc.WorksharingCentralGUID`
– thanks @jeremy.tammik for pointing out this function to me (apparently it has been there since 2019.1)...
Many thanks to Simon for sharing this.
I added it
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) to
make it easy to keep track of, cf.
the [diff to the previous release](https://github.com/jeremytammik/the_building_coder_samples/compare/2022.0.150.5...2022.0.150.6).
#### How to Refresh GroupType.Groups
The [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
asking [why `GroupType.Groups` contains ungrouped groups](https://forums.autodesk.com/t5/revit-api-forum/why-does-grouptype-groups-contain-ungrouped-groups/m-p/10291256) demonstrates
another example of
the [need to regenerate](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.33)
and shows that sometimes more than just regenerating the model is required to avoid accessing stale data.
Committing the transaction is the next level up after regeneration:
\*\*Question:\*\* The following code prints the same number twice:
```csharp
var groupType = group.GroupType;
Debug.Print( groupType.Groups.Size.ToString() );
group.UngroupMembers();
Debug.Print( groupType.Groups.Size.ToString() );
```
Is this the correct behaviour?
What do I need to do to make sure groupType.Groups always returns the correct GroupSet?
I tried calling `Regenerate` and that doesn't help.
It looks like I need to end and restart the transaction.
This code shows that GroupType.Groups still reports the ungrouped group after doc.Regenerate() and finally disappears after committing and restarting a new transaction. I tried to get a new reference to the GroupType, hoping that avoiding some caching would help, but it didn't:
```csharp
tx.Start( "Test 1" );
var elementIds = new List
{
Utils.CreateModelLine(new XYZ(0, 0, 0), new XYZ(1, 0, 0), doc).Id,
Utils.CreateModelLine(new XYZ(0, 1, 0), new XYZ(1, 1, 0), doc).Id,
Utils.CreateModelLine(new XYZ(0, 2, 0), new XYZ(1, 2, 0), doc).Id,
};
var group1 = doc.Create.NewGroup( elementIds );
var group2 = doc.Create.PlaceGroup( new XYZ( 0, 5, 0 ), group1.GroupType );
var group3 = doc.Create.PlaceGroup( new XYZ( 0, 10, 0 ), group1.GroupType );
Debug.Print( group1.GroupType.Groups.Size.ToString() );
group2.UngroupMembers();
Debug.Print( group1.GroupType.Groups.Size.ToString() );
doc.Regenerate();
Debug.Print( group1.GroupType.Groups.Size.ToString() );
var groupType = new FilteredElementCollector( doc )
.OfClass( typeof( GroupType ) )
.Cast()
.FirstOrDefault( gt => gt.Name == group1.GroupType.Name );
Debug.Print( groupType.Groups.Size.ToString() );
tx.Commit();
tx.Start( "Test 2" );
Debug.Print( groupType.Groups.Size.ToString() );
groupType = new FilteredElementCollector( doc )
.OfClass( typeof( GroupType ) )
.Cast()
.FirstOrDefault( gt => gt.Name == group1.GroupType.Name );
Debug.Print( groupType.Groups.Size.ToString() );
tx.Commit();
```
I tried using a subtransaction, but that didn't help.
At this point I do have a workaround, but I don't like to create transactions only because I can't rely on GroupType.Groups not reporting ungrouped groups.
Any advice?
\*\*Answer:\*\* Just firing off the top of my head here but... instead of transactions and subtransactions, you might try a transaction group with transactions within it... works just a little differently and perhaps that difference will be a little cleaner in the end...
\*\*Response:\*\* Using a transaction group seem to be the best workaround. Here are the steps:
- transactionGroup.Start
- transaction.Start
- group.UngroupMemebers
- group.GroupType.Groups still contains the ungrouped group
- transaction.Commit
- group.GroupType.Groups does not contain the ungrouped group
Unfortunately, such a workaround cannot be used in a generic case.
Wrapping the call to `UngroupMembers` inside its own transaction only because any following `groupType.Groups` would return the wrong result is ugly, because it forces the caller to use a transaction group rather than a transaction.
\*\*Answer:\*\* I recently had a somewhat similar problem (working w/ completely different objects though). It seems like in some cases the value returned by a Revit API object's property or method is always the same as when it was first obtained from the API, even after regenerating the model. To resolve this in my case I had to obtain a new copy of the object from the API after regenerating the model (e.g., myElement = myDocument.GetElement(myElementId);). I don't know if this will help in your situation, but I hope it does.
\*\*Response:\*\* Thanks for the advice.
I just tried and in this stale case the cached info sticks around even after getting a new element.
Adding these two lines before closing the transaction to my code, still return the wrong value:
```csharp
group1 = doc.GetElement( group1.Id ) as Group;
Debug.Print( groupType.Groups.Size.ToString() );
groupType = doc.GetElement( group1.GroupType.Id ) as GroupType;
Debug.Print( groupType.Groups.Size.ToString() );
```
#### Online Access to RevitAPI.chm Help Files
Guy Talarico set up a repository sharing the RevitAPI.chm Windows help file provided by the Revit SDK for all Revit API releases reaching back to Revit 2012.
These files are used to generate the online Revit API documentation
resources [revitapidocs.com](https://www.revitapidocs.com/)
and [apidocs.co](https://apidocs.co/) that
provide web presentations of the same content.
Due to the growing size of the CHM files, we were forced to turn on
the [Git Large File Storage (LFS)](https://git-lfs.github.com/) and move the repository to a new location in the ADN-DevTech organisation
at [github.com/ADN-DevTech/revit-api-chms](https://github.com/ADN-DevTech/revit-api-chms).
#### C++/C# Frontend Engineer
Friend and colleague Carlo asked me to point out
a [career opportunity for a C++/C# frontend engineer](https://www.comeet.com/jobs/knauf/56.004/cc-frontend-engineer-mfd/9F.D1F) with
Knauf in Germany to develop and deliver solutions for CAD programs, e.g., Revit and ArchiCAD, mixing native SDK technologies in C++, .NET and web views.