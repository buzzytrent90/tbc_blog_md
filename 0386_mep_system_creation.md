---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: code_example
optimization_date: '2025-12-11T11:44:13.860645'
original_url: https://thebuildingcoder.typepad.com/blog/0386_mep_system_creation.html
post_number: 0386
reading_time_minutes: 6
series: mep
slug: mep_system_creation
source_file: 0386_mep_system_creation.htm
tags:
- elements
- family
- python
- references
- revit-api
- selection
- transactions
- views
- windows
- mep
title: MEP System Creation
word_count: 1282
---

### MEP System Creation

Yesterday was a great day here at the AEC DevCamp, except that I was unable to attend a session held by Scott Conover presenting material by Arnošt Löbel, since it collided with my own.
I enjoyed myself in the latter, anyway, and so did the audience, it seemed.

Meanwhile, life outside DevCamp is also continuing, and here is a very interesting case submitted by my old friend Thomas Wegener and handled by Joe Ye on Revit MEP system creation.
Our MEP expert Martin Schmid and Jorgen Dahl have also chipped in to help answer this.

**Question:** I am working at creating a new duct system using the Revit API method

```
MechanicalSystem NewMechanicalSystem(
  Connector baseEquipmentConnector,
  ConnectorSet connectors,
  DuctSystemType );
```

According to the API documentation, the base equipment is optional for the system, so this argument may be a null reference.

In spite of this, I am always getting an exception if I try to create the system without the base equipment. Why?

I have also the feeling that Revit is checking the connector directions for specific systems, e.g. an air supply system requires a base equipment with an "out connector", and an air return system requires an "in connector".
Is that true?
If so, is there a rule list for all pipe and duct systems?

Or are there general requirements like:

- A system is always structured as a tree, with a singleton root, the base equipment connector.- If the base equipment connector is an "out connector" all other equipment connectors must be "in" and vice versa.

**Answer:** Yes, you are correct in that these kind of rules exist and they are not completely documented in the API reference material.

I implemented the following test code to create a new mechanical system without a base equipment.
It can be used on the RVT model file provided with the AutoRoute sample in the 2011 SDK.
Please select the two air terminals and launch the command.
It creates a mechanical system consisting of these two air terminals:
```python
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
class CmdNewDuctSystem : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    UIDocument uidoc = app.ActiveUIDocument;
    Document doc = uidoc.Document;

    Transaction trans = new Transaction( doc,
      "New Duct System" );

    trans.Start();

    ConnectorSet connectorSet = new ConnectorSet();

    Connector baseConnector = null;

    ConnectorSetIterator csi;

    // select a Parallel Fan Powered VAV
    // and some Supply Diffusers prior to running
    // this command

    ElementSet selection = uidoc.Selection.Elements;

    foreach( Element e in selection )
    {
      if( e is FamilyInstance )
      {
        FamilyInstance fi = e as FamilyInstance;

        Family family = fi.Symbol.Family;

        // assume the selected Mechanical Equipment
        // is the base equipment for new system:

        if( family.FamilyCategory.Name
          == "Mechanical Equipment" )
        {
          // find the "Out" and "SupplyAir" connectors
          // on the base equipment

          if( null != fi.MEPModel )
          {
            csi = fi.MEPModel.ConnectorManager
              .Connectors.ForwardIterator();

            while( csi.MoveNext() )
            {
              Connector conn = csi.Current as Connector;

              if( conn.Direction == FlowDirectionType.Out
                && conn.DuctSystemType == DuctSystemType.SupplyAir )
              {
                baseConnector = conn;
                break;
              }
            }
          }
        }
        else if( family.FamilyCategory.Name == "Air Terminals" )
        {
          // add selected Air Terminals to
          // connector set for new mechanical system

          csi = fi.MEPModel.ConnectorManager
            .Connectors.ForwardIterator();

          csi.MoveNext();

          connectorSet.Insert( csi.Current as Connector );
        }
      }
    }

    // create a new SupplyAir mechanical system

    MechanicalSystem ductSystem = doc.Create.NewMechanicalSystem(
      baseConnector, connectorSet, DuctSystemType.SupplyAir );

    trans.Commit();
    return Result.Succeeded;
  }
}
```

