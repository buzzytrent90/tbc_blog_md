---
post_number: "1640"
title: "Part Atom Direct"
slug: "part_atom_direct"
author: "Jeremy Tammik"
tags: ['family', 'parameters', 'python', 'references', 'revit-api', 'sheets', 'walls', 'windows']
source_file: "1640_part_atom_direct.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1640_part_atom_direct.html"
---

### Standalone BasicFileInfo and ExtractPartAtom
I had an interesting discussion with
Håvard Dagsvik of [Symetri](https://www.symetri.com) on
the use of `TransmissionData` and standalone access to the `BasicFileInfo`, without the need for a valid Revit API context, in the course of which Håvard shared his Revit-independent `GetFamilyXMLData` method implementation, replacing
the Revit API [`Family` `ExtractPartAtom` method](http://www.revitapidocs.com/2018.1/d477cf8f-0dfe-4055-a787-315c84ef5530.htm):
- [No document required for `TransmissionData` access](#2)
- [`TransmissionData` requires a valid Revit API context](#3)
- [Standalone `GetFamilyXmlData` method replacing `ExtractPartAtom`](#4)
- [Windows explorer `BasicFileInfo` right click utility](#5)
#### No Document Required for TransmissionData Access
\*\*Question:\*\* I fear I know the answer to this one already, but I will try to ask you anyway :-)
When reading `TransmissionData` or `BasicFileInfo`, you don't need a document:
![Link Manager](img/hd_link_manager.jpg)
From the `TransmissionData`, I can access the `ExternalFileReference` via `GetLastSavedReferenceData`, etc.
This works fine:
![ExternalFileReference](img/hd_external_file_reference.png)
But, if I implement the same functionality in a standalone executable and use my own TransmissionData function...
![Standalone TransmissionData](img/hd_transmissiondata.png)
It throws this exception on the first method that uses the Revit API:
![Revit API required](img/hd_requires_revit_api.png)
No Revit process involved here, of course.
It's most probably as you have written before:
> In general, as noted in both there and in the summary above, it is not possible to access Revit data or make any use whatsoever of the Revit API from outside of Revit.
> Moreover, you need to be in a valid Revit API context to make any Revit API calls.
Maybe these methods still need the Revit `Application` or `UIApplication` in some way – or they are just locked for some other reason.
Or, should this work and it is just as the message says:
> ... one of the dependencies to RevitAPI.dll is not found?
I'm 99% sure it finds RevitAPI.dll.
After all, this seems like metadata that logically could be available without Revit.
#### TransmissionData Requires a Valid Revit API Context
\*\*Answer:\*\* Yes indeed, you do know the answer: use of the Revit API `TransmissionData` object requires a valid Revit API context.
In other words, you can read data from an unopened Revit document, but you can only do so from within a valid Revit session.
However, there are many ways of getting at the basic file info without use of the Revit API, e.g.,
[determining RVT file version using Python](http://thebuildingcoder.typepad.com/blog/2017/06/determining-rvt-file-version-using-python.html), or
one of the other approaches discussed there.
#### Standalone GetFamilyXmlData Method Replacing ExtractPartAtom
\*\*Response:\*\* That's great; I will definitely try to make use of Victor's raw data approach on `BasicFileInfo`.
Is it thinkable that `TransmissionData` and `BasicFileInfo` doesn't really need Revit?
That they just happen to be packaged within a larger library that overall can`t be used without Revit?
Maybe it is a just licensing policy decision, even though users always have a license.
Meaning, if those methods where refactored out, it could have worked, technically?
If so, let me be the first one who makes the request to get them refactored :-)
For example, take the `Family.ExtractPartAtom` method.
It exports an XML file with all family types, including parameters and values for each type.
But we don't use it.
First, because I (luckily) didn't realize that the Revit API provides this method.
Secondly, because my own custom method is much faster than `ExtractPartAtom`.
More importantly, it can be used without Revit, and it doesn't save to an external XML file that we don't need anyway.
It just returns the raw XML string that is parsed to JSON before it enters a database.
Method here:
```csharp
///
/// Faster ExtractPartAtom reimplementation,
/// independent of Revit API, for standalone
/// external use. By Håvard Dagsvik, Symetri.
/// summary>
/// Family file pathparam>
/// XML datareturns>
static string GetFamilyXmlData(
string family_file_path )
{
byte[] array = File.ReadAllBytes( family_file_path );
string string_file = Encoding.UTF8.GetString( array );
string xml_data = null;
int start = string_file.IndexOf( " );
if( start == -1 )
{
Debug.Print( "XML start not detected: "
+ family_file_path );
}
else
{
int end = string_file.IndexOf( "/entry>" );
if( end == -1 )
{
Debug.Print( "XML end not detected: "
+ family_file_path );
}
else
{
end = end + 7;
int length = end - start;
if( length <= 0 )
{
Debug.Print( "XML length is 0 or less: "
+ family_file_path );
}
else
{
xml_data = string_file.Substring(
start, length );
}
}
}
return xml_data;
}
```
I added Håvard's method to the existing external
command [CmdPartAtom](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdPartAtom.cs)
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) to try this out, in
[release 2018.0.138.4](httpshttps://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.138.4), and cleaned up the existing code calling the built-in Revit API method `ExtractPartAtomFromFamilyFile` at the same time.
#### Windows Explorer BasicFileInfo Right Click Utility
Håvard also used Victor's code as a base to implement his
own [Windows Explorer `BasicFileInfo` right click utility](https://www.screencast.com/t/o78ugUUwY), cf.
the [20-second recording of his RVT BasicFileInfo in Windows Explorer](https://youtu.be/WJCsNywPMVU)
[^](zip/ExplorerBasicFileInfo.mp4):

Works nicely :-)
Not on all Revit files, though.
Workshared files seem to not work, and a few others.
The data is probably there, just have to tweak it a bit more.
Many thanks to Håvard for raising this question and sharing his standalone implementation!