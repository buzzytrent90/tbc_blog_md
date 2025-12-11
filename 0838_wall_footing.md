---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: code_example
optimization_date: '2025-12-11T11:44:14.705536'
original_url: https://thebuildingcoder.typepad.com/blog/0838_wall_footing.html
post_number: 0838
reading_time_minutes: 4
series: general
slug: wall_footing
source_file: 0838_wall_footing.htm
tags:
- elements
- filtering
- levels
- python
- references
- revit-api
- transactions
- walls
title: Wall Footing Relationship Revisited
word_count: 707
---

### Wall Footing Relationship Revisited

In the far distant past, I looked at various ways to programmatically determine
[relationships between associated objects](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html)
(also in [VB](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships-in-vb.html)),
including references between a host object and its hosted dependent elements, e.g. determining the
[relationship between a wall and its footing](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html).

That was back in the year 2009, using Revit 2010.

A few things have changed and been added since then, but the more things change, the more they stay the same, don't they?

Anyway, this very question was raised again:

**Question:** Is there a way to obtain a reference to a wall footing from the wall attached to it?

**Answer:** Yes, sure, there are several different possibilities to achieve this.

You could start off by determining the
[bottom face of the wall](http://thebuildingcoder.typepad.com/blog/2009/08/bottom-face-of-a-wall.html).

Using that, you could set up an ElementIntersectsSolidFilter for a filtered element collector to retrieve all foundation elements touching or intersecting that face, as described for
[retrieving all touching beams](http://thebuildingcoder.typepad.com/blog/2012/09/filter-for-touching-beams-using-solid-intersection.html).
You would obviously want to apply as many quick filters as possible before taking recourse to the slow geometric filtering, e.g. check for level, category, type, etc. first.

Another way to determine adjacency of Revit BIM elements is to make use of the ray casting functionality provided by the FindReferencesByDirection method, FindReferencesWithContextByDirection method and the ReferenceIntersector class, e.g. cast a ray downwards from the wall and detect the first foundation hit by it.

Last but not least, another way to determine relationships between associated objects such as a wall and its footing is to temporarily delete the host object, causing the dependent objects to be deleted as well.
The deleted element ids can be determined, and the whole operation undone to leave the model unchanged.

I described this principle several times, e.g. looking at
[object relationships](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html)
(in [VB](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships-in-vb.html)),
and specifically
[determining the wall footing of a given wall](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html).

I cannot tell off-hand which of these methods is most useful and efficient in your specific case, although I would be very interested in seeing the results of some comparisons and benchmarks evaluating this.

Anyway, I note that The Building Coder sample command CmdWallFooting implementing the latter approach is pretty much out of date, having been simply flat ported across several Revit API releases in the past few years.

I therefore updated it for and tested it with Revit 2013, and can now confirm that it still works.

Here is the updated implementation:
```python
[Transaction( TransactionMode.Manual )]
class CmdWallFooting : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    UIDocument uidoc = app.ActiveUIDocument;
    Document doc = uidoc.Document;

    Wall wall = Util.SelectSingleElementOfType(
      uidoc, typeof( Wall ), "a wall", false )
        as Wall;

    if ( null == wall )
    {
      message
        = "Please select a single wall element.";

      return Result.Failed;
    }

    ICollection<ElementId> delIds = null;

    using( Transaction t = new Transaction( doc ) )
    {
      try
      {
        t.Start( "Temporary Wall Deletion" );

        delIds = doc.Delete( wall );

        t.RollBack();
      }
      catch ( Exception ex )
      {
        message = "Deletion failed: " + ex.Message;
        t.RollBack();
      }
    }

    ContFooting footing = null;

    foreach ( ElementId id in delIds )
    {
      footing = doc.GetElement( id ) as ContFooting;

      if( null != footing )
      {
        break;
      }
    }

    string s = Util.ElementDescription( wall );

    Util.InfoMsg( ( null == footing )
      ? string.Format( "No footing found for {0}.", s )
      : string.Format( "{0} has {1}.", s,
        Util.ElementDescription( footing ) ) );

    return Result.Succeeded;
  }
}
```

I ran this command on a minimal sample model containing just one structural wall with its footing:

![Structural wall and footing](img/wall_footing_2013.png)

The command reports the expected relationship:

![Wall footing relationship](img/wall_footing_message_2013.png)

It says:

```
Wall Walls <164432 Generic - 200mm> has
ContFooting Structural Foundations <164470
Retaining Footing - 300 x 600 x 300>.
```

Here is an updated
[version 2013.0.99.3](zip/bc_13_99_3.zip) of
The Building Coder samples including the updated CmdWallFooting external command.