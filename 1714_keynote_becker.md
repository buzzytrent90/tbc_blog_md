---
post_number: "1714"
title: "Keynote Becker"
slug: "keynote_becker"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'revit-api', 'sheets', 'views', 'walls']
source_file: "1714_keynote_becker.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1714_keynote_becker.html"
---

### pyRevit Keynote Manager and Becker Forge Blog
Today, I discuss a simple Revit API getting started question from StackOverflow, a new Forge blog and a request for feedback on the keynote manager beta:
- [Retrieving a wall type for creating a wall](#2)
- [CAD Becker Forge blog](#3)
- [pyRevit keynote manager beta](#4)
#### Retrieving a Wall Type for Creating a Wall
Here is a quick 'getting started' Q and A from the StackOverflow question
on the [C# Revit API and how to create a simple wall using `ExternalCommand`](https://stackoverflow.com/questions/53921260/c-sharp-revit-api-how-to-create-a-simple-wall-using-externalcommand):
\*\*Question:\*\* I just wanted to create a simple wall using `ExternalCommand`, but I cannot figure it out...
I think my problem is here:
```csharp
var symbolId = document.GetDefaultFamilyTypeId(
new ElementId( BuiltInCategory.OST_Walls ) );
```
When I debug it, `symbolId` always equals `-1`.
\*\*Answer:\*\* Work through
the [Revit API getting started material](https://thebuildingcoder.typepad.com/blog/about-the-author.html#2) and
all will be explained.
That will save you and others many further questions and answers.
To address this specific question anyway, `GetDefaultFamilyTypeId` presumably does not do what you expect it to for wall elements.
In the [`GetDefaultFamilyTypeId` method API documentation](https://apidocs.co/apps/revit/2019/34d20683-dfea-b1f8-14cf-750611b218ed.htm),
it is used for structural columns, a standard loadable family hosted by individual RFA files.
Walls are built-in system families and behave differently.
Maybe `GetDefaultFamilyTypeId` only works for non-system families.
The `-1` value represent an undefined element id, equal to
the [constant `ElementId.InvalidElementId` property](https://apidocs.co/apps/revit/2019/08ae8886-6ab3-3ef5-d2e0-0da2ffa7bd2c.htm).
To retrieve an arbitrary (not default) wall type, use a filtered element collector to retrieve all `WallType` elements and pick the first one you find.
Here is a code snippet that retrieves all wall types with a filtered element collector and applies .NET LINQ post-processing to that to pick the first one with a specific name,
from [The Building Coder](https://thebuildingcoder.typepad.com) discussion
on [creating face wall and mass floor
](https://thebuildingcoder.typepad.com/blog/2017/12/creating-face-wall-and-mass-floor.html):
```csharp
WallType wType = new FilteredElementCollector( doc )
.OfClass( typeof( WallType ) )
.Cast().FirstOrDefault( q
=> q.Name == "Generic - 6\" Masonry" );
```
When you start getting deeper into the use of filtered element collectors, be aware of the significant performance difference
between [quick and slow filters versus .NET post-processing](http://thebuildingcoder.typepad.com/blog/2015/12/quick-slow-and-linq-element-filtering.html).
\*\*Response:\*\* I solved the problem and created many different types of walls already after reading
the [Revit online help](http://help.autodesk.com/view/RVT/2019/ENU) >
Revit Developer's Guide
> [Revit API Developers Guide](http://help.autodesk.com/view/RVT/2019/ENU/?guid=Revit_API_Revit_API_Developers_Guide_html).
Your getting started link is also helpful information for newbies.
#### CAD Becker Forge Blog
A quick welcome to another member of the Forge blogosphere:
Jürgen [@CADBeckerde](https://twitter.com/CADBeckerde) Becker of [CAD-Becker.de](https://www.cad-becker.de) published
two German-language blog posts on 3-legged Forge authentication:
- [Autodesk Forge – 3-legged Authentifizierung](https://cad-becker.de/index.php/de/blog/autodesk-forge-3-legged-authentifizierung)
- [Autodesk Forge – 3-legged Authentifizierung die Zweite](https://cad-becker.de/index.php/de/blog/blog-development/autodesk-forge-3-legged-authentifizierung-die-zweite)
Thank you very much, Jürgen, for sharing this information for our German-speaking friends!
It is also useful for any non-German-speaker willing to run a translation tool on it.
Keep up the good work :-)
#### pyRevit Keynote Manager Beta
Ehsan [@eirannejad](https://twitter.com/eirannejad) Iran-Nejad
requests input on his pyRevit keynote manager prototype, included with
the [pyRevit toolset](https://eirannejad.github.io/pyRevit):
> Who has used the new keynote manager successfully? Does it work fine? Is it intuitive?
> Docs are work in progress. Keynote change history with full diff view is next. Open to suggestions...
![pyRevit keynote manager change history](img/pyrevit_keynote_manager_diff.png)
For more information, please look at the 56-minute presentation
of [pyRevit 4.6.9 and Keynote Manager Beta](https://youtu.be/1s6JZOBVAHs):

Thank you very much, Ehsan, for your invaluable work on pyRevit and the new useful keynote manager!