Regarding the base connector direction, I didn't see any documentation that explicitly points out this rule.
According to my test, in a supply air system, if I select a connector whose direction is FlowDirectionType.In, then it fails to create the system, so the following rule that you assumed probably does apply:

- If the base equipment connector is an "out connector" all other equipment connectors must be "in" and vice versa.

**Response:** Thank you, you are right, your code works for me as well without providing a base connector for the SupplyAir system.

If you change the DuctSystemType.SupplyAir in the call to NewMechanicalSystem to DuctSystemType.ReturnAir your code will fail too in the AutoRoute model.
It works again if you insert return air terminals instead (also without the base connector).

I am still unclear about the constraints between:

- Base connector direction and system types, for both duct and pipe.- ConnectorSet and system types, for both duct and pipe.

**Answer:** According to my testing so far, I found that this is necessary: an air supply system needs an 'out connector' as a base equipment connector.

The 'rules' you refer to are right on.

Here is the material from Martin Schmid's presentation ME204-3 'Getting into the Flow: Understanding Connectors in Revit MEP Content' at Autodesk University 2008, which deals with this topic.
It does not cover everything, as quite a bit was left for the in-class demo, but should help:

- [PDF handout document](zip/ME204-3_Connectors_in_Revit_MEP_Content.pdf).- [PPT presentation](zip/ME204-3_Connectors_in_Revit_MEP_Content.ppt).

In summary, yes, the flow direction has to match and systems should be setup as a "tree".
The flow direction should be based on the system type.

Flow direction should be set to "out" on the base equipment and "in" on all other equipment for a supply system.
The reverse is true for return systems.

Here is the more detailed specification extracted from Martin's material:

#### Creating Duct Systems

Systems can be created from equipment or air terminals:

1. Supply Air systems can be created containing families with a Supply Air IN Connector- Return Air systems can be created containing families with a Return Air OUT Connector- Exhaust Air systems can be created containing families with an Exhaust Air OUT Connector

Note: You cannot create an 'Other' air system.

Mechanical Equipment Families can be the 'equipment for the system' using the following logic.
The equipment must have a:

1. Supply Air OUT connector to be the equipment for a Supply Air system.- Return Air IN connector to be the equipment for a Return Air system.- Exhaust Air IN connector to be the equipment for an Exhaust Air system.

Thus, it is possible to create a system named 'Relief Air' by using components with Return or Exhaust system connectors.
Similarly, a system named 'Outside Air' can be created using components with Supply system connectors.

So you assumption of the relationship between connector's direction and system type is correct.

Furthermore, just as you say, a system is always structured as a tree, with a singleton root, the base equipment connector.

**Final question:** You described rules for duct systems.
For pipe systems, Revit provides types like SupplyHydronic, ReturnHydronic, etc.
Are there similar restrictions as above defined for duct systems?

**Answer:** Yes, for pipe systems, the rules are similar.
The following rule of connector direction always applies in Revit MEP:

Flow direction has to match and systems should be setup as a tree.
What the flow direction should be is based on the system type.

Flow direction should be set to out on the base equipment and in on all other equipment for a supply system.
The reverse is true for return systems.

I packaged the test code provided by Joe above as a new external command CmdNewDuctSystem in The Building Coder samples.
Here is
[version 2011.0.71.0](zip/bc_11_71.zip)
of the complete source code and Visual Studio solution including the new command.

Many thanks to all involved for this interesting discussion.

Time for packing, checking out, breakfast and a new day at DevCamp now.
Once again I am admiring a wonderful view on the brilliant morning outside the window.
Again the cloudless blue sky which started out in all kinds of beautiful hues shifting from red and pink and orange through to clear blue and sunny now.

My session today is on the Revit MEP API, so this was a good warm-up to get into the subject...