---
post_number: "0171"
title: "View Sketch Plane"
slug: "view_sketch_plane"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'python', 'revit-api', 'sheets', 'views', 'walls', 'windows']
source_file: "0171_view_sketch_plane.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0171_view_sketch_plane.html"
---

### View Sketch Plane

Some views provide a sketch plane, others do not. In particular, the section views do not.
Here is a little exploration into this subject prompted by the following question from Toste Wallmark of
[Tecton Limited](http://www.i-tecton.com):

**Question:**
I'm trying to get the SketchPlane property of currently active view, which is a section view.
However, it returns null.
Do I need to manually create a sketch plane for section views?
Other types of views, like floor plans, do correctly return a SketchPlane object.

**Answer:**
I implemented a little dedicated sample application to test your assertion, and I can reproduce what you say in the sample projects that I have looked at so far.

I would recommend to always check whether the sketch plane returned from a view is null or not.
In case it is null, you will obviously have to create your own.
You can do so using the view origin and direction.
These values are always available from the corresponding properties on the view object, even though the view plane is null.

Here is the result of searching for all document elements that are of the View class or one of its five derived classes.
Looking at a separate listing of all database elements, I noticed that many of the view related objects occur in pairs, like the elements used for level 1 and 2 below with the element id pairs (13073, 13077) and (15915, 15919).
In the other cases, the object type of only one of the two related objects is View, whereas the other is simply a Revit Element and thus does not appear in this list:

![View sketch planes](img/view_sketch_planes.png)

I also print out this list to the Visual Studio debug output window.
The overly long lines can be seen by copying this text to an editor:

```
List of document views' sketch planes:

View [3651] Project View origin (0,0,0) direction (0,-1,0):
ViewPlan [13073] Level 1 origin (0,0,0) direction (0,0,1): Level 1 plane origin (0,0,0), plane normal (0,0,1)
ViewPlan [13077] Level 1 origin (0,0,0) direction (0,0,1): Level 1 plane origin (0,0,0), plane normal (0,0,1)
ViewPlan [15915] Level 2 origin (0,0,0) direction (0,0,1): Level 2 plane origin (0,0,13.12), plane normal (0,0,1)
ViewPlan [15919] Level 2 origin (0,0,0) direction (0,0,1): Level 2 plane origin (0,0,13.12), plane normal (0,0,1)
View [29152] North origin (-1.04,84.1,3.94) direction (0,1,0):
View [29193] East origin (84.12,-2.54,3.94) direction (1,0,0):
View [29214] West origin (-84.13,-2.42,3.94) direction (-1,0,0):
View [29233] South origin (-1.08,-84.12,3.94) direction (0,-1,0):
ViewPlan [29273] Site origin (0,0,0) direction (0,0,1): Level 1 plane origin (0,0,0), plane normal (0,0,1)
View [92030] System Browser origin (0,0,0) direction (0,0,1):
View [138046] Section 1 origin (-20.55,2.78,13.12) direction (-1,0,0):
```

Note that this list includes a view object for the system browser, which obviously has no sketch plane at all.
It still returns valid values for the view origin and direction.

Here are the utility methods used by the external command which generates this list:

```csharp
static string RealString( double a )
{
  return a.ToString( "0.##" );
}

static string PointString( XYZ p )
{
  return string.Format(
    "({0},{1},{2})",
    RealString( p.X ),
    RealString( p.Y ),
    RealString( p.Z ) );
}

static string PlaneString( Plane p )
{
  return string.Format(
    "plane origin {0}, plane normal {1}",
    PointString( p.Origin ),
    PointString( p.Normal ) );
}

static Filter OrType(
  Filter f,
  Type t,
  Autodesk.Revit.Creation.Filter cf )
{
  Filter f2 = cf.NewTypeFilter( t );
  return cf.NewLogicOrFilter( f, f2 );
}
```

Here is the code of the external command Execute method:

```python
Application app = commandData.Application;
Document doc = app.ActiveDocument;
View activeView = doc.ActiveView;

Autodesk.Revit.Creation.Filter cf
  = app.Create.Filter;

Filter f1 = cf.NewTypeFilter( typeof( View ) );
Filter f2 = OrType( f1, typeof( View3D ), cf );
Filter f3 = OrType( f2, typeof( ViewDrafting ), cf );
Filter f4 = OrType( f3, typeof( ViewPlan ), cf );
Filter f5 = OrType( f4, typeof( ViewSection ), cf );
Filter f6 = OrType( f5, typeof( ViewSheet ), cf );

List<RvtElement> views = new List<RvtElement>();
doc.get\_Elements( f6, views );

Debug.Assert( 0 < views.Count,
  "expected document to have at leat one view" );

string msg
  = "List of document views' sketch planes: "
  + Environment.NewLine;

foreach( View view in views )
{
  msg += string.Format(
    "{0}{1} [{2}] {3} origin {4} direction {5}: ",
    Environment.NewLine,
    view.GetType().Name,
    view.Id.Value,
    view.Name,
    PointString( view.Origin ),
    PointString( view.ViewDirection ) );

  SketchPlane sketch = view.SketchPlane;
  msg += ( null == sketch )
    ? "<null sketch>"
    : sketch.Name + " " + PlaneString( sketch.Plane );
}
Debug.Print( msg );
WinForms.MessageBox.Show( msg, "View Sketch Planes" );
return IExternalCommand.Result.Failed;
```

Note that specifying a type filter when calling the document get\_Elements method does not return derived types, only the specific type given.
Since I wish to retrieve all types derived from the View class as well as base class View elements, I need to create a Boolean expression or'ing together all of them.
For this purpose, I invented a nice new little OrType method for succinctly or'ing together a sequence of type filters.