---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: code_example
optimization_date: '2025-12-11T11:44:14.277343'
original_url: https://thebuildingcoder.typepad.com/blog/0619_retrieve_railings.html
post_number: 0619
reading_time_minutes: 3
series: general
slug: retrieve_railings
source_file: 0619_retrieve_railings.htm
tags:
- csharp
- elements
- filtering
- geometry
- levels
- python
- revit-api
- rooms
- selection
- transactions
title: Retrieve Railing Elements
word_count: 693
---

### Retrieve Railing Elements

I already mentioned that API access to stairs and railings is currently still a bit patchy.
Still, you can
[retrieve their geometry](http://thebuildingcoder.typepad.com/blog/2010/02/retrieving-column-and-stair-geometry.html),
[determine the instances on a given level](http://thebuildingcoder.typepad.com/blog/2010/04/retrieve-stairs-on-level.html), and
[list the railing types](http://thebuildingcoder.typepad.com/blog/2009/02/list-railing-types.html).

As you can see from the comments by Berria and Andrew, it is currently
[not possible](http://thebuildingcoder.typepad.com/blog/2009/02/list-railing-types.html?cid=6a00e553e1689788330112790ba9af28a4#comment-6a00e553e1689788330112790ba9af28a4) to
[create railings](http://thebuildingcoder.typepad.com/blog/2009/03/create-room-on-level-in-phase.html?cid=6a00e553e168978833011168f39e48970c#comment-6a00e553e168978833011168f39e48970c),
even though I did once implement a command named
[CmdNewRailing](http://thebuildingcoder.typepad.com/blog/2009/02/list-linked-elements.html) in
The Building Coder samples to test this which unfortunately does **not** create a new railing instance.

Anyway, as discussed with Renzo, we can easily
[retrieve railing instances](http://thebuildingcoder.typepad.com/blog/2010/07/retrieve-structural-elements.html?cid=6a00e553e168978833013489158949970c#comment-6a00e553e168978833013489158949970c).
This question of retrieving railings came up again now, and here is yet another answer and snippet of sample code to address it, this time demonstrating a nice compact concatenation of filtered element collector, generic LINQ, and string methods to create a string listing the element ids of all instances found.
First, here is the question:

**Question:** I implemented the following code to retrieve railing instances, but it does not do what I expect, and colElmts.Count always returns zero:
```csharp
  Dim FilterList As Generic.IList(Of DB.ElementFilter) =
    New Generic.List(Of DB.ElementFilter)
  Dim collector As DB.FilteredElementCollector =
    New DB.FilteredElementCollector(doc)
  Dim Filter As DB.ElementCategoryFilter =
    New DB.ElementCategoryFilter(BuiltInCategory.OST\_Railings)
  Dim LOrF As DB.LogicalOrFilter =
    New DB.LogicalOrFilter(FilterList)
  Dim colElmts As Generic.List(Of DB.Element) =
    collector.WherePasses(LOrF).
    WhereElementIsNotElementType.ToElements()
```

The problem only occurs with railings.
All other categories seem to work fine.
By the way, the current user selection in rvtDoc.Selection does return railing objects if they have been selected in the user interface.

**Answer:** As mentioned above, railings are currently not represented by the Revit API, so there is only limited API access to them.

Still, you should be able to retrieve them using filters in the manner you indicate.

I implemented some sample code based on yours and ran it on the model you provided:

![Railings](img/railings.png)

The initial code does indeed retrieve zero elements.

I thereupon analysed the railings using RevitLookup, and see that their built-in category is not OST\_Railings, which is what you are filtering for, but OST\_StairsRailing. Modifying my code to retrieve that category produces the following:

![Selected railings](img/railings_selected.png)

Here is the modified code producing that message – only four lines of relevant code, albeit two of them rather long and sub-divided:
```python
[Transaction( TransactionMode.ReadOnly )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    string [] ids
      = new FilteredElementCollector( doc )
        .OfCategory(
          BuiltInCategory.OST\_StairsRailing )
        .WhereElementIsNotElementType()
        .Select<Element,string>(
          e => e.Id.IntegerValue.ToString() )
        .ToArray<string>();

    int n = ids.Length;

    string s = (0 == n)
      ? "No railings selected."
      : (n.ToString()
        + " railing"
        + ( 1 == n ? "" : "s" )
        + " selected: "
        + string.Join( ", ", ids )
        + ".");

    TaskDialog.Show( "Railings", s );

    return Result.Succeeded;
  }
}
```

The moral of the story is: use RevitLookup to explore your model. Use the exact built-in category of the objects that you are looking for.

You might of course want to filter for all the different variations of the railing category, in which case the approach described to
[retrieve all MEP](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html) or
[structural elements](http://thebuildingcoder.typepad.com/blog/2010/07/retrieve-structural-elements.html) might
be of interest to you, since it includes handling of multiple categories.

Here is [FilterRailings.zip](zip/FilterRailings.zip) containing the complete Visual Studio solution including source code and add-in manifest file for my little test command.