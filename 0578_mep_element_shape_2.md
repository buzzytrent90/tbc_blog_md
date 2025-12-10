---
post_number: "0578"
title: "Improved MEP Element Shape and Ararat"
slug: "mep_element_shape_2"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'python', 'revit-api', 'windows']
source_file: "0578_mep_element_shape_2.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0578_mep_element_shape_2.html"
---

### Improved MEP Element Shape and Ararat

We recently talked about
[distinguishing Revit MEP duct element shapes](http://thebuildingcoder.typepad.com/blog/2011/03/distinguishing-mep-element-shape.html),
and Max aka
[Maciej Szlek](http://maciejszlek.pl) presented
a solution based on the analysis of the string value of the duct's Size parameter.

I suggested that the algorithm might be improved, both to make it more general and robust and also to remove the dependency on the different string formats used for imperial and metric units.
It probably becomes more efficient as well, by the way.

Instead of looking at the duct size parameter, one can query it for its connectors and extract the shape from them directly, avoiding all string manipulations and language or unit dependencies.

Max went ahead and implemented this new algorithm.
Simultaneously, he demonstrates that some useful helper methods can be reused from the Revit SDK samples, in this case ExtractMechanicalOrPipingSystem and ExtractSystemFromConnectors from the TraverseSystem sample.
Here is the code of the new algorithm:
```python
static public string GetElementShape(
  Element e,
  Element pe = null,
  Element ne = null )
{
  if( is\_element\_of\_category( e,
    BuiltInCategory.OST\_DuctCurves ) )
  {
    // assuming that transition is using to change shape..

    ConnectorManager cm = ( e as MEPCurve )
      .ConnectorManager;

    foreach( Connector c in cm.Connectors )
      return c.Shape.ToString()
        + " 2 " + c.Shape.ToString();
  }
  else if( is\_element\_of\_category( e,
    BuiltInCategory.OST\_DuctFitting ) )
  {
    MEPSystem system
      = ExtractMechanicalOrPipingSystem( e );

    FamilyInstance fi = e as FamilyInstance;
    MEPModel mm = fi.MEPModel;

    ConnectorSet connectors
      = mm.ConnectorManager.Connectors;

    if( fi != null && mm is MechanicalFitting )
    {
      PartType partType
        = ( mm as MechanicalFitting ).PartType;

      if( PartType.Elbow == partType )
      {
        // assuming that transition is using to change shape..

        foreach( Connector c in connectors )
        {
          return c.Shape.ToString()
            + " 2 " + c.Shape.ToString();
        }
      }
      else if( PartType.Transition == partType )
      {
        string[] tmp = new string[2];

        if( system != null )
        {
          foreach( Connector c in connectors )
          {
            if( c.Direction == FlowDirectionType.In )
              tmp[0] = c.Shape.ToString();

            if( c.Direction == FlowDirectionType.Out )
              tmp[1] = c.Shape.ToString();
          }
          return string.Join( " 2 ", tmp );
        }
        else
        {
          int i = 0;

          foreach( Connector c in connectors )
          {
            if( pe != null )
            {
              if( is\_connected\_to( c, pe ) )
                tmp[0] = c.Shape.ToString();
              else
                tmp[1] = c.Shape.ToString();
            }
            else
            {
              tmp[i] = c.Shape.ToString();
            }
            ++i;
          }

          if( pe != null )
            return string.Join( " 2 ", tmp );

          return string.Join( "-", tmp );
        }
      }
      else if( PartType.Tee == partType
        || PartType.Cross == partType
        || PartType.Pants == partType
        || PartType.Wye == partType )
      {
        string from, to;
        from = to = null;
        List<string> unk = new List<string>();

        if( system != null )
        {
          foreach( Connector c in connectors )
          {
            if( c.Direction == FlowDirectionType.In )
              from = c.Shape.ToString();
            else
              unk.Add( c.Shape.ToString() );

            if( ne != null && is\_connected\_to( c, ne ) )
              to = c.Shape.ToString();
          }

          if( to != null )
            return from + " 2 " + to;

          return from + " 2 " + string.Join( "-",
            unk.ToArray() );
        }
        else
        {
          foreach( Connector c in connectors )
          {
            if( ne != null && is\_connected\_to(
              c, ne ) )
            {
              to = c.Shape.ToString();
              continue;
            }

            if( pe != null && is\_connected\_to(
              c, pe ) )
            {
              from = c.Shape.ToString();
              continue;
            }

            unk.Add( c.Shape.ToString() );
          }

          if( to != null )
            return from + " 2 " + to;

          if( from != null )
            return from + " 2 "
              + string.Join( "-", unk.ToArray() );

          return string.Join( "-", unk.ToArray() );
        }
      }
    }
  }
  return "unknown";
}
```

It is encapsulated as a static method in the class MepElementShape.
I created a second wrapper class MepElementShapeV1 for the first implementation and call them both in The Building Coder sample CmdMepElementShape external command Execute method like this:
```csharp
  Element e = <selected element>;

  Util.InfoMsg( string.Format(
    "{0} is {1} ({2})",
    Util.ElementDescription( e ),
    MepElementShape.GetElementShape( e ),
    MepElementShapeV1.GetElementShape( e ) ) );
```

This prints the following in the Visual Studio debug output window on selecting some of the elements generated by the AvoidObstruction SDK sample:

```
Mechanical Equipment Parallel Fan Powered VAV
  <378728 Size 4 - 10 inch Inlet> is unknown (unknown)

Duct Fittings Rectangular Duct Radius Elbow <382720 1.5 W>
  is RectProfile 2 RectProfile (rectangular2rectangular)

Ducts <382719 Radius Elbows / Tees>
  is RectProfile 2 RectProfile (rectangular)

Duct Fittings Rectangular Duct Tee <382736 Standard>
  is RectProfile 2 RectProfile-RectProfile (unknown)

Air Terminals Supply Diffuser
  <378716 24 x 24 Face 12 x 12 Connection> is unknown (unknown)
```

As you see, the results are the same for the string-based first version as for the new connector-based one, except for the tee junction, which was not handled properly by the former.

Very many thanks to Max for implementing and sharing this!

Here is an updated
[version 2012.0.87.3](zip/bc_12_87_3.zip) of
The Building Coder sample code including this new classification method.

#### Climbing the Ararat

I am off for another break now, this time a bit more adventurous, to climb
[Mount Ararat](http://en.wikipedia.org/wiki/Mount_Ararat) in far Eastern Turkey.
The flight from Istanbul to
[Van](http://simple.wikipedia.org/wiki/Van,_Turkey), the closest city, is longer than the one from Switzerland to Istanbul!

Since I am also busy preparing my
[Autodesk University proposals](http://through-the-interface.typepad.com/through_the_interface/2011/04/call-for-au-2011-proposals.html),
which are due for submission today, as well as the upcoming Revit 2012 API webcast, I am pretty occupied and having difficulty finding time to even pack.
With skis and winter clothing and mountain equipment, that is a non-trivial task...
Still, I will make it, I am sure.

![Mount Ararat](img/ararat.jpg)