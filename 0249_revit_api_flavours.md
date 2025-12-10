---
post_number: "0249"
title: "Revit API Flavours"
slug: "revit_api_flavours"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'revit-api', 'rooms', 'views']
source_file: "0249_revit_api_flavours.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0249_revit_api_flavours.html"
---

### Revit API Flavours

Maher raised an interesting
[question](http://thebuildingcoder.typepad.com/blog/2009/09/the-revit-mep-api.html#comment-6a00e553e1689788330120a6a0e650970b) regarding the different flavours of the Revit API:

**Question:** I have a few questions on Revit MEP, Arch, and Structure in general.
What are the major highlights of each API?
I downloaded the SDK and tried to read through.
But could not get my head around it as it is very overwhelming.
It would be great if you could post something in the difference between these three flavours both in terms of API and in general use.
They seem to do the same thing in many cases.
I am confused.

**Answer:** By far the major part of the Revit API is identical for the three flavours, and to get started with it you need not worry about the differences. All the basic functionality and most objects that I have ever had occasion to use are provided in all three flavours.

If you try to access a property that is not available in the particular flavour of Revit that your application is running in, the property will generally simply return a null value or sometimes throw an exception, as Shifali is currently painfully discovering, being
[unable to access the gbXML settings in Revit Architecture](http://thebuildingcoder.typepad.com/blog/2009/04/new-2010-events.html#comments).

You can check what flavour you are running in by querying the application object as demonstrated by the
[Revit API introduction labs](http://thebuildingcoder.typepad.com/files/rac_labs_2009-07-30.zip) sample command Lab1\_2\_CommandArguments:
```csharp
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;
  View view = commandData.View;
  string s = "Application = " + app.VersionName;
  s += "\r\nVersion = " + app.VersionNumber;
  s += "\r\nDocument path = " + doc.PathName;
  s += "\r\nDocument title = " + doc.Title;
  s += "\r\nView name = " + view.Name;
  LabUtils.InfoMsg( s );
```

In the Revit 2010 API, new LanguageType and ProductType enumerations and corresponding properties on the application object were introduced to enable an add-in to query and analyse the underlying product and user interface language without resorting to any string comparisons:
```csharp
  LanguageType lt = app.Language;
  ProductType pt = app.Product;
```

We have previously taken a deeper look at the Revit
[Structure](http://thebuildingcoder.typepad.com/blog/2009/04/revit-structure-resources.html) and
[MEP](http://thebuildingcoder.typepad.com/blog/2009/09/the-revit-mep-api.html) APIs and mentioned some of the specific features of each.

The easiest way that I can think of to see the exact differences between the three flavours of the Revit API is to look at the class diagram in the Revit SDK provided in "Revit API Class Diagram.png".
It shows all of the namespaces and classes defined in the Revit API and colour codes them into various groups, including separate groups for the three product flavours.

The developer guide in the document "Revit 2010 API Developer Guide.pdf" also includes the three separate chapters dealing with the API specifics of each flavour:

23. **Revit Architecture**

    23.1   Rooms

    23.2   Energy Data- **Revit Structure**

      24.1   Structural Model Elements

      24.2   AnalyticalModel

      24.3   Loads

      24.4   Analysis Link- **Revit MEP**

        25.1   MEP Element Creation

        25.2   Connectors

        25.3   Family Creation