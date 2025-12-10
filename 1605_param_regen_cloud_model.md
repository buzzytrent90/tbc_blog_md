---
post_number: "1605"
title: "Param Regen Cloud Model"
slug: "param_regen_cloud_model"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'parameters', 'revit-api', 'schedules', 'sheets', 'views']
source_file: "1605_param_regen_cloud_model.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1605_param_regen_cloud_model.html"
---

### Cloud Model Predicate, and Set Parameter Regenerates
The [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) continues
to reach ever new levels of depth and coverage.
Here are a couple of recent topics:
- [Welcome to the top solution authors, Jim!](#2)
- [Setting a parameter to regenerate the model](#3)
- [Checking model for C4R versus local file](#4)
#### Welcome to the Top Solution Authors, Jim!
The breadth and depth
of [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) solutions
can only be achieved thanks to a growing amount of input from real developers – unlike myself and my developer support colleagues – providing advanced answers to hitherto unsolved problems.
They are honoured in the list of top solution authors:
![Top solution authors](img/2017-11-28_top_solution_author.png)
Very many thanks as always to
Rudi [@Revitalizer](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1103138) Honke,
Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) Ignatovich and
Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen
for sharing their professional experience and ideas that most of us others would never be able to come up with.
My Chinese colleague Jim Jia has also been participating in the forum for quite a while.
Now he made it into the list of top solution authors as well.
Congratulations, Jim, and thank you very much for all your work!
#### Setting a Parameter to Regenerate the Model
We have frequently discussed
the [need to regenerate](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.33) the
model or individual elements and various ways to achieve that efficiently.
Another aspect of this and a simple solution came up in the thread
on [new sloped roof not visible](https://forums.autodesk.com/t5/revit-api-forum/new-sloped-roof-not-visible/m-p/7574411):
\*\*Question:\*\* I have a problem creating a new sloped roof using code.
I use the basic sample shown in the developer guide and The Building Coder article
on [creating a roof](http://thebuildingcoder.typepad.com/blog/2015/09/revit-answer-day-and-creating-a-roof.html).
I create a new `FootPrintRoof`, set `DefinesSlopes` to true for each model curve, and assign a `SlopeAngle`.
My macro doesn't have errors, but I can't see the new roof in any view.
I can find it only in a roof schedule, but I cannot see the 3D element.
I have tried to refresh the view in the code and I have noticed that the roof appears on the screen only for one second and then it disappears again.
I have tried using Basic roof and Sloped glazing, but it still doesn't work.
If I don't set the sloping, I can create a planar roof without any issue
What is the problem with the slope angle?
How can I solve that and make the sloped roof visible?
\*\*Answer:\*\* I solved the problem.
Now, I create my sloped glazing and then simply set one of its parameters using the API.
I've tried many different parameters, using either parameters connected to the UI and descriptive parameters: all of them are ok to regenerate the correct visualization of the sloped glazing.
This is my trick, I hope it is useful!
If anyone has a better way, please let me know.
Many thanks
to [@newbieng](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/3379844) for
raising the issue and sharing this simple and effective solution!
#### Checking Model for C4R versus Local File
A new issue was raised and solved in the long discussion
on [browsing model files in the cloud](https://forums.autodesk.com/t5/revit-api-forum/browsing-model-files-in-the-cloud-a360-c4r/m-p/6537130):
\*\*Question:\*\* Does anyone know how to check whether this file is C4R versus a local file?
Is it simply file extension, or a document property?
\*\*Answer:\*\* There's an internal property `IsModelInCloud` on the `Document` object in the Revit API that you can access using reflection:
```csharp
public static bool GetIsModelInCloud(
Document document )
{
PropertyInfo p = typeof( Document ).GetProperty(
"IsModelInCloud",
BindingFlags.NonPublic | BindingFlags.Instance );
return (bool) p.GetValue( document );
}
```
Many thanks to
Paul [@pvella](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/561100) Vella
for sharing this neat little secret!