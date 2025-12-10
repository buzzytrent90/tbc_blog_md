---
post_number: "0162"
title: "RVT and RFA Thumbnail Image"
slug: "rfa_thumbnail"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'revit-api', 'views', 'windows']
source_file: "0162_rfa_thumbnail.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0162_rfa_thumbnail.html"
---

### RVT and RFA Thumbnail Image

Here is a question that reappears pretty regularly.

**Question:**
How can I obtain the preview bitmap image or thumbnail from a Revit project or family file using the Revit API?

This is an example of the kind of image I am interested in:

![RFA thumbnail preview bitmap image](img/rfa_thumbnail.jpg)

**Answer:**
The topic of
[getting your Revit thumbnails](http://redbolts.com/blog/post/2008/12/01/Getting-your-Revit-thumbnails.aspx)
was already been discussed by
[Guy Robinson](http://redbolts.com/blog)
in
[Bolt out of the Red](http://redbolts.com/blog),
but for completeness sake I will partially reiterate it here as well, since the query keep reappearing.

The Revit API does not provide any built-in support for this, but you can make use of generic Windows API functionality instead.
Revit project files use Windows structured storage to manage resources internally.
You can use the DocFile Viewer utility dfview.exe to look at the structured storage file contents.
Here is an example of a Revit project file with the preview image highlighted:

![RVT file structured storage and preview image in dfview.exe](img/dfview2.png)

We had a short look at the internal RVT file structure when exploring how to extract the Revit build version from
[RVT](http://thebuildingcoder.typepad.com/blog/2008/10/rvt-file-version.html)
and
[RFA](http://thebuildingcoder.typepad.com/blog/2009/06/rfa-version-grey-commands-family-context-and-rdb-link.html#1)
files.

The thumbnail is a standard PNG file inserted into the Revit structured storage document.

To extract the preview thumbnail image, you can use the Windows
[IExtractImage interface](http://msdn.microsoft.com/en-us/library/bb761848(VS.85).aspx).
Preview.dll is a shell plug-in, i.e. an object that implements this interface.
It is used by the Windows Shell Folders to extract preview images for "known" file types.
The preview extractor needs to register itself in the registry and implement the two following standard API functions:

```csharp
STDMETHOD(GetLocation)(
LPWSTR pszPathBuffer,
DWORD cchMax,
DWORD \*pdwPriority,
const SIZE \*prgSize,
DWORD dwRecClrDepth,
DWORD \*pdwFlags);
STDMETHOD(Extract)(HBITMAP\*);
```

More details and sample code are available from
[Guy's post](http://redbolts.com/blog/post/2008/12/01/Getting-your-Revit-thumbnails.aspx).