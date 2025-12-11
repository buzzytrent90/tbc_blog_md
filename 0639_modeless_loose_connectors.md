---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.6
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.311090'
original_url: https://thebuildingcoder.typepad.com/blog/0639_modeless_loose_connectors.html
post_number: 0639
reading_time_minutes: 6
series: mep
slug: modeless_loose_connectors
source_file: 0639_modeless_loose_connectors.htm
tags:
- csharp
- elements
- family
- filtering
- revit-api
- mep
title: Modeless Loose Connector Navigator Update
word_count: 1120
---

### Modeless Loose Connector Navigator Update

Here is an update to my
[modeless loose connector navigator](http://thebuildingcoder.typepad.com/blog/2010/07/modeless-loose-connectors.html),
which I created as a sample application to demonstrate how to set up a modeless dialogue to drive Revit using the
Idling event.
Please refer to the original post for a full description.

The original Revit 2011 version had one flaw: the external command it implements subscribes to the Idling event.
That means that a new subscription is added for every call to the command.
Also, the subscription was never revoked.

That prompted me to present an updated version of the application for Revit 2012, which brings up a number of little items of interest:

- [Adding an external application and loading in MEP only](#1)- [Idling sender type changed](#2)- [Multiplatform support](#3)- [ElementMulticlassFilter](#4)- [Logical connector check](#5)

#### Adding an External Application and Loading in MEP Only

The safest way to subscribe to application events is to implement an external application, and subscribe and unsubscribe to them from the OnStartup and OnShutdown event handlers.

The external command already makes use of the
[VisibilityMode tag](http://thebuildingcoder.typepad.com/blog/2010/05/addin-visibility-mode.html) to
appear only in the MEP product, but that tag is not supported for external applications.

Therefore, I implemented a check in the OnStartup method to refuse to load the application unless it is being loaded into Revit MEP, as described in
[add-in applications for multiple Revit products](http://thebuildingcoder.typepad.com/blog/2010/06/addin-applications-for-multiple-revit-products.html):
```csharp
public Result OnStartup( UIControlledApplication a )
{
  ProductType pt = a.ControlledApplication.Product;

  if( ProductType.MEP == pt )
  {
    a.Idling += new EventHandler<IdlingEventArgs>(
      OnIdling );

    return Result.Succeeded;
  }
  else
  {
    return Result.Cancelled;
  }
}
```
On successful loading, we subscribe to the Idling event.
The application OnIdling method simply forwards the notification to the existing handler in the external command, which is now a static method:
```csharp
void OnIdling( object sender, IdlingEventArgs e )
{
  Command.OnIdling( sender, e );
}
```

All the rest remains unchanged.

Oh no, not true at all.

#### Idling Sender Type Changed

One thing that changed between the Revit 2011 and Revit 2012 API is the
[type of the Idling event sender argument](http://thebuildingcoder.typepad.com/blog/2011/05/cascaded-events-and-renaming-the-active-document.html).
It used to be an Application instance, and was changed in 2012 to be a UIApplication instance instead.
Here is a snippet showing the simplest and most foolproof method for handling this that I could come up with, which eliminates any need to actually check which version of Revit your add-in is running in:
```csharp
  // Support both 2011, where sender is an
  // Application instance, and 2012, where
  // it is a UIApplication instance:

  UIApplication uiapp
    = sender is UIApplication
      ? sender as UIApplication
      : new UIApplication(
        sender as Application );
```

If this code is executed by Revit 2011, the sender will be an Application, and we instantiate our own UIApplication instance from it. Otherwise, we can use the UIApplication instance provided by the sender argument directly.

#### Multiplatform Support

Note that the change above refers to **running** the add-in in Revit 2012.
However, it does not mean that I have to **compile** it specifically for Revit 2012.
I am still referencing the Revit 2011 API DLLs, which means that I can run it in both Revit 2011 and 2012.

I did however migrate the codebase from Visual Studio 2008 to 2010.
I created backup files of the entire loose connector navigator source code just before and after this migration, so I actually have three different versions of it to share with you here which are compiled using the Revit 2011 API and run on both the Revit 2011 and 2012 platforms, in case you are interested in comparing the gradual evolution:

- [loose\_connectors\_7\_2008.zip](zip/loose_connectors_7_2008.zip) –
  last Visual Studio 2008 version.- [loose\_connectors\_7\_2010.zip](zip/loose_connectors_7_2010.zip) –
    first Visual Studio 2010 version.- [loose\_connectors\_8.zip](zip/loose_connectors_8.zip) –
      updated version using an external application to subscribe to the Idling event.

#### ElementMulticlassFilter

Actually, there is one Revit 2012 API feature that one might potentially make use of, although it makes very little difference:

The ElementMulticlassFilter allows us to check for multiple types of elements using one single filtered element collector.
We can use this in the GetConnectorElements method for
[retrieving all elements which may have some kind of MEP connector](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html).
It selects all elements which are family instances belonging to one of a given list of categories, which are retrieved by a filter named 'familyInstanceFilter', OR are one of a given list of types, which is defined like this in the Revit 2011 code:
```csharp
  IList<ElementFilter> b
    = new List<ElementFilter>( 6 );

  b.Add( new ElementClassFilter( typeof( CableTray ) ) );
  b.Add( new ElementClassFilter( typeof( Conduit ) ) );
  b.Add( new ElementClassFilter( typeof( Duct ) ) );
  b.Add( new ElementClassFilter( typeof( Pipe ) ) );

  if( include\_wires )
  {
    b.Add( new ElementClassFilter( typeof( Wire ) ) );
  }
  b.Add( familyInstanceFilter );

  LogicalOrFilter classFilter
    = new LogicalOrFilter( b );
```

Making use of the ElementMulticlassFilter enables us to simplify this a little bit:
```csharp
  List<Type> types = new List<Type>( 5 );

  types.Add( typeof( CableTray ) );
  types.Add( typeof( Conduit ) );
  types.Add( typeof( Duct ) );
  types.Add( typeof( Pipe ) );

  if( include\_wires )
  {
    types.Add( typeof( Wire ) );
  }

  List<ElementFilter> b
    = new List<ElementFilter>( 2 );

  b.Add( new ElementMulticlassFilter( types ) );

  b.Add( familyInstanceFilter );

  LogicalOrFilter classFilter
    = new LogicalOrFilter( b );
```

Well, maybe not simplify, exactly, but make use of a smaller total number of filters, by using one single ElementMulticlassFilter instead of five separate ElementClassFilter instances OR'ed together.
Since we are OR'ing them with the familyInstanceFilter in any case, it makes little difference, and actually increases the number of lines of code instead of decreasing them.
Therefore, I will revert this change in my code to retain the compatibility with Revit 2011.
Introducing this change would be the first and single stumbling block causing the application not to work on Revit 2011 any longer.

#### Logical Connector Check

By the way, here is another minute change introduced by the Revit 2012 API that I also discovered while switching back and forth between the two versions:

In Revit 2011, I skipped unconnected logical connectors using the following code:
```csharp
  ConnectorType.LogicalConn != c.ConnectorType // 2011
```

The ConnectorType enumeration value has been renamed in the Revit 2012 API, and the check is now
```csharp
  ConnectorType.Logical != c.ConnectorType // 2012
```

It really is pretty impressive how the .NET API shields us so well from the underlying platform, that we can use the old or new API interchangeably to run on the new platform, if we are able to simply avoid using any functionality that changed significantly.