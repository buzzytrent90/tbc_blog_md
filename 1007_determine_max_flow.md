---
post_number: "1007"
title: "Determining Maximal Flow in HVAC Duct Connectors"
slug: "determine_max_flow"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'python', 'revit-api', 'transactions']
source_file: "1007_determine_max_flow.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1007_determine_max_flow.html"
---

### Determining Maximal Flow in HVAC Duct Connectors

A colleague came up with a request for a simple Revit MEP HVAC utility add-in that I would like to present.

**Question:** We need to extract some specific ductwork values in Revit MEP and report them to the user.

To begin with, all we need is the largest flow value on each duct, exported to a shared parameter on the duct.

The purpose of the routine is for the users to confirm their duct sizing with this data reflecting the worst case condition.
For this reason, it is required only on the ducts, as the fittings sizes will be driven by the duct sizes.

I created the following shared parameter file defining the 'Duct Max Airflow' parameter that I would like to populate:

```
*GROUP ID NAME
GROUP   1 Mechanical - Airflow
*PARAM  GUID  NAME   DATATYPE  DATACATEGORY  GROUP  VISIBLE
PARAM   XXXX  Duct Max Airflow HVAC_AIR_FLOW  1 1
```

**Answer:** This is a very simple issue, really.
It requires the following steps:

- Retrieve all duct elements.
- For each, retrieve all its HVAC connectors.
- Determine their maximal Flow property value.
- Populate the shared parameter.

The first step is almost always the implementation of a representative, minimal and yet complete sample model to research and test in.
It needs to contain as few elements as possible to simplify navigation and identification, and still provide examples of all the various situations that need to be handled by the add-in.

Here is a sample file to start with:

![Sample model](img/duct_max_flow_sample_rvt.png)

You can then explore this model interactively using RevitLookup and the
[BipChecker](http://thebuildingcoder.typepad.com/blog/2013/01/built-in-parameter-enumeration-duplicates-and-bipchecker-update.html) to
find out how to access the required data.
Here is an example showing where the desired flow values can be found on the duct connectors:

![Snooping max flow on duct](img/duct_max_flow_snoop.png)

After some further analysis, we discover that a possible way to structure this add-in breaks it up into the following tasks and methods:

- [GetConnectorManager](#2) – return the connector manager for a given element.
- [GetMaxFlow](#3) – return the max flow for a given set of connectors.
- [SetMaxFlowOnElement](#4) – determine the max flow for a given element and set its shared parameter value.
- [Execute](#5) – external command mainline: create a transaction, retrieve and process all ducts.

#### Return the Connector Manager for an Element

We initially thought we would have to process both ducts and fittings.

They use different properties to access their connector manager.

The following method encapsulates and hides that difference.
This makes it possible to implement higher level generic methods that handle both types of elements identically:

```python
  /// <summary>
  /// Return the given element's connector manager,
  /// using either the family instance MEPModel or
  /// directly from the duct connector manager.
  /// </summary>
  static ConnectorManager GetConnectorManager(
    Element e )
  {
    Duct duct = e as Duct;

    return null == duct
      ? ( e as FamilyInstance ).MEPModel.ConnectorManager
      : duct.ConnectorManager;
  }
```

#### Return Max Flow for a Connector Set

The following method iterates over all connectors in the given set and returns the maximum flow value.

The Flow property throws an exception unless the connector belongs to the HVAC or piping domain, so all other connectors are skipped:

```python
  /// <summary>
  /// Retrieve max flow from all the given connectors.
  /// </summary>
  static double GetMaxFlow(
    ConnectorSet connectors )
  {
    double flow = 0.0;

    foreach( Connector c in connectors )
    {
      // Accessing flow property requires these
      // domains or throws an exception saying
      // "Flow is available only for connectors
      // of DomainHavc and DomainPiping."

      Domain d = c.Domain;

      if( Domain.DomainHvac != d
        && Domain.DomainPiping != d )
      {
        continue;
      }

      if( flow < c.Flow )
      {
        flow = c.Flow;
      }
    }
    return flow;
  }
```

#### Determine Max Flow for an Element and Set its Shared Parameter

We use the GUID to identify the 'Duct Max Airflow' shared parameter in a language independent way.

Then all that need to be done to process a duct element is retrieve its connectors, determine their max flow value and populate the parameter, returning an error if the latter fails:

```python
  /// <summary>
  /// Identify the 'Duct Max Airflow' shared parameter.
  /// </summary>
  static Guid \_shared\_param\_duct\_max\_airflow
    = new Guid( "87b12ca4-8a4c-4731-bf88-f50bccd9c5d4" );

  /// <summary>
  /// Set the max flow parameter on the given
  /// element and return true on success.
  /// This is generic, so it can handle both
  /// ducts and fittings. Later, this proved
  /// unnecessary, and we use ot for ducts only.
  /// </summary>
  static bool SetMaxFlowOnElement( Element e )
  {
    ConnectorSet connectors
      = GetConnectorManager( e ).Connectors;

    int n = connectors.Size;

    double flow = GetMaxFlow( connectors );

    Debug.Print(
      "{0} has {1} connector{2} and max flow {3}.",
      Util.ElementDescription( e ), n,
      Util.PluralSuffix( n ), flow );

    Parameter p = e.get\_Parameter(
      \_shared\_param\_duct\_max\_airflow );

    bool rc = false;

    if( null == p )
    {
      //Util.InfoMsg( "Please ensure that all "
      //  + "duct and their fittings have a "
      //  + "'Duct Max Airflow' shared parameter." );

      Debug.Print( "{0} has no 'Duct Max Airflow' "
        + "shared parameter.",
        Util.ElementDescription( e ) );
    }
    else
    {
      // Store the max flow value in the specified
      // parameter on the given element.

      rc = p.Set( flow );
    }
    return rc;
  }
```

This method is implemented in a generic fashion that enables it to handle fittings as well as duct elements, although we are currently only using it for the latter.

#### Mainline to Retrieve and Process all Ducts

The external command mainline creates a transaction to enable database modification, retrieves all ducts and processes them one at a time:

```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;

    if( null == uidoc )
    {
      Util.InfoMsg( "Please run this command "
        + "in a valid document context." );

      return Result.Failed;
    }

    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    int nDucts = 0;

    using( Transaction tx = new Transaction( doc ) )
    {
      tx.Start( Util.Caption );

      FilteredElementCollector ducts
        = new FilteredElementCollector( doc )
          .OfClass( typeof( Duct ) );

      foreach( Duct duct in ducts )
      {
        if( SetMaxFlowOnElement( duct ) )
        {
          ++nDucts;
        }
        else
        {
          return Result.Failed;
        }
      }
      tx.Commit();
    }

    Util.InfoMsg( string.Format(
      "Set max flow parameter on {0} duct{1}.",
      nDucts, Util.PluralSuffix( nDucts ) ) );

    return Result.Succeeded;
  }
```

We originally retrieved and processed the fittings in the same manner, but that proved unnecessary, so the relevant code has been commented out again.
You can still see it in the source code download below, though, in case you have need for that functionality for other purposes.

#### Conclusion

As said, this is all very simple.

Next steps might include adding an external application to define a slightly nicer user interface and implementing additional functionality to populate other parameters, check for open connectors, etc.

For the moment, here is
[FlowParam02.zip](zip/FlowParam02.zip) containing
the complete source code, Visual Studio solution and add-in manifest for the command in its current state.