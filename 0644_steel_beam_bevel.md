---
post_number: "0644"
title: "Bevelled Steel Beams"
slug: "steel_beam_bevel"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'python', 'references', 'revit-api', 'schedules', 'transactions', 'views']
source_file: "0644_steel_beam_bevel.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0644_steel_beam_bevel.html"
---

### Bevelled Steel Beams

I recently mentioned a couple of
[options for performing Boolean operations](http://thebuildingcoder.typepad.com/blog/2011/06/boolean-operations-and-instancevoidcututils.html).
Here is another question which initially seemed related to that, from Simon Jones of Autodesk UK, on creating a
[mitre joint](http://en.wikipedia.org/wiki/Miter_joint) between
steel beams, i.e.
[bevelling](http://en.wikipedia.org/wiki/Bevel) the
two parts to be joined.
It turns out to have a satisfyingly simple and automatic answer with no need for any explicit Boolean operations after all.

Simon returned to a technical role in consulting after a period as Technical Sales Manager of Autodesk UK.
Even in his non-technical role, he always remained very interested and productive in technical aspects as well, for instance creating the
[UK Electrical Schedule Sample](http://thebuildingcoder.typepad.com/blog/2009/12/uk-electrical-schedule-sample.html) before
the introduction of the electrical panel schedule API in Revit 2011.

**Question:** Through the user interface, it is possible to cut back the end of a beam at an angle using the Cut tool and selecting a reference plane.

Within the API I see there is a SolidSolidCutUtils class but I can't find anything to cut a solid using a reference plane.

Is this possible in the Revit Structure 2011 API?
Or is there any other way of trimming back a beam?

What I actually wish to achieve is to enable a better workflow for creating steel staircases in RST2011.

One solution I'm considering is automating the creation of a structural frame to represent the stair stringers, treads and landing. Something that can be done manually but not very efficiently!

I saw the SolidSolidCut utility but didn't think this could help me as I want to edit beams within a project.

**Answer:** As I mentioned, there are a number of ways of
[performing Boolean operations](http://thebuildingcoder.typepad.com/blog/2011/06/boolean-operations-and-instancevoidcututils.html) in
the Revit API.
All of the options performing a Boolean operation in the project environment place a significant burden on the project, so the optimal approach is definitely to apply the cut in the family context, especially if it is something like a standard operation to shorten or add an angle to all the beam ends.

**Response:** It actually worked out much easier than I was expecting.
If the beam endpoints touch, Revit cleans them up on its own accord:

![Automatic beam cleanup in Revit Structure](img/beam_cleanup.png)

**Answer:** I used Simon's sample code to create a new external command CmdSteelStairBeams for The Building Coder samples.

Simon encapsulated the beam creation functionality in an own prototyping class named SteelStairs.
It defines the following helper method CreateBeam to implement the beam creation:
```csharp
FamilyInstance CreateBeam(
  FamilySymbol familySymbol,
  Level level,
  XYZ startPt,
  XYZ endPt )
{
  StructuralType structuralType
    = StructuralType.Beam;

  Line line = \_doc.Application.Create.NewLineBound(
    startPt, endPt );

  FamilyInstance beam = \_doc.Create
    .NewFamilyInstance( startPt, familySymbol,
      level, structuralType );

  LocationCurve beamCurve
    = beam.Location as LocationCurve;

  if( null != beamCurve )
  {
    beamCurve.Curve = line;
  }
  return beam;
}
```

This helper method is called by the main Run method to create a series of three connected steel beams between four hard-wired 3D points in the XZ plane, with Y equal to zero:
```csharp
public void Run()
{
  View view = \_doc.ActiveView;

  Level level = view.GenLevel;

  if( null == level )
  {
    throw new Exception( "No level associated with view" );
  }

  XYZ pt1 = Util.MmToFoot( new XYZ( 0, 0, 1000 ) );
  XYZ pt2 = Util.MmToFoot( new XYZ( 1000, 0, 1000 ) );
  XYZ pt3 = Util.MmToFoot( new XYZ( 2000, 0, 2500 ) );
  XYZ pt4 = Util.MmToFoot( new XYZ( 3000, 0, 2500 ) );

  FamilySymbol familySymbol = Util.FindFamilySymbol(
    \_doc,
    CmdSteelStairBeams.FamilyName,
    CmdSteelStairBeams.SymbolName );

  if( familySymbol == null )
  {
    throw new Exception( "Beam Family not found" );
  }

  CreateBeam( familySymbol, level, pt1, pt2 );
  CreateBeam( familySymbol, level, pt2, pt3 );
  CreateBeam( familySymbol, level, pt3, pt4 );
}
```

The class constructor does nothing more than initialise the one and only class data member for the current document:
```csharp
Document \_doc;

public SteelStairs( Document doc )
{
  \_doc = doc;
}
```

The SteelStairs helper class is driven by the following external command mainline Execute method, which mainly fusses about to manage a transaction and ensure that the appropriate rectangular hollow section steel beam family and symbol is loaded.
The family and symbol names to use depend on the exact flavour of Revit content that you have installed.
The rectangular hollow section steel beam family is provided in both the architectural and structural libraries.
In my case, I am loading it from the architectural content.

I hope you enjoy my use of the generic Any template method to check whether a family of the specified name has already been loaded.
By the way, I just noticed a flaw in my logic: while I ensure that the family is available, I do not check for the specific symbol as well.
This command will fail if some other symbols have been loaded, but not the one required.
I leave that as an exercise to the reader to clear up, though.

Once the beam symbol is loaded, all that is needed is to call the SteelStairs constructor and Run method:
```python
[Transaction( TransactionMode.Manual )]
class CmdSteelStairBeams : IExternalCommand
{
  public const string FamilyName
    = "RHS-Rectangular Hollow Section";

  public const string SymbolName
    = "160x80x4RHS";

  const string \_extension
    = ".rfa";

  const string \_directory
    = "C:/ProgramData/Autodesk/RAC 2012/Libraries"
    + "/US Metric/Structural/Framing/Steel/";

  const string \_family\_path
    = \_directory + FamilyName + \_extension;

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    Transaction t = new Transaction( doc );

    t.Start( "Create Steel Stair Beams" );

    // Check whether the required family is loaded:

    FilteredElementCollector collector
      = new FilteredElementCollector( doc )
        .OfClass( typeof( Family ) );

    // If the family is not already loaded, do so:

    if( !collector.Any<Element>(
      e => e.Name.Equals( FamilyName ) ) )
    {
      FamilySymbol symbol;

      if( !doc.LoadFamilySymbol(
        \_family\_path, SymbolName, out symbol ) )
      {
        message = string.Format(
          "Unable to load '{0}' from '{1}'.",
          SymbolName, \_family\_path );

        t.RollBack();

        return Result.Failed;
      }
    }

    try
    {
      // Create a couple of connected beams:

      SteelStairs s = new SteelStairs( doc );

      s.Run();

      t.Commit();

      return Result.Succeeded;
    }
    catch( Exception ex )
    {
      message = ex.Message;

      t.RollBack();

      return Result.Failed;
    }
  }
}
```

This is the result of running this command in Revit Architecture instead of Revit Structure:

![Automatic beam cleanup in Revit Archicture](img/beam_cleanup2.png)

Here is
[version 2012.0.90.0](zip/bc_12_91.zip) of
The Building Coder samples including the new command CmdSteelStairBeams.