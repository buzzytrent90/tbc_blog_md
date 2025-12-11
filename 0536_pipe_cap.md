---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.124608'
original_url: https://thebuildingcoder.typepad.com/blog/0536_pipe_cap.html
post_number: '0536'
reading_time_minutes: 5
series: mep
slug: pipe_cap
source_file: 0536_pipe_cap.htm
tags:
- csharp
- elements
- family
- filtering
- parameters
- references
- revit-api
- transactions
- mep
title: Create a Pipe Cap
word_count: 936
---

### Create a Pipe Cap

Things have been a bit hectic for me in the last few days, with several complex cases to solve.
I also suddenly and unexpectedly had to find out how to apply for a visa to Saudi Arabia, which is a kind of interesting process.
Apparently, you start out with the inviting company in Saudi Arabia, which has to issue an invitation letter. This has to include a reference number, for which a form needs to be filled in and submitted to the Ministry of Foreign Affairs.
With that in place, a letter from my company addressed to the embassy of Saudi Arabia in Bern can be submitted to request the visa. This letter must be legalised by the chamber of commerce in Switzerland.
So quite an impressive process before I can even get started applying...

Anyway, another thing that came up today that was immediately solvable is the following simple question on how to create a new cap on a pipe in the Revit MEP context:

**Question:** The Revit SDK samples do not include any example showing how to create a cap on as pipe.
Can you show me how to do it in C#?

Let's say I create a pipe as follows:
```csharp
  UIApplication uiapp = commandData.Application;
  Document rvtDoc = uiapp.ActiveUIDocument.Document;

  XYZ start = new XYZ( 0, 0, 0 );
  XYZ end = new XYZ( 6, 0, 4 );

  FilteredElementCollector collector
    = new FilteredElementCollector( rvtDoc );

  collector.OfClass( typeof( ElementType ) );

  PipeType pipeType = collector.FirstElement()
    as PipeType;

  Pipe pipe = rvtDoc.Create.NewPipe(
    start, end, pipeType );

  pipe.get\_Parameter(
    BuiltInParameter.RBS\_PIPE\_DIAMETER\_PARAM )
    .Set( 0.1667 ); // revise to 2" dia pipe
```

In the project browser I see the node Families > Pipe Fittings > M\_Cap - Generic > Standard.

How can I use that family to create a 2" dia cap at the start point of the 2" dia pipe in C#?

**Answer:** The solution is simple, actually, in the following two steps:

1. Insert a cap family instance.- Connect it to the pipe.

By the way, in your code, you are selecting the pipe type from a collector returning all element types.
That seems a bit liberal to me.
I would suggest adding a filter for the built-in category as well.

Here is my code that does just that.
First, I need to define a few constants to identify and load the cap symbol:
```csharp
  const string \_libFolder = "C:/Documents and Settings"
    + "/All Users/Application Data/Autodesk/RME 2011"
    + "/Metric Library/Pipe/Fittings/Generic";

  const string \_rfaExtension = ".rfa";
  const string \_capFamilyName = "M\_Cap - Generic";
  const string \_capSymbolName = "Standard";
```

I then create the pipe like before, select the cap family symbol, and insert an instance of it at the end point of the pipe:
```csharp
  UIApplication uiapp = commandData.Application;
  Document doc = uiapp.ActiveUIDocument.Document;

  PipeType pipeType
    = new FilteredElementCollector( doc )
    .OfClass( typeof( ElementType ) )
    .OfCategory( BuiltInCategory.OST\_PipeCurves )
    .FirstElement() as PipeType;

  XYZ start = new XYZ( 0, 0, 0 );
  XYZ end = new XYZ( 6, 0, 4 );

  Transaction t = new Transaction( doc,
    "Create Pipe Cap" );

  t.Start();

  Pipe pipe = doc.Create.NewPipe(
    start, end, pipeType );

  pipe.get\_Parameter(
    BuiltInParameter.RBS\_PIPE\_DIAMETER\_PARAM )
    .Set( 0.1667 ); // revise to 2" dia pipe

  // get cap symbol:

  List<FamilySymbol> symbols = new List<FamilySymbol>(
    new FilteredElementCollector( doc )
      .OfCategory( BuiltInCategory.OST\_PipeFitting )
      .OfClass( typeof( FamilySymbol ) )
      .OfType<FamilySymbol>()
      .Where<FamilySymbol>( s
        => s.Family.Name.Equals( \_capFamilyName ) ) );

  FamilySymbol capSymbol = null;

  if( 0 < symbols.Count )
  {
    capSymbol = symbols[0];
  }
  else
  {
    string filename = Path.Combine( \_libFolder,
      \_capFamilyName + \_rfaExtension );

    // requires a transaction, obviously:

    doc.LoadFamilySymbol( filename, \_capSymbolName,
      out capSymbol );
  }

  Debug.Assert( capSymbol.Family.Name.Equals( \_capFamilyName ),
    "expected cap pipe fitting to be of family " + \_capFamilyName );

  FamilyInstance fi = doc.Create.NewFamilyInstance(
    end, capSymbol, StructuralType.NonStructural );
```

The result of running the code so far looks like this:

![Pipe cap unconnected](img/pipe_cap_unconnected.png)

This implements the first step above.
As you can see, the cap is not connected to the pipe, and it also has a wrong diameter.

To implement the second step, we need to pick the suitable connectors on the cap and the pipe and hook them up with each other.
On the cap, there is just one connector, so that is easy.

On the pipe, there are two, and it is more complex.
The order of connectors returned by the connector manager may change, so one should always use a location or some other criterion to select the right one.
In this case, we check the distance of the pipe connectors to the endpoint we are capping and pick the closest one.

Once the two connectors have been selected, we can connect them to each other:
```csharp
  // pick connector on cap:

  ConnectorSet connectors
    = fi.MEPModel.ConnectorManager.Connectors;

  Debug.Assert( 1 == connectors.Size,
    "expected nly one connector on pipe cap element" );

  Connector cap\_end = null;

  foreach( Connector c in connectors )
  {
    cap\_end = c;
  }

  // pick closest connector on pipe:

  connectors = pipe.ConnectorManager.Connectors;

  Connector pipe\_end = null;

  // the order of connectors returned by the
  // connector manager may change, so we
  // always use a location (or even more
  // information if several connectors are
  // at the same location) to get the right
  // connector!

  double dist = double.MaxValue;

  foreach( Connector c in connectors )
  {
    XYZ p = c.Origin;
    double d = p.DistanceTo( end );

    if( d < dist )
    {
      dist = d;
      pipe\_end = c;
    }
    break;
  }

  cap\_end.ConnectTo( pipe\_end );

  t.Commit();

  return Result.Succeeded;
```

The result of running the complete code looks like this:

![Pipe cap connected](img/pipe_cap_connected.png)

As you can see, the cap is now automatically positioned and dimensioned correctly with no further input required from my side.

Here is
[CreatePipeCap.zip](zip/CreatePipeCap.zip) containing
the entire source code and Visual Studio solution implementing this command.