---
post_number: "0216"
title: "Remove all Geometry from a Family"
slug: "remove_geometry_rfa"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'python', 'references', 'revit-api', 'selection', 'views']
source_file: "0216_remove_geometry_rfa.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0216_remove_geometry_rfa.html"
---

### Remove all Geometry from a Family

Here is a question from Joel Karr of
[Environmental Systems Design, Inc](http://www.esdesign.com)
that nicely complements the issue of
[identifying model elements in a project file](http://thebuildingcoder.typepad.com/blog/2009/05/selecting-model-elements.html)
with the subsequent
[correction](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html)
added.

**Question:**
Is it possible to delete all geometry from a Revit RFA family file?
I would like to keep all parameters but delete all geometry.

**Answer:**
To achieve what you desire, we first need to identify all elements contributing geometry to the model, similarly to the model element selection issues mentioned above.

This is simpler in a family file than in a project file, since there are normally less elements and almost all elements having geometry are the ones we are interested in.

I implemented a sample application RemoveRfaGeom to test this, with the following mainline code for its Execute method:
```python
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  ElementIterator it = doc.Elements;
  Options opt = app.Create.NewGeometryOptions();
  List<Element> a = new List<Element>();

  while( it.MoveNext() )
  {
    Element e = it.Current as Element;

    if( null != e.get\_Geometry( opt ) )
    {
      a.Add( e );
    }
  }

  int n = a.Count;
  Debug.Print(
    "{0} element{1} have non-null geometry{2}",
    n,
    ( 1 == n ? "" : "s" ),
    ( 0 == n ? "." : ":" ) );

  ElementSet els = new ElementSet();

  foreach( Element e in a )
  {
    string cat = (null == e.Category)
      ? "<null>"
      : e.Category.Name;

    Debug.Print(
      "Category={0}; Name={1}; Id={2}; Type={3}",
      cat,
      e.Name,
      e.Id.Value.ToString(),
      e.GetType().Name );

    if( null == e.Category )
    {
      els.Insert( e );
    }
  }

  ElementIdSet ids = doc.Delete( els );

  n = (null == ids) ? 0 : ids.Size;

  Debug.Print( "{0} element{1} deleted.",
    n,
    ( 1 == n ? "" : "s" ) );

  return IExternalCommand.Result.Succeeded;
}
```

It performs the following steps, assuming that the active document is indeed a family document:

- Iterate over all family document elements.- Identify and list all elements with non-null geometry.- Create a set of all elements with geometry whose category is null.- Delete the latter set and display the number of deleted elements.

Running this in the metric library rectangular column family file 'M\_Rectangular Column.rfa' produces the following output:

```
2 elements have non-null geometry:
Category=; Name=Extrusion; Id=105; Type=Extrusion
Category=Cameras; Name=View 1; Id=427; Type=Element
13 elements deleted.
```

As you can see from this, there are some elements having geometry that you presumably do not want to eliminate, such as the camera, so you will need to remove those from the set before deletion.
In this case, I simply noted that the camera has a non-null category, whereas the category of the extrusion I want to remove is null, so I just check the null category property to select the extrusion.

Secondly, deleting an element causes other dependent elements to be deleted also, so even though we just pass in one single extrusion element to the Delete method, a whole group of other elements are eliminated as well.
We already discussed and made use of this effect to determine the
[association between a tag and the tagged element](http://thebuildingcoder.typepad.com/blog/2009/04/tag-association.html)
and between
[a host and its hosted elements](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html).

Here is the complete Visual Studio solution
[RemoveRfaGeom](zip/RemoveRfaGeom.zip)
implementing the new command.