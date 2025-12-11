---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.647102'
original_url: https://thebuildingcoder.typepad.com/blog/0813_titbits.html
post_number: 0813
reading_time_minutes: 3
series: general
slug: titbits
source_file: 0813_titbits.htm
tags:
- csharp
- elements
- family
- levels
- parameters
- revit-api
- views
title: Titbits of the Week
word_count: 635
---

### Titbits of the Week

Here are a couple of things to round off this warm week (a least here) that I think might be of interest to us all:

- [Identical document GUIDs](#2)- [Journal file path](#3)- [Opening a background document](#4)- [Creating a perspective view](#5)

#### Identical Document GUIDs

Every Revit document now has a GUID, which can be retrieved from the UniqueId property on the ProjectInformation singleton class.

It would be very useful if that could be used to reliably and uniquely distinguish and identify documents.

**Question:** Unfortunately, under certain circumstances, different documents sometimes have the same GUID.

How can that be?

**Answer:** Here are some possible treatments of the documents that could cause this:

- They were 'saved as'.- They were created from the same template.- They were copied as RVT files using the underlying operating system functionality.

This is actually pretty obvious, when you think of it, and as-designed.
The same behaviour is exhibited by any conceivable unique id.

This is not really a Revit API problem, either.
The behaviour would be the same with any other file type.

If you need to identify documents uniquely, one possible thing to do is to add a hidden shared parameter or some extensible storage data to the project info element and store a custom GUID there yourself.
Even in this case, the same problem will occur unless you are in complete control of each and every document to start with.

If you require a document management solution, you can have a look at
[Vault](http://www.autodesk.com/vault) :-)

#### Journal File Path

**Question:** Is there a way in the API to retrieve the path to the journals folder?

This is the default location for Revit Structure:

- C:\Users\USERNAME\AppData\Local\Autodesk\Revit\Autodesk Revit Structure 2013\Journals

Note that this is not the location in the same folder hierarchy as the Revit executable Revit.exe file.

I would like to create a tool that handles deletion of old journal files, to recover the wasted space.

**Answer:** I ended up using this for now:
```csharp
  foreach( RevitProduct product in
    RevitProductUtility
    .GetAllInstalledRevitProducts() )
  {
    string s = null;

    if( product.Version == RevitVersion.Revit2013 )
    {
      string s = product.Product.ToString();

      s = s.Replace( "Revit", "" );

      if( 0 < s.Length )
      {
        s += " ";
      }

      s = string.Format( "{0}\\Autodesk\\Revit\\"
        + "Autodesk Revit {1}2013\\Journals\\",
        Environment.GetFolderPath(
          Environment.SpecialFolder
          .LocalApplicationData ),
        s );
    }
  }
```

You can use standard .NET functionality to check whether a path exists, e.g. File.Exists and Directory.Exists from the System.IO namespace.

#### Opening a Background Document

**Question:** Is it possible to open a 'background' document, also known as a 'side database'?

For example, consider the following scenario:

1. Have a model open, e.g. A.rvt.- Open a second model, e.g. B.rvt, with no views open. As far as the user is concerned, there is no sign of the second model being open.- Read information from B.rvt, and do something interesting with the information in A.rvt.- Maybe even modify B.rvt?

A simple example might do something like determine from B.rvt that it contains an instance of a certain family F.
If F does not exist in A.rvt, it would load it in, essentially copying it from B.rvt, without the user ever seeing or knowing that B.rvt had been opened.

**Answer:** Sure, this can easily be achieved.

Simply use Application.OpenDocumentFile, as opposed to UIApplication.OpenAndActivateDocument.

The former method will open the document at the DB-level without creating a UI document frame, which is just what you want.
You could then call Document.LoadFamily to copy the family.
Just be sure to close the DB document when you are done.

#### Creating a Perspective View

**Question:** Is it possible to programmatically create a **perspective** View3D?

**Answer:** Sure.
Use the static View3D.CreatePerspective method.