---
post_number: "0923"
title: "Getting Started with the Revit API"
slug: "getting_started"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'revit-api', 'views']
source_file: "0923_getting_started.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0923_getting_started.html"
---

### Getting Started with the Revit API

I am regularly forced to return to this topic, to provide a useful starting point for beginners.

The amount of available material is getting huge, and some help is required deciding where to start exploring the Revit API.

The most important thing to be aware of is the
[Revit Developer Center](http://www.autodesk.com/developrevit).

Start there by looking at the three DevTV video introductions and then work through the
[My First Revit Plug-in](http://www.autodesk.com/myfirstrevitplugin) tutorial.

In these tutorials, you are shown how to locate, install and use the most basic development tools and information, including:

- The Revit SDK or Software Developer Kit, including
  - The Revit API help file RevitAPI.chm
  - RevitLookup, an interactive database element and relationship explorer and massive source code example in itself
  - The SDK sample collection, including
  - The samples themselves, each one equipped with its own read-me file
  - SamplesReadMe.htm: documentation and overview
  - SDKSamples2013.sln: a Visual Studio solution to compile all samples in one go
  - RvtSamples: an external application to load, test and debug all samples
- The Revit API
  [Developer Guide](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00006-API_Developer%27s_Guide)

More detailed overviews of the available Getting Started material are provided for both Revit
[2012](http://thebuildingcoder.typepad.com/blog/2011/10/getting-started-with-the-revit-2012-api.html) (still valid for 2013) and
[2013](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-transparency-support.html#4).
You should also take a look at the list of things to do to
[prepare for a hands-on Revit API training](http://thebuildingcoder.typepad.com/blog/2012/01/preparing-for-a-hands-on-revit-api-training.html).

I suggest performing these steps:

- Watch the videos
- Work through the tutorial
- Install and explore the SDK
- Compile, install and test RevitLookup
- Compile the SDK samples
- Install and test RvtSamples
- Use RevitLookup to explore a Revit database or two
- Browse the documentation and developers guide to your heart's content

Once you have completed these steps, you will be amply set up and ready to go,
[raring](http://www.englishclub.com/ref/esl/Idioms/R/raring_to_go_477.htm) to
get started properly on your own project.

Here is a list of typical questions you might run into:

- [How can I determine the relationship and dependency between elements?](#02)
- [How can I integrate Revit data with an external database?](#03)
- [How can I extract DWFX data from Revit?](#04)
- [How can I modify the DWFX results?](#05)
- [How can I extract a thumbnail image for a specific element?](#06)
- [How can I get a total count of properties, attributes and custom attributes for a given element?](#07)
- [How can I compare the properties of two elements?](#08)
- [When extracting data from Revit, what is the best approach to organize and retrieve that data?](#09)
- [Is there a standard for all families and element types (symbols) that remains the same from model to model?](#10)
- [Is there an easy way to list of all families and element types from a model?](#11)
- [What methods are there to "walk" all objects?](#12)

#### How can I determine the relationship and dependency between elements?

Revit maintains a huge number of different relationships between database elements. How to determine these depends on the elements in question.

An extremely important tool to explore the relationships between elements in the Revit database is
[RevitLookup](http://thebuildingcoder.typepad.com/blog/2010/05/revitlookup-update.html),
included with the Revit SDK.
Use of RevitLookup to explore the Revit database and relationships between elements is discussed and demonstrated in a large number of subsequent posts on The Building Coder blog.

An important technique to discover relationships that are not officially provided through the Revit API is demonstrated by the
[object relationship analyser](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html)
([VB](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships-in-vb.html)).
Use of this technique is also discussed and demonstrated in many places.

Most relationships are accessible through much simpler official means, though.

As said, it depends on what elements you are talking about.

#### How can I integrate Revit data with an external database?

Please look at the following discussions:

- [Database integration](http://thebuildingcoder.typepad.com/blog/2009/01/database-integration.html)
- [ERP systems](http://thebuildingcoder.typepad.com/blog/2009/07/integration-with-a-database-or-erp-system.html)
- [Delux database](http://thebuildingcoder.typepad.com/blog/2013/01/delux-database-enabled-loading-and-updating.html)

#### How can I extract DWFX data from Revit?

Use the Document.Export method taking (String, String, ViewSet, DWFXExportOptions) arguments.

#### How can I modify the DWFX results?

You can export the DWFX.
For any further modifications, you would have to take recourse to the
[DWF Toolkit](http://www.autodesk.mx/adsk/servlet/item?siteID=123112&id=5522878).

#### How can I extract a thumbnail image for a specific element?

Look at the discussions on

- [RVT and RFA thumbnail image](http://thebuildingcoder.typepad.com/blog/2009/06/rvt-and-rfa-thumbnail-image.html)
- [OLE storage](http://thebuildingcoder.typepad.com/blog/2010/06/open-revit-ole-storage.html)
- [BasicFileInfo](http://thebuildingcoder.typepad.com/blog/2013/01/basic-file-info-and-rvt-file-version.html)

#### How can I get a total count of properties, attributes and custom attributes for a given element?

You can easily achieve this, but there is no specific API call provided for this task.
You have to implement it yourself, and the number will vary depending on your requirements, criteria, and implementation.

#### How can I compare the properties of two elements?

[Q] For instance, if two different objects have a property of the same name, can their values be compared or summarized together?
E.g., a property of "Material" for a length of pipe might be copper; will a property of the same name on a copper elbow joint have the same "Material" value?

[A] I am pretty sure that a clever user can create a project in which this will not be the case.

#### When extracting data from Revit, what is the best approach to organize and retrieve that data?

That depends completely on your own needs.
You will need to perform a careful analysis.
Look at the [database integration links](#03) listed above.

#### Is there a standard for all family element types (symbols) that remains the same from model to model?

No, unfortunately, the contrary is true.
Families and types are completely user defined, and creating their definitions is just as much an art as a science.

#### Is there an easy way to list of all families and element types from a model?

Yes.
To list all types, just filter for all ElementType objects:

```csharp
  FilteredElementCollector a
    = new FilteredElementCollector( doc )
      .WhereElementIsElementType();

  foreach( Element e in a )
  {
    Debug.Print( "{0} {1} is an ElementType.",
      e.Name, e.Id.IntegerValue );
  }
```

To retrieve the families that they belong to, you can use the FamilySymbol.Family property.

And: I can promise you some surprises if you start exploring this :-)

#### What methods are there to "walk" all objects?

The filtered element collector is the only way to access in and retrieve elements from the Revit database, unless you have a unique id or an element id and can access the individual object directly.

Please look at the
[RevitLookup](http://thebuildingcoder.typepad.com/blog/2013/03/units-and-revitlookup.html#3) application,
which is provided in source code as part of the Revit SDK.

First of all, it is an invaluable tool both for developers and even end users to interactively explore the database and relationships such as the ones you are asking about.

Secondly, it iterates the entire database and walks all the objects, so you can use that code directly.

#### Conclusion

I hope you find this useful.

In general, almost every single question that you run into in your initial steps of implementing a Revit add-in has already been asked and answered somewhere.
Make good use of the help file, global search of the sample code, developer guide and then turn to an Internet query.

Believe me, it is easy and it is fun... both the development part and the research... it is all creative :-)

Good luck!

#### Project Tracking for Kids

Oh, and one last thing... also fun...
[where tracking is child's play](http://www.atlassian.com/jirajr)...
a few days late for April first :-)