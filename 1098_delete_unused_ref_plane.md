---
post_number: "1098"
title: "Deleting Unnamed Non-Hosting Reference Planes"
slug: "delete_unused_ref_plane"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'references', 'revit-api', 'transactions']
source_file: "1098_delete_unused_ref_plane.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1098_delete_unused_ref_plane.html"
---

### Deleting Unnamed Non-Hosting Reference Planes

Once upon a time, while teaching a Revit API class, the training participants identified a problem that was simple enough to be used as a filtered element collector learning example and did something useful at the same time:
[deleting reference planes not hosting any elements](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-devlab.html#2),
implemented as the stand-alone external command DeleteUnnamedNonHostingReferencePlanes.

It deletes all reference planes that have not been named and are not hosting any elements.

Here is another thread discussing
[filtering reference planes](http://www.revitforum.org/architecture-general-revit-questions/419-filtering-reference-planes.html) and
explaining the use of this.

The first part of the check, for a valid element name, is achieved by using a filtered element collector with an ElementParameterFilter that returns only those elements whose built-in parameter DATUM\_TEXT that returns the reference plane element name has an empty string value.

The second part of the check, whether the reference plane is currently hosting any elements, is automatically executed by the Delete method itself, which fails if that is the case.

A developer now raised an issue with that command:

**Question:** I have been using the code to delete all unnamed reference planes that do not host any elements.

It worked fine for a long time, but now it is throwing an error and shutting down Revit.

Will you please check this.

**Answer:** Thank you for your query.

Basically, the short answer to this question is: no.

The utility command that you refer to was provided on The Building Coder as a learning example to show you how to solve a specific problem yourself.

It is completely unsupported, and any use you make of it is completely at your own risk.

We are more than happy to provide you with developer support for your own applications, but we cannot maintain sample code, nor can we provide any guarantees that the examples presented on the blog will work or be of any productive use at all, except for learning purposes.

Please rephrase the question to request support for solving a specific API related issue that you are encountering in your own application development, and we will be more than happy to help.

I hope you understand the difference.

So, alas, no help for that specific query.

On the other hand, I do wonder for myself what might have changed, and decided to explore the issue anyway.

Meaning that, as an exception, I can change that 'no' to a 'yes' for the nonce.
Lucky you.

To do so, I implemented a new external command named CmdDeleteUnusedRefPlanes in
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples),
migrated the code to the current version and ran a quick test.

Unfortunately, it ended up being not so quick after all, as usual.

The original command uses a filtered element collector to retrieve all unnamed reference planes like this:

```csharp
  // Construct a parameter filter to get only
  // unnamed reference planes, i.e. reference
  // planes whose name equals the empty string:

  BuiltInParameter bip
    = BuiltInParameter.DATUM\_TEXT;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  FilterStringRuleEvaluator evaluator
    = new FilterStringEquals();

  FilterStringRule rule = new FilterStringRule(
    provider, evaluator, "", false );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .OfClass( typeof( ReferencePlane ) )
      .WherePasses( filter );

  int n = 0;
  int nDeleted = 0;

  // No need to cast ... this is pretty nifty,
  // I find ... grab the elements as ReferencePlane
  // instances, since the filter guarantees that
  // only ReferencePlane instances are selected.

  foreach( ReferencePlane rp in col )
  {
    ++n;
    nDeleted += DeleteIfNotHosting( rp ) ? 1 : 0;
  }
```

That part still works fine and remains unchanged.

As you can see, the original command loops through the resulting reference planes and calls the method DeleteIfNotHosting that performs the deletion using the doc.Delete method like this:

```csharp
  /// <summary>
  /// Delete the given reference plane
  /// if it is not hosting anything.
  /// </summary>
  /// <returns>True if the given reference plane
  /// was in fact deleted, else false.</returns>
  bool DeleteIfNotHosting( ReferencePlane rp )
  {
    bool rc = false;

    Document doc = rp.Document;

    Transaction tx = new Transaction( doc );

    tx.Start( "Delete ReferencePlane "
      + ( ++\_i ).ToString() );

    // Deletion simply fails if the reference plane
    // hosts anything. If so, the return value ids
    // is null:

    ICollection<ElementId> ids = doc.Delete( rp );

    if( null == ids || 1 < ids.Count )
    {
      tx.RollBack();
    }
    else
    {
      tx.Commit();
      rc = true;
    }
    return rc;
  }
```

The Delete method overload taking an Element argument is deprecated in Revit 2014, so I replaced it by a call to the overload taking an element id instead:

```csharp
    ICollection<ElementId> ids = doc.Delete( rp.Id );
```

When running the resulting command, the following exception is thrown:

```
Autodesk.Revit.Exceptions.ArgumentException:
  HResult=-2146233088
  Message=ElementId cannot be deleted.
Parameter name: elementId
  Source=RevitAPI
  ParamName=elementId
Stack Trace:
  at Autodesk.Revit.DB.Document.Delete(ElementId)
  at DeleteIfNotHosting(ReferencePlane rp)
  at CmdDeleteUnusedRefPlanes.Execute(...) ...
```

The comment in the original code states that "Deletion simply fails if the reference plane hosts anything. If so, the return value ids collection is null".

This behaviour has been modified in Revit 2014 to throw the exception listed above instead of simply failing.

In other words, this exception is thrown if the reference plane is in fact hosting any elements and therefore should not be removed.

Furthermore, the original command was implemented before I started pointing out that all
[transactions should best be encapsulated within a 'using' statement](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html) block.

Such a block and an exception handler should be added to the call.

Another check that was added in Revit 2014 prevents element deletion while iterating over a collection.

Attempting to delete an element as shown above within the loop iterating over the filtered element collector throws the following exception:

```
Autodesk.Revit.Exceptions.InvalidOperationException:
  HResult=-2146233088
  Message=The iterator cannot proceed due to changes made
  to the Element table in Revit's database (typically,
  This can be the result of an Element deletion).
  Source=RevitAPI
Stack Trace:
  at Autodesk.Revit.DB.FilteredElementIterator.MoveNext()
  at BuildingCoder.CmdDeleteUnusedRefPlanes.Execute() ...
```

One solution to all this is the following simplification.

Instead of using a separate transaction for each individual attempt to delete a reference plane, do them all in one single go.

Furthermore, extract the element ids from the filtered element collector and process them one by one in a separate follow-up loop.

Finally, add an exception handler around each call to the Delete method, since each call to it with a plane hosting elements will cause it to throw an exception.

Here is the complete resulting external command mainline Execute method implementation:

```csharp
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  // Construct a parameter filter to get only
  // unnamed reference planes, i.e. reference
  // planes whose name equals the empty string:

  BuiltInParameter bip
    = BuiltInParameter.DATUM\_TEXT;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  FilterStringRuleEvaluator evaluator
    = new FilterStringEquals();

  FilterStringRule rule = new FilterStringRule(
    provider, evaluator, "", false );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .OfClass( typeof( ReferencePlane ) )
      .WherePasses( filter );

  int n = 0;
  int nDeleted = 0;

  ICollection<ElementId> ids = col.ToElementIds();

  n = ids.Count();

  if( 0 < n )
  {
    using( Transaction tx = new Transaction( doc ) )
    {
      tx.Start( string.Format(
        "Delete {0} ReferencePlane{1}",
        n, Util.PluralSuffix( n ) ) );

      List<ElementId> ids2 = new List<ElementId>(
        ids );

      foreach( ElementId id in ids2 )
      {
        try
        {
          ICollection<ElementId> ids3 = doc.Delete(
            id );

          nDeleted += ids3.Count;
        }
        catch( Autodesk.Revit.Exceptions.ArgumentException )
        {
        }
      }

      tx.Commit();
    }
  }

  Util.InfoMsg( string.Format(
    "{0} unnamed reference plane{1} examined, "
    + "{2} element{3} in total were deleted.",
    n, Util.PluralSuffix( n ),
    nDeleted, Util.PluralSuffix( nDeleted ) ) );

  return Result.Succeeded;
```

Running this in the rac\_basic\_sample\_project.rvt model produces the following result:

![Delete unused reference planes](img/delete_unused_reference_planes_msg.png)

The new command CmdDeleteUnusedRefPlanes is included in
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples),
and the version discussed above is
[release 2014.0.107.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.107.0).