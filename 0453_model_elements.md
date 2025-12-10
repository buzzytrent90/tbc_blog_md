---
post_number: "0453"
title: "Selecting Model Elements"
slug: "model_elements"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'parameters', 'python', 'revit-api', 'selection', 'transactions', 'views', 'walls', 'windows']
source_file: "0453_model_elements.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0453_model_elements.html"
---

### Selecting Model Elements

We repeatedly looked at the
[selection of model elements](http://thebuildingcoder.typepad.com/blog/2009/11/select-model-elements-2.html),
and in the discussion on
[material quantity extraction](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html) I
mentioned the idea that one possible additional improvement might be to make use of the Category.HasMaterialQuantities property.

The topic keeps returning, and now it arose again in a question from Konstanty Seniut.
I have updated the answer after the initial posting to add some
[later notes](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html) by
Scott Conover, so that they are not missed.
To see Scott's notes and explanations in full, please refer to the
[follow-up post](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html).

**Question:** I would like to retrieve all elements from the model, for example floors, walls, windows, lines, but ignoring all views, levels and etc.
For now my solution is simply to make list of all the categories of the things that I don't want to retrieve.
Is there any easier way to achieve this?

Currently, the following approach is working for me:
```python
Dictionary<string, Category> GetAllCategories(
  List<Document> allDocuments )
{
  Dictionary<string, Category> categories
    = new Dictionary<string, Category>();

  Dictionary<int, Element> elemnts
    = new Dictionary<int, Element>();

  foreach( Document doc in allDocuments )
  {
    FilteredElementCollector collector
      = new FilteredElementCollector( doc );

    IList<Autodesk.Revit.DB.Element> found
      = collector
        .WhereElementIsNotElementType()
        .WhereElementIsViewIndependent()
        .WherePasses( new LogicalOrFilter(
          new ElementIsElementTypeFilter( false ),
          new ElementIsElementTypeFilter( true ) ) )
        .ToElements();

    var disElems = (from elem in found select elem)
      .Distinct();

    foreach( Element element in disElems )
    {
      if( element.Category != null )
      {
        if( element.Parameters.Size > 0 )
        {
          if( element.PhaseCreated != null )
          {
            if( !categories.ContainsKey(
              element.Category.Name ) )
            {
              categories.Add(
                element.Category.Name,
                element.Category );

              elemnts.Add(
                element.Category.Id.IntegerValue,
                element );
            }
          }
          else if( element.Location != null )
          {
            LocationPoint point = null;
            LocationCurve curve = null;
            try
            {
              point = element.Location
                as LocationPoint;
            }
            catch { }

            try
            {
              curve = element.Location
                as LocationCurve;
            }
            catch { }

            if( curve != null || point != null )
            {
              if( !categories.ContainsKey(
                element.Category.Name ) )
              {
                categories.Add(
                  element.Category.Name,
                  element.Category );

                elemnts.Add(
                  element.Category.Id.IntegerValue,
                  element );
              }
            }
          }
        }
      }
    }
  }
  return categories;
}
```

Is this a good solution?
What do you think?

**Answer:** As said, I presented one approach in the
[selection of model elements](http://thebuildingcoder.typepad.com/blog/2009/11/select-model-elements-2.html).
Scott
[later pointed out](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html)
that that solution appends filters to each other one at a time.
The new Revit 2011 LogicalOrFilters support more than two inputs, so it should actually be updated to make more efficient use of 2011 filtering.

The other approach I mentioned, based on the Category.HasMaterialQuantities property, allows a much shorter implementation like this:
```python
[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
public class Lab2\_2\_ModelElements : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    FilteredElementCollector collector
      = new FilteredElementCollector( doc )
        .WhereElementIsNotElementType();

    List<string> a = new List<string>();

    foreach( Element e in collector )
    {
      //  && null != e.Materials
      //  && 0 < e.Materials.Size

      if( null != e.Category
        && e.Category.HasMaterialQuantities )
      {
        a.Add( string.Format(
          "Category={0}; Name={1}; Id={2}",
          e.Category.Name, e.Name,
          e.Id.IntegerValue ) );
      }
    }

    LabUtils.InfoMsg(
      "Project contains {0} model element{1}{2}", a );

    return Result.Failed;
  }
}
```

Note that this approach may not retrieve all desired elements, as Scott explains in his
[update notes](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html).

Which approach to use, or how to combine them, depends on your exact needs, obviously.

Some notes on your implementation:

- I find the check for parameters interesting, and also checking the PhaseCreated and Location properties.- I would recommend you never to use an
    [exception handler](http://en.wikipedia.org/wiki/Exception_handling) to
    check for a valid condition, if you can avoid it.
    [Exceptions should always be exceptional](http://www.jacopretorius.net/2009/10/exceptions-should-be-exceptional.html),
    for handling unexpected errors.- Scott
      [later pointed out](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html)
      that the use of the WherePasses clause is totally unnecessary and should be removed.

**Response:** The additional check for Category.HasMaterialQuantities suits me well for now.
Thank you very much!

The first approach you suggested in the blog is very good; in the beginning of solving this problem I had a similar idea to make list of redundant categories, but I wanted a simpler approach.

My final solution for now is:
```csharp
private void GetAllCategories(
  List<Document> allDocuments )
{
  Dictionary<string, Category> categories
    = new Dictionary<string, Category>();

  List<Element> elements = new List<Element>();

  foreach( Document doc in allDocuments )
  {
    FilteredElementCollector collector
      = new FilteredElementCollector( doc );

    collector
      .WhereElementIsNotElementType()
      .WhereElementIsViewIndependent()
      .ToElements();

    foreach( Element element in collector )
    {
      if( null != element.Category
        && 0 < element.Parameters.Size
        && (element.Category.HasMaterialQuantities
          || null != element.PhaseCreated) )
      {
        if( !categories.ContainsKey(
          element.Category.Name ) )
        {
          categories.Add(
            element.Category.Name,
            element.Category );
        }
        elements.Add( element );
      }
    }
  }
}
```

If in future there will be any need to retrieve some specific categories, I can use a list of specific categories in addition to this approach.

For now I do not know how to eliminate the try catch when needing to check the Location property, because I had situations when an element had a Location, but trying to access it threw an exception.
At the moment I can avoid checking that property completely, though.

I think it is good idea to share :) maybe someone like me will have a similar problem also :)

**Answer:** The call to ToElements in this code is completely redundant, as Scott explains in his
[update notes](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html),
so it should simply be removed.

**Response:** Interesting things. I will try to improve them.
The HasMaterialQuantities property is not set in every element I am interested in, just as Scott said.
This is why I used the Location property to populate the dictionary with all specific elements :)