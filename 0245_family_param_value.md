---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.614046'
original_url: https://thebuildingcoder.typepad.com/blog/0245_family_param_value.html
post_number: '0245'
reading_time_minutes: 6
series: parameters
slug: family_param_value
source_file: 0245_family_param_value.htm
tags:
- csharp
- elements
- family
- geometry
- levels
- parameters
- revit-api
title: Family Parameter Value
word_count: 1128
---

### Family Parameter Value

Pramod asked an interesting
[question](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html#comment-6a00e553e1689788330120a6a4cd6f970c) on
obtaining the values of family parameters in a family document which led me to explore this topic and discover a little more complexity than I was originally expecting:

**Question:** We are using the API of Revit Architecture 2010.
We want to get all Parameters and their values from the Family document.
We are referring to the AutoParameter example from the Revit SDK.
With the help of this example, we are able to get parameters of Family, but didn't get any property or method for getting Value of those parameters.
So please give any code example for getting values of the Family Parameters.

**Answer:** I dug around a bit to provide an answer for you, and was surprised at the result.
I had not previously noticed the additional level of indirection required to access a family parameter value.

Before we start working on reading family parameter values, let me mention that the addParameters method in the
[Revit Family API labs](http://thebuildingcoder.typepad.com/blog/2009/10/revit-family-creation-api-labs.html)
provides a sample demonstrating how to create family parameters and set their values.

The family has two main aspects, the geometry and the parameter data. Some of the parameters can be used to drive the geometry, so the two areas are interlinked. The data is stored in parameters, and the family contains a set of predefined types which already have predefined values for certain parameters. The family parameter has a pointer to its definition which specifies the name and data type, just like a project parameter. However, it does not have a parameter value, because the values are specific to the family types. Instead, each family type has its own value for each of the family parameters.

One of the main objects in the family document is its family manager, which manages its family types and parameters. A family parameter looks a lot like a normal Revit parameter on a BIM element instance, and has many of the same properties such as the Definition property which defines the parameter name and data type. The big difference between a family and a non-family parameter is that the latter has a value and the former does not.

In a project document, a parameter has a pointer to the parameter definition which defines its name and data type and other properties, and the main data item of the individual parameter is its own personal parameter value. A family parameter has one additional level of complexity and indirection, since the family parameter values are defined individually for each of the family types.

When you retrieve a value for a parameter from a family type, you have to supply the family parameter instance to specify which parameter value to retrieve.

I created a new Building Coder sample command named CmdFamilyParamValue to demonstrate how to read the values of family parameters.

First of all, it ensures that we are running in a family document:
```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;
if( !doc.IsFamilyDocument )
{
  message =
    "Please run this command in a family document.";
}
else
```

In order to retrieve a parameter value from a family type, we need to provide a pointer to the family parameter instances that we are interested in, so we start off by creating a dictionary containing all the family parameters and mapping their names to the corresponding object instances:
```csharp
FamilyManager mgr = doc.FamilyManager;

int n = mgr.Parameters.Size;

Debug.Print(
  "\nFamily {0} has {1} parameter{2}.",
  doc.Title, n, Util.PluralSuffix( n ) );

Dictionary<string, FamilyParameter> fps
  = new Dictionary<string, FamilyParameter>( n );

foreach( FamilyParameter fp in mgr.Parameters )
{
  string name = fp.Definition.Name;
  fps.Add( name, fp );
}
List<string> keys = new List<string>( fps.Keys );
keys.Sort();
```

I create a sorted list of the parameter names, so that the final result will be sorted and thus more readable.

Before we move on to the iteration over the family types and determining the family parameter values for each, here is a helper method to retrieve and format the family parameter value as a string.
I noticed that a string-valued parameter value can only be retrieved using AsString, and AsValueString returns an empty string for it, so I implemented a switch statement and an optimal specialised access and display for each storage type:
```csharp
static string FamilyParamValueString(
  FamilyType t,
  FamilyParameter fp,
  Document doc )
{
  string value = t.AsValueString( fp );
  switch( fp.StorageType )
  {
    case StorageType.Double:
      value = Util.RealString(
        ( double ) t.AsDouble( fp ) )
        + " (double)";
      break;

    case StorageType.ElementId:
      ElementId id = t.AsElementId( fp );
      Element e = doc.get\_Element( ref id );
      value = id.Value.ToString() + " ("
        + Util.ElementDescription( e ) + ")";
      break;

    case StorageType.Integer:
      value = t.AsInteger( fp ).ToString()
        + " (int)";
      break;

    case StorageType.String:
      value = "'" + t.AsString( fp )
        + "' (string)";
      break;
  }
  return value;
}
```

With the helper method in hand, we can iterate over the family types and display their family parameter values.
We use HasValue to determine whether a type has a value for each parameter, retrieve its value, and display the result:
```csharp
n = mgr.Types.Size;

Debug.Print(
  "Family {0} has {1} type{2}{3}",
  doc.Title,
  n,
  Util.PluralSuffix( n ),
  Util.DotOrColon( n ) );

foreach( FamilyType t in mgr.Types )
{
  string name = t.Name;
  Debug.Print( "  {0}:", name );
  foreach( string key in keys )
  {
    FamilyParameter fp = fps[key];
    if( t.HasValue( fp ) )
    {
      string value
        = FamilyParamValueString( t, fp, doc );

      Debug.Print( "    {0} = {1}", key, value );
    }
  }
}
```

Here is the result of opening a new family document based on the Metric Column template,
running the Family API lab 4 command to create a simple column family and some types, and then listing the family parameter values using CmdFamilyParamValues:

```
Family Family1 has 13 parameters.
Family Family1 has 5 types:
   :
    ColumnFinish = -1 ()
    Depth = 1.97 (double)
    Td = 0.49 (double)
    Tw = 0.49 (double)
    Width = 1.97 (double)
  1000x300:
    ColumnFinish = -1 ()
    Depth = 0.98 (double)
    Td = 0.25 (double)
    Tw = 0.82 (double)
    Width = 3.28 (double)
  600x900:
    ColumnFinish = -1 ()
    Depth = 2.95 (double)
    Td = 0.74 (double)
    Tw = 0.49 (double)
    Width = 1.97 (double)
  Glass:
    ColumnFinish = 574 (Materials <574 Glass>)
    Depth = 1.97 (double)
    Td = 0.49 (double)
    Tw = 0.49 (double)
    Width = 1.97 (double)
  600x600:
    ColumnFinish = -1 ()
    Depth = 1.97 (double)
    Td = 0.49 (double)
    Tw = 0.49 (double)
    Width = 1.97 (double)
```

Apparently, it includes one unnamed type as well as the four that our sample created intentionally.

I hope this clarifies the structure and usage of family parameters and possibly provides a useful additional little utility for your tool kit.

Here is
[version 1.1.0.53](zip/bc11053.zip)
of the complete Building Coder sample source code and Visual Studio solution including the new command.