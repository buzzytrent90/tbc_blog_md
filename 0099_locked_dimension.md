---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.1
content_type: code_example
optimization_date: '2025-12-11T11:44:13.357525'
original_url: https://thebuildingcoder.typepad.com/blog/0099_locked_dimension.html
post_number: 0099
reading_time_minutes: 6
series: general
slug: locked_dimension
source_file: 0099_locked_dimension.htm
tags:
- elements
- filtering
- geometry
- parameters
- python
- references
- revit-api
title: Locked Dimensioning
word_count: 1111
---

### Locked Dimensioning

Here is a little exploration I embarked upon prompted by a
[question](http://thebuildingcoder.typepad.com/blog/2009/02/bolt-out-of-the-red.html#comments)
from
[Pierre-Nelson Navarra](http://forums.augi.com/showthread.php?t=84766).
It starts off as a rather esoteric search for connections between objects that have no API-accessible data, but ends up with a surprising twist, the ability to unlock all dimensions through the API.

**Question:**
Is there a way to determine if a dimension element is locked or not, and is it possible to unlock it through the API?

**Answer:**
To make a short answer long, here is a description of the exploration this prompted me to undertake.

I had a quick look at a dimension element.
I made heavy use of
[RvtMgdDbg](http://download.autodesk.com/media/adn/RvtMgdDbg2009_0429_2008.zip),
the most important tool for examining the Revit database, its elements, their properties and parameters, and the relationships between elements.
Another important tool that I use a lot is the
[Revit API introduction labs](http://thebuildingcoder.typepad.com/blog/files/rac_labs_20081117.zip)
built-in parameter checker.
One thing I noted when examining the dimension element using the latter is the built-in parameter ELEMENT\_LOCKED\_PARAM, which is a read-write Boolean value and initially set to false.
Unfortunately, when I lock the dimension, it still remains false, so this does not give us the information we are looking for.
The 'Revit 2009 API Developer Guide' has nothing to say about this parameter either.

I then examined in detail what elements are added to the database by inserting a dimensioning element and locking it. Adding a dimension introduces one new element into the database:

```
diff RevitElementsBeforeDimension.txt RevitElementsAfterDimension.txt
2259a2260
> Id=130751; Class=Dimension; Category=Dimensions; Name=Linear - 3mm Arial
```

I use the Revit API intro Lab2\_1\_Elements to examine what elements are added, as described in
[exploring element parameters](http://thebuildingcoder.typepad.com/blog/2008/11/exploring-element-parameters.html).

Locking the dimension adds another element:

```
diff RevitElementsAfterDimension.txt RevitElements.txt
2260a2261
> Id=130770; Class=Dimension; Category=Constraints; Name=Linear Dimension Style
```

So the locking information is stored in a separate element.
I thought it might be might be possible to find the connection between these two elements and use that to determine the locked status of the dimension element.
I also hoped that an analysis of the underlying geometry of the two elements would discover that one of them is a lock constraint for the other.

If I remove the lock again, the constraint element id remains in the file but is invalid.
Putting the lock back in again removes the invalid constraint element and adds a new one with a new element id:

```
diff RevitElementsAfterDimension.txt RevitElements.txt
2260a2261
> Id=130813; Class=Dimension; Category=Constraints; Name=Linear Dimension Style
```

The constraint element has almost no accessible data, but it does have a valid class, category and name.
It has a Location property, but we cannot access its internal data either.
As a Dimension instance, it also has a Curve property and a ReferenceArray.
I tried to use these to determine that this constraint matches the existing dimension and is locking it.

Here is the code for the Execute method of an external command that I used to explore this in a model with exactly one locked dimension in it, so that the dimension element 'd' and the constraint 'c' are both well defined:

```python
Application app = commandData.Application;
CreationFilter cf = app.Create.Filter;
Document doc = app.ActiveDocument;

Filter f1 = cf.NewCategoryFilter(
  BuiltInCategory.OST\_Dimensions );

Filter f2 = cf.NewTypeFilter(
  typeof( Dimension ) );

Filter f = cf.NewLogicAndFilter( f1, f2 );

ElementIterator iter = doc.get\_Elements( f );

Dimension d = null;

while( iter.MoveNext() )
{
  d = iter.Current as Dimension;

  Debug.Assert( null != d,
    "expected to find a dimension element" );

  break;
}

f1 = cf.NewCategoryFilter(
  BuiltInCategory.OST\_Constraints );

f = cf.NewLogicAndFilter( f1, f2 );

iter = doc.get\_Elements( f );

Dimension c = null;

while( iter.MoveNext() )
{
  c = iter.Current as Dimension;

  Debug.Assert( null != c,
    "expected to find a constraint element" );

  break;
}

// both locations have no valid information:
Location locc = c.Location;
Location locd = d.Location;

// both lines have no valid information:
Line linc = c.Curve as Line;
Line lind = d.Curve as Line;

// this throws an exception:
//XYZ pc = linc.get\_EndPoint( 0 );
//XYZ qc = linc.get\_EndPoint( 1 );
//XYZ pd = lind.get\_EndPoint( 0 );
//XYZ qd = lind.get\_EndPoint( 1 );

// this cast returns null:
LocationCurve locc2 = c.Location as LocationCurve;
LocationCurve locd2 = d.Location as LocationCurve;

ReferenceArray rc = c.References;
ReferenceArray rd = d.References;

if( rc.Size == rd.Size )
{
  ReferenceArrayIterator ic = rc.ForwardIterator();
  ReferenceArrayIterator id = rd.ForwardIterator();
  while( ic.MoveNext() && id.MoveNext() )
  {
    Reference r1 = ic.Current as Reference;
    Reference r2 = id.Current as Reference;
    if( r1.Equals( r2 ) )
    {
      // this never happens:
      Debug.Print( "Equal" );
    }
  }
}
return CmdResult.Failed;
```

Unfortunately, none of the attempts to read some valid geometrical information return anything useful.
I also explored the parameters of both elements, and found no link there either.
The constraint has no official parameters at all, and the hidden ones provide no clue.

They both have a reference array with two elements each.
The elements in the reference arrays do not compare equal, and trying to explore their innards leads us to an undocumented Pick object which we cannot explore further.

Even if this exploration in Revit 2009 did not lead us anywhere useful, we still discovered some interesting background information on dimensioning and locking in Revit.

And here come two pieces of good news at the end:

First, in the Revit 2010 API, you can simply use the new properties Dimension.IsLocked and DimensionSegment.IsLocked.

#### How to Unlock all Dimensions

Later, Pierre followed up with a useful new result and an additional question:

> I've got the code now to get all constraints.
> When I delete them, the effect is that the dimensions are unlocked.
> That's good.
>
> I'd like to store the original constraints before deleting them in order to relock the dimensions afterwards;
> do you think that is possible?

I am glad that we at least discovered a possibility to unlock the locked dimensions.
Unfortunately, I cannot think of any way to store the constraints prior to deleting them, since we have no access to their internal data.
Maybe they could be copied into a different database beforehand?
But then it might be necessary to copy additional objects as well to preserve the associativity, and I cannot see a way to achieve this.
Or just move them away somewhere where they do no harm instead of deleting them to unlock, and then move them back again to the original position to lock again?