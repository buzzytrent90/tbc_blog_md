---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.9
content_type: code_example
optimization_date: '2025-12-11T11:44:17.169100'
original_url: https://thebuildingcoder.typepad.com/blog/1974_64_bit_element_id.html
post_number: '1974'
reading_time_minutes: 7
series: elements
slug: 64_bit_element_id
source_file: 1974_64_bit_element_id.md
tags:
- csharp
- elements
- family
- parameters
- python
- revit-api
- rooms
- schedules
- sheets
- views
title: 64 Bit Element Id
word_count: 1355
---

### 64-Bit Element Ids, Maybe?
We may need to scale up the handling of element id integer values in future, a sample snippet to retrieve schedule headers, a Dynamo book, and a web-based family showroom browser:
- [64-bit element ids](#2)
- [Amendment – how to handle overflow](#2.1)
- [Revit schedule title headers](#3)
- [Beyond Dynamo: Python manual for Revit](#4)
- [Web-based family management showroom](#5)
- [Tree view in pure CSS](#6)
- [High-documentation, low-meeting work culture](#7)
#### 64-Bit Element Ids
Autodesk almost never discusses any upcoming functionality in future products whatsoever, for legal reasons.
Still, just as a heads-up warning, the development team thought it worthwhile to point out that we are thinking about possibly converting the internal representation of Revit element ids from 32 to 64 bit in a future release of Revit.
Here is a writeup of what that would imply and what a developer would need to do about it:
ElementIds will change from storing 32-bit integers to storing 64-bit values (type `long` in C# or `Int64` in .NET).
This will allow for larger and more complicated models.
Most functions which take or return ElementIds will continue to work with no changes.
However, there are a few things to keep in mind:
If you are storing ElementIds externally as integers, you will need to update your storage to take 64-bit values.
The constructor Autodesk.Revit.DB.ElementId(Int32) has been deprecated and replaced by a new constructor, Autodesk.Revit.DB.ElementId(Int64).
The property Autodesk.Revit.DB.ElementId.IntegerValue has been deprecated.
It returns only the lowest 32-bits of the ElementId value.
Please use the replacement property Autodesk.Revit.DB.ElementId.Value, which will return the entire value.
Support has been added for using 64-bit types in ExtensibleStorage.
Both Autodesk.Revit.DB.ExtensibleStorage.SchemaBuilder.AddSimpleField() and AddMapField() can now take in 64-bit values for the fieldType, keyType, and valueType parameters.
Two binary breaking changes have been made.
Both Autodesk.Revit.DB.BuiltInCategory and Autodesk.Revit.DB.BuiltInParameter have been updated so that the size type of the underlying enums are 64-bits instead of 32.
Code built against earlier versions of the API may experience type cast and other type related exceptions when run against the next versions of the API when working with the enum.
Please rebuild your addins against the next release's API when it is available.
Here is an overview mapping the deprecated to the replacement members:
- Autodesk.Revit.DB.ElementId(Int32)
 → Autodesk.Revit.DB.ElementId(Int64)
- Autodesk.Revit.DB.ElementId.IntegerValue
 → Autodesk.Revit.DB.ElementId.Value
#### Amendment – How to Handle Overflow
We have an amendment to add to the original post.
One of the things we originally said above has actually changed, and I think it also helps
address [cadferret’s question below](https://thebuildingcoder.typepad.com/blog/2022/11/64-bit-element-ids-maybe.html#comment-6054377627):
With the caveat that anything we say here might change, we want to amend the initially published proposal.
The previous version stated that `ElementId.IntegerValue` would
handle [integer overflow](https://en.wikipedia.org/wiki/Integer_overflow) by
truncating 64-bit values down to 32 bits.
For values which will fit in 32 bits, we will return the value as an integer.
However, if the value would actually need more than 32 bits to represent it, we will throw an exception.
To add a bit more about our intentions, as things currently stand:
Our intention is to find an optimal balance between 64-bit readiness and minimising disruption for API developers.
We would NOT remap existing ElementIds to higher values.
An `Element` with an `Id` of 50 would still have an `Id` of 50, and either property would return the correct value.
Most models will not get so large as to exhaust the 32-bit id space.
So, in general, `ElementId.IntegerValue` would still work.
This would give developers a chance to update their applications, rather than having the function immediately disappear.
However, if a model were so large as to have Ids that needed more than 32 bits to store the value, the `ElementId.IntegerValue` property would throw an exception, return a truncated value, or something similar.
This is an attempt to allow models which need 64-bit ids sooner, simultaneously minimising outright breaking changes in the API.
#### Revit Schedule Title Headers
A number of developers asked for a snippet of sample code in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [Revit schedule title/headers](https://forums.autodesk.com/t5/revit-api-forum/revit-schedule-title-headers/m-p/11573145).
Hernan [H.echeva](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/3063892) Echevarria
kindly jumped in and shared his implementation:
> I found this post, which was very helpful; thank you for the info.
> I created a small example macro that gets the header text.
I hope this helps:
```csharp
///
/// This macro gets the header text of the active Schedule View
/// summary>
public void ScheduleHeader()
{
UIDocument uidoc = this.ActiveUIDocument;
Document doc = uidoc.Document;
ViewSchedule mySchedule = uidoc.ActiveView as ViewSchedule;
TableData myTableData = mySchedule.GetTableData();
TableSectionData myData = myTableData.GetSectionData(SectionType.Header);
TaskDialog.Show("Header Info", "The Header Text is: \n" + myData.GetCellText(0, 0));
}
```
Thank you, Hernan!
#### Beyond Dynamo: Python Manual for Revit
I avoid advertising commercial products, but I made an exception
for [Más Allá de Dynamo](https://thebuildingcoder.typepad.com/blog/2020/12/dynamo-book-and-texture-bitmap-uv-coordinates.html#3),
the Spanish-Language Python manual focused on Dynamo and the Revit API.
It is now also available in English
as [Beyond Dynamo: Python manual for Revit](https://www.amazon.com/dp/B0BMSV6YXD),
by Kevin Himmelreich, Alejandro Martín-Herrer and Ignacio Moreu:
> This is a Python handbook specifically created for working on BIM methodology with Autodesk Dynamo and Revit.
It has a practical approach aimed at professionals who have never programmed before.
If you know how to program in Python, it will be equally useful, as it explains deeply most of the classes, methods and properties of the Revit API.
![Beyond Dynamo](img/beyond_dynamo_en.png "Beyond Dynamo")
Note that we also discussed lots of other resources
for [learning Python and Dynamo](https://thebuildingcoder.typepad.com/blog/2021/02/addin-file-learning-python-and-ifcjs.html#3).
#### Web-Based Family Management Showroom
Emiliano Capasso, Head of BIM at [Antonio Citterio Patricia Viel](https://www.citterio-viel.com),
shared some interesting advice on managing thousands of families and
on [how to make an interior designer happy with Electron, IFC.js and Revit API](https://www.linkedin.com/pulse/how-make-interior-designer-happy-electron-ifcjs-revit-capasso).
The aim is to speed up the process of comparing and selecting families for large interior design projects.
They require numerous categories of families with hundreds of families in each.
Initial idea: developing a nice web viewer in IFC.js for viewing families instead of buildings.
Now, every interior designer or architect in the office can navigate (way faster than opening the showrooms in Revit) inside all the showrooms using their browser.
That leads to the even better idea...
Thank you, Emiliano, for sharing this great idea and nice write-up.
#### Tree View in Pure CSS
A very nice, clean and lean tree view (collapsible list) can be created using only HTML and CSS, without any need for JavaScript.
Check out the demonstration and detailed step-by-step explanation,
[tree views in CSS](https://iamkate.com/code/tree-views).
#### High-Documentation, Low-Meeting Work Culture
I enjoyed this analysis
of [the perks of a high-documentation, low-meeting work culture](https://www.tremendous.com/blog/the-perks-of-a-high-documentation-low-meeting-work-culture).
It seems highly relevant to our distributed DAS team, Autodesk Developer Advocacy and Support.
It also lines up very well with my personal experience working within our team, and also in my external interactions, both with the diffuse blog- and forum-based Revit API pseudo-community as well as occasionally consulting individually add-in developers with special requirements.
Looking forward to hearing what you think of it.