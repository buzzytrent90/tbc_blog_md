---
post_number: "0663"
title: "Set New Pipe Type Properties"
slug: "set_new_pipe_type_prop"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'parameters', 'revit-api', 'transactions', 'walls', 'windows']
source_file: "0663_set_new_pipe_type_prop.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0663_set_new_pipe_type_prop.html"
---

### Set New Pipe Type Properties

Here is an interesting Revit MEP API question on setting the preferred elbow, tee and junction type information after creating a new Revit MEP pipe type, answered by my colleague Joe Ye in Beijing:

**Question:** I am creating a new Revit MEP pipe type by filtering for all element types of the category OST\_PipeCurves and selecting the one named "Standard".
On the resulting new ElementType instance, I would like to define the preferred settings as follows:

- Change the preferred elbow from 'Elbow - Generic: Standard ' to 'Elbow - Thread - MI - Class 150: Standard'.- Change the preferred tee from 'Tee - Generic: Standard' to 'Tee - Thread - MI - Class 150: Standard'.- Change the preferred junction type from 'Tee' to 'Tap'.

Can you show me in C# how to achieve this, please?

**Answer:** These three changes can be achieved by changing the values of some properties on the PipeType element:

- Elbow: the default elbow fitting of the MEP curve type.- Tee: the default tee fitting of the MEP curve type.- PreferredJunctionType: the preferred junction type of the MEP curve type.

The values stored in the first two properties are family symbols.
You need to find the desired target element in the Revit database and assign it to them.

For example, for the elbow, you can filter out the desired FamilySymbol 'Elbow - Thread - MI - Class 150: Standard'.
This element's class is 'FamilySymbol' and its built-in category is 'OST\_PipeFitting'.
Use these two filters and the traverse the returned element collection to find the target FamilySymbol by name comparison, and assign it to the Elbow property.

Instead of using the PipeType Elbow property, you can also use the generic Revit parameter access and the appropriate built-in parameter.
This can be determined by exploring an existing pipe type using the RevitLookup database snoop tool.
In that case, you can assign the value by calling the Parameter.Set(ElementId) method and passing in the family symbol element id.

Setting the desired tee is similar to the elbow.
In this case, you can filter out the target FamilySymbol 'Tee - Thread - MI - Class 150: Standard' and assign it to the Tee property, or its element id to the equivalent parameter.

For the preferred Junction Type, you specify your choice using an integer value to indicate one of the two available options.
0 indicates Tap, and 1 indicates Tee.

So in your case, you would assign the integer value 0 to the 'Preferred Junction Type' property or parameter.

The complete code making use of the generic parameter access might look like this:
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Plumbing;
using Autodesk.Revit.UI;

[TransactionAttribute( TransactionMode.Manual )]
public class RevitCommand : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document rvtDoc = app.ActiveUIDocument.Document;

    FilteredElementCollector collector
      = new FilteredElementCollector( rvtDoc );

    collector.OfCategory( BuiltInCategory.OST\_PipeCurves );
    collector.OfClass( typeof( ElementType ) );

    PipeType pipeType = null;
    foreach( Element ptype in collector )
    {
      if( string.Compare( ptype.Name, "standard", true ) == 0 )
      {
        pipeType = ptype as PipeType;
        break;
      }
    }

    if( pipeType != null )
    {
      string name = "ADNNewType";

      Transaction trans = new Transaction( rvtDoc )
      trans.Start(  "Set Pipe Type Information" );

      ElementType newType = pipeType.Duplicate( name );

      if( newType != null )
      {
        ElementId id1 = FindFamilyType( rvtDoc,
          typeof( FamilySymbol ),
          "Elbow - Threaded - MI - Class 150", "Standard",
          BuiltInCategory.OST\_PipeFitting ).Id;

        ElementId id2 = FindFamilyType( rvtDoc,
          typeof( FamilySymbol ),
          "Tee - Threaded - MI - Class 150", "Standard",
          BuiltInCategory.OST\_PipeFitting ).Id;

        newType.get\_Parameter(
          BuiltInParameter.RBS\_CURVETYPE\_DEFAULT\_ELBOW\_PARAM )
            .Set( id1 );

        newType.get\_Parameter(
          BuiltInParameter.RBS\_CURVETYPE\_DEFAULT\_TEE\_PARAM )
            .Set( id2 );

        newType.get\_Parameter(
          BuiltInParameter.RBS\_CURVETYPE\_PREFERRED\_BRANCH\_PARAM )
            .Set( 0 );
      }
      trans.Commit();
    }
    return Result.Succeeded;
  }

  /// <summary>
  /// Find an element of the given type, name,
  /// and category (optional).
  /// You can use this, for example, to find a
  /// specific wall and window family with the
  /// given name, e.g.:
  /// FindFamilyType( \_doc, GetType(WallType),
  ///   "Basic Wall", "Generic - 200mm" )
  /// FindFamilyType( \_doc, GetType(FamilySymbol),
  ///   "M\_Single-Flush", "0915 x 2134mm",
  ///   BuiltInCategory.OST\_Doors )
  /// </summary>
  public static Element FindFamilyType(
    Document rvtDoc,
    Type targetType,
    string targetFamilyName,
    string targetTypeName,
    Nullable<BuiltInCategory> targetCategory )
  {
    var collector
      = new FilteredElementCollector( rvtDoc )
        .OfClass( targetType );

    if( targetCategory.HasValue )
    {
      collector.OfCategory( targetCategory.Value );
    }

    // Parse the collection for the given names
    // using LINQ

    var targetElems =
        from element in collector
        where element.Name.Equals( targetTypeName )
          && element.get\_Parameter(
            BuiltInParameter.SYMBOL\_FAMILY\_NAME\_PARAM )
              .AsString().Equals( targetFamilyName )
        select element;

    // Put the result as a list of element fo accessibility.

    IList<Element> elems = targetElems.ToList();

    // Return the result.

    if( elems.Count > 0 )
    {
      return elems[0];
    }
    return null;
  }
}
```

By the way, these preferred element options can also be set to None.
This is achieved just like we described for
[setting the underlay display property](http://thebuildingcoder.typepad.com/blog/2011/08/set-underlay-display-property-to-none.html),
by using the parameter setting approach and providing ElementId.InvalidElementId as a value, e.g.
```csharp
    newType.get\_Parameter(
      BuiltInParameter.RBS\_CURVETYPE\_DEFAULT\_ELBOW\_PARAM )
        .Set( ElementId.InvalidElementId );
```

Many thanks to Joe for this solution!