---
post_number: "0398"
title: "Retrieve MEP Elements and Connectors"
slug: "retrieve_mep_elements"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'python', 'revit-api']
source_file: "0398_retrieve_mep_elements.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0398_retrieve_mep_elements.html"
---

### Retrieve MEP Elements and Connectors

My colleague Martin Schmid recently asked about the feasibility of listing all unconnected connectors in a Revit MEP model.
To do so, the first step is to retrieve all the elements that can potentially have connectors, and then examine their connectors to see which of them are not connected to a neighbouring element.

We looked at many examples of using the filtered element collectors in the past. A summary of previous posts is given in the discussion on
[parameter filters](http://thebuildingcoder.typepad.com/blog/2010/06/parameter-filter.html),
and we since also discussed a correction on using the
[element name parameter filter](http://thebuildingcoder.typepad.com/blog/2010/06/element-name-parameter-filter-correction.html) and
the recommendation
[not to retrieve all database elements](http://thebuildingcoder.typepad.com/blog/2010/06/filter-for-all-elements.html).

The types of MEP connectors that we are looking for include the following:

- Duct- Pipe- Conduit- Cable Tray- Electrical

The elements that can conceivably host such connectors include:

- Duct- Pipe- Conduit- Cable Tray- Wire- Fittings between the aforementioned objects, including certain categories of family instances.

After some initial prototyping, we discovered that wires are special.
In Martin's words:

The one area that may be suspect is wires.
If a wire is a homerun, it would be expected that one of its ends would be disconnected.
However, it does not seem to be possible to check if the wire is a homerun or not.
There is no property such as IsHomerun.
Another thought that comes to mind is that whether a wire is intended to be a homerun or not cannot be known:

![Disconnected homerun](img/loose_connector_disconnected_homerun.png)

I therefore suggest that we skip checks on 'Electrical' connectors, but still check for Conduit and Cable Tray connectors.

With that in mind, I implemented the following method GetConnectorElements to retrieve all elements in given document which may have some kind of MEP connector.
I start off with a list of all categories of family instances that I am interested in, OR them all together, and AND that category list with the family instance type.
The resulting specialised family instance filter is ORed with the other types of interest, and a filtered element collector is created from that.
The categories included in the list but commented out are actually only used by the specialised
MEP elements that we retrieve anyway by their System.Type.
When actually using it, I have the choice of including wires or not by setting the Boolean argument include\_wires:

```csharp
static FilteredElementCollector GetConnectorElements(
  Document doc,
  bool include\_wires )
{
  // what categories of family instances
  // are we interested in?

  BuiltInCategory[] bics = new BuiltInCategory[] {
    //BuiltInCategory.OST\_CableTray,
    BuiltInCategory.OST\_CableTrayFitting,
    //BuiltInCategory.OST\_Conduit,
    BuiltInCategory.OST\_ConduitFitting,
    //BuiltInCategory.OST\_DuctCurves,
    BuiltInCategory.OST\_DuctFitting,
    BuiltInCategory.OST\_DuctTerminal,
    BuiltInCategory.OST\_ElectricalEquipment,
    BuiltInCategory.OST\_ElectricalFixtures,
    BuiltInCategory.OST\_LightingDevices,
    BuiltInCategory.OST\_LightingFixtures,
    BuiltInCategory.OST\_MechanicalEquipment,
    //BuiltInCategory.OST\_PipeCurves,
    BuiltInCategory.OST\_PipeFitting,
    BuiltInCategory.OST\_PlumbingFixtures,
    BuiltInCategory.OST\_SpecialityEquipment,
    BuiltInCategory.OST\_Sprinklers,
    //BuiltInCategory.OST\_Wire,
  };

  IList<ElementFilter> a
    = new List<ElementFilter>( bics.Count() );

  foreach( BuiltInCategory bic in bics )
  {
    a.Add( new ElementCategoryFilter( bic ) );
  }

  LogicalOrFilter categoryFilter
    = new LogicalOrFilter( a );

  LogicalAndFilter familyInstanceFilter
    = new LogicalAndFilter( categoryFilter,
      new ElementClassFilter(
        typeof( FamilyInstance ) ) );

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

  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.WherePasses( classFilter );

  return collector;
}
```

Once I have collected all elements that may contain connectors, I want to iterate over all their connectors and check each one to see whether it is connected to a neighbouring element at the other side.

All connectors on a Revit MEP element are accessed through the Connectors collection of a connector manager which is available through the element properties.
Unfortunately, the access to the connector manager varies somewhat by element type, so I implemented a dedicated function to unify the different access methods:

- On a family instance, the connector manager is accessible as a property of the MEPModel property, both of which may be null.
- On a wire, the connector manager is directly accessible as a property of the Wire class.
- All other MEP elements are derived from the MEPCurve base class, which also provides direct access to the connector manager through a property.

This leads to following implementation of the GetConnectors method, which returns the connectors of any given element, or null if none are present:

```python
static ConnectorSet GetConnectors( Element e )
{
  ConnectorSet connectors = null;

  if( e is FamilyInstance )
  {
    MEPModel m = ( ( FamilyInstance ) e ).MEPModel;

    if( null != m
      && null != m.ConnectorManager )
    {
      connectors = m.ConnectorManager.Connectors;
    }
  }
  else if( e is Wire )
  {
    connectors = ( ( Wire ) e )
      .ConnectorManager.Connectors;
  }
  else
  {
    Debug.Assert(
      e.GetType().IsSubclassOf( typeof( MEPCurve ) ),
      "expected all candidate connector provider "
      + "elements to be either family instances or "
      + "derived from MEPCurve" );

    if( e is MEPCurve )
    {
      connectors = ( ( MEPCurve ) e )
        .ConnectorManager.Connectors;
    }
  }
  return connectors;
}
```

With these tools in hand, I am ready to address Martin's original intent, which will be discussed in a future post.