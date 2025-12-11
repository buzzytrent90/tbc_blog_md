---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.049027'
original_url: https://thebuildingcoder.typepad.com/blog/0983_space_adjacency.html
post_number: 0983
reading_time_minutes: 15
series: general
slug: space_adjacency
source_file: 0983_space_adjacency.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- references
- revit-api
- rooms
- views
- walls
title: Football and Space Adjacency for Heat Load Calculation
word_count: 3010
---

### Football and Space Adjacency for Heat Load Calculation

Here is a summary of a discussion with Jasper Desmet, who is studying engineering and working on a research project for his master thesis concerning heat load calculation using BIM software, proving that it is possible for a novice programmer to extract the required information from the model using the Revit API.

#### Football Tournament, Fleece and Shard

First, though, let me mention that I visited the UK this weekend to participate in the Autodesk European soccer tournament, which took place on Saturday at the
[Royal Holloway](http://www.rhul.ac.uk) University in Egham:

![Neuchâtel Eagles at the Royal Holloway in Egham](file:////j/photo/jeremy/2013/2013-07-14_london/1360.jpg)

Sunday I took a walk along the Thames from Waterloo to London Bridge on the way to the airport, and happened to pass by the
[Bargehouse](http://southbanklondon.com/bargehouse) exhibition space, currently housing the
[Fleece](http://fleeceshow.com) photography exhibition including 'Climax' by
[Holly Buckle](http://fleeceshow.com/artists/Holly_Buckle.html):

!['Climax' by Holly Buckle in the Bargehouse](file:////j/photo/jeremy/2013/2013-07-14_london/1457.jpg)

Among the many other fascinating images, I also discovered a poem by
[Lord Byron](http://en.wikipedia.org/wiki/Lord_Byron), from
[Childe Harold's Pilgrimage](http://en.wikipedia.org/wiki/Childe_Harold%27s_Pilgrimage), Canto iv, Verse 178, which reflects my opinion of nature and society quite nicely:

There is a pleasure in the pathless woods,

There is a rapture on the lonely shore,

There is society, where none intrudes,

By the deep sea, and music in its roar:

I love not man the less, but Nature more ...

Finally, here is a close-up of the 72-storey
[Shard](http://the-shard.com) skyscraper,
currently the tallest building in the European Union, from the London Bridge station platform, just before boarding the train for Gatwick:

![Shard](file:////j/photo/jeremy/2013/2013-07-14_london/1470.jpg)

I think those are more than enough impressions from this eventful weekend, so let's return to the heat load calculation space adjacency discussion with Jasper:

#### Space Adjacency for Heat Load Calculation

**Question:** I'm doing my master thesis concerning heat load calculation using BIM-software (obviously I use Revit).
As a part of this thesis I'm writing an application to extract all necessary information from Revit, and calculate the heat load using Belgian standards. This is mainly to show the possibilities of automation using the Revit API.

As I am studying engineering: construction, programming is not my strong suit (although I love it very much), and your site has been invaluable for my efforts to making this application work. Thanks for all the help already (through the blog), but I write with a question:

To implement the calculation I need a lot of data about the spaces and their bounding elements.
I use the SpatialElementGeometryCalculator to calculate the geometry, and get the SpatialBoundaryElements, as shown in the Developer Guide wiki page on
[Room and Space Geometry](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00006-API_Developer's_Guide/0074-Revit_Ge74/0108-Geometry108/Room_and_Space_Geometry.) to
calculate a room's geometry and find its boundary faces.
This works great.

Apart from that I also need to know the space adjacencies, and I adapted the code you provided for determining
[space adjacencies](http://thebuildingcoder.typepad.com/blog/2009/07/space-adjacency.html) to
retrieve the segments and the adjacent space for each segment.

What I want to do now is merge these two different data sets into a single component class MySurface.

Here is a schematic view of the data I am collecting from the model that makes it easier to understand how I defined the MyComponents class to manage the data:

![Heat load calculation data schema](img/jd_heat_load_calculation_data_schema.jpeg)

Right now I can't find the link I need between the faces/subfaces of the SpatialElementGeometryCalculator and the segments from the BoundarySegments.

So basically my question is how to know which segment and which (sub)face go together.
Is there a way to achieve this?
Is there a way to get the boundary segments through the SpatialElementGeometryCalculator?

**Answer:** Thank you for raising this interesting topic.

Some questions back:

- What exact data have you already retrieved?
- 2D space boundaries + adjacent spaces on each side?
- SpatialElementGeometryCalculator faces and sub-faces?
- Do the SpatialElementGeometryCalculator not tell you the originating element producing each face?

You can ask a space to return its 3D volume and all its bounding elements in 3D.

**Response:** Here is some clarification on what I already have and the paths I’m considering next:

Certain assumptions are made to simplify the project:

- All boundary curves are linear and coincident with the possible calculated segments.
- The floor type is defined by the level (1 level, 1 floor).

As said, I made use of your
[CmdSpaceAdjacency](http://thebuildingcoder.typepad.com/blog/2009/07/space-adjacency.html) command
and added the adjacent space (\_adjSpace) to the Segment class, so that both the space of origin and the adjacent space (null for exterior walls) are available for each given segment, not only for the space.

```csharp
  #region Adjacencies
  #region Segment Class
  public class Segment
  {
    XYZ \_sp;
    XYZ \_ep;
    Space \_space;
    Space \_adjSpace = null;
    // . . .
```

So this is what I have at the moment.
A list of segments that give me access to the adjacencies of spaces at the given 2D-boundaries, and a list of surfaces containing geometric and thermal information about the spaces.

One problem is that the adjacencies at the moment are only in 2D (on the same level). For a correct heat load calculation you would need surface division for the floors (e.g. a bathroom [at 24°C for calculation], above an entrance hall [at 20°C for calculation] will need transmission calculation, whereas two rooms at the same temperature will not need that, so that surface information is necessary => see plan views in attachment). The SpatialElementGeometryCalculator does not take the layout of spaces above or below into consideration though (this I derived from the printed info of the faces). However much I would like to puzzle out how to implement this, that would take me too far, as this application is not the main focus of my master-thesis.

What I’m considering:

Now I’m looking for how to link both data together.
I’m considering one of two following options:

- Using the segments (2D boundaries) to find the corresponding face edge inside a space (comparing XYZ-points). Although this might work, it doesn’t solve the fact that only adjacency on the same level is considered.
- Implementing a similar adjacency system using the faces instead of the 2D-boundaries, using the normal vector as direction for the testing point (as this always points outward).

I think the second option is the more widely usable option, as it takes adjacency directly from the same calculated data, taking away certain problems. It might not be the most performant of the two, but I have no way of knowing this.

The first option might be the fastest to implement with what I have so far.

**Later:** I’ve been trying to get the adjacency system to work, and here’s what I got.
It works for the horizontal adjacencies (on the same level), but vertical adjacencies are still an issue as the SpatialElementGeometryCalculator does not divide floor/ceiling faces to reflect the adjacent level space layout (for differences in internal design temperature between different spaces).
I have an idea on how to solve that issue later.

Here is what I do:

My method for retrieving the Space data from a given Space in Revit is implemented by the GetSpaceData method below.

For the given Space I retrieve the Geometry through the SpatialElementGeometryCalculator.
For each face of the Space I do the following:

- Create a personal Surface component to store the data retrieved for later use.
- Retrieve the defining points of the edges of the face, for calculation of a point inside the face (if necessary).
- Retrieve the normal-vector of the face (which according to the information I found always points outwards of the Solid the Face is part of), for retrieval of the adjacent Space.
- Retrieve the adjacent Space through the method FindAdjacentSpaceFromSurface, which takes the following arguments: the considered Face, a list of XYZ points defining the polygon, and the normal-vector of the face.

- At first I tried to use a Ray Projection (ReferenceIntersector) to retrieve the adjacent space (to ‘poke’ into a certain direction), but this resulted in a failure, and my guess is that it’s because the Space element is not considered 3D-geometry and is not ‘visible’ in the 3D-view, and can thereby not be found by intersection.
- As an adjacent Space has a certain proximity to the other space I construct a test point from a point on the Face, using the normal and a MaxWallThickness-constant to make sure the test point surpasses the bounding Element responsible for the Space-face creation.
- Then I test if this point is inside a Space of the Project (using a List of spaces constructed earlier on in the application), and if it is I assign this space as the adjacent Space to the considered Space in GetSpaceData-method.

- Through the subfaces I then retrieve the necessary data for the calculations, as through these subfaces I can access the boundary element responsible for the surface creation. Floors however don’t have a boundary element (and as I have come to notice, nor has my second floor upper surface (my roof is not considered a bounding element, and I don’t see why…).
  The method FindAdjacentSpaceFromSurface listed below to retrieve the underlying Floor, but it currently causes a fatal error and closes the Revit session as soon as it hits the second floor Spaces.

So, in conclusion, I successfully asses the space adjacency for horizontal adjacencies (although maybe not totally fool proof), but still have an issue with the vertical adjacencies, as to which space is adjacent for which portion of the surface. I also have an issue with determining the floor of a space, as my attempt to use Ray Projection to do so results in a fatal error. Also I don’t understand why the RoofBase-element is not a bounding element for my Spaces on the top-most floor, have you got an idea as to why this is?

**Answer:** The issue with the SpatialElementGeometryCalculator may be due to modelling errors.
I cannot say for sure, of course, just offer this as a wild guess.

You say that you suspect that the ReferenceIntersector does not consider the Space element as 3D-geometry and it is therefore not ‘visible’ in the 3D-view.
Yes, I agree.

The other issues you report are not surprising.
I run into similar issues when I try new things.
It is often an unclear mix between imperfect modelling and rather sensitive API methods with detailed input requirements.

By the way, the new
[CustomExporter class](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html),
for which I just published a couple of samples, provides another completely different way to access surfaces which may be of interest to you as well.

**Response:** Your wild (educated) guess was right.
The problem with the Roof not being a BoundingElement of the Space was due to a modelling error.
The space top constraint was ‘level 2’, which was also the level the roof was drawn on, so the space did not run totally up to the roof.

After fixing this, the Roof became a BoundingElement and the output was correct.

Even better, because I took away this modelling error, the fatal error concerning the ReferenceIntersector was also gone, and now I’m able to implement this to determine the floor type of a Space.
Thanks a lot for this helpful input, it saved me a lot of time (and sweat :-).

That fixed the most important issues.
For now I’ve got what I need to get my small thesis-application to work.
As to the issue with vertical adjacency (the only one left), I will try to look into that, but I don’t think I’ll have time to do so before the end of august (which is when my thesis is due). It’s an interesting issue though, and an important one for Energy Analysis, I think.

Here is the code in its current state, and working for me in my test project:

```csharp
public void GetSpaceData(
  MySpace mySpace, Space revitSpace )
{
  mySpace.surfaces = new List<MySurface>();

  SpatialElementGeometryCalculator calculator
    = new SpatialElementGeometryCalculator( doc );

  SpatialElementGeometryResults results
    = calculator.CalculateSpatialElementGeometry(
      revitSpace );

  Solid spaceSolid = results.GetGeometry();

  foreach( Face face in spaceSolid.Faces )
  {
    //
    //Creating personal Surface component
    //
    MySurface surf = new MySurface();
    surf.area = SQFeetToSQMeter( face.Area );
    mySpace.surfaces.Add( surf );

    //
    //Get the edgePoints of the face
    //
    UV px = null;
    List<XYZ> edgesPoints = new List<XYZ>();

    foreach( EdgeArray a in face.EdgeLoops )
    {
      int nEdges = a.Size;
      List<Curve> curves = new List<Curve>( nEdges );
      XYZ p = null;

      foreach( Edge e in a )
      {
        Curve curve = e.AsCurveFollowingFace( face );
        curves.Add( curve );
      }

      foreach( Curve curve in curves )
      {
        p = curve.get\_EndPoint( 0 );
        edgesPoints.Add( p );
      }
      px = new UV( p.X, p.Y );
    }
    surf.edges = edgesPoints;
    surf.normal = face.ComputeNormal( px );
    Space adjacentSpace = FindAdjacentSpaceFromSurface(
      face, edgesPoints, surf.normal );
    surf.adjSpace = adjacentSpace;
    if( adjacentSpace != null )
    {
      surf.adjSpaceTemperature = adjacentSpace
        .get\_Parameter( "BE\_SpaceTemperature" )
        .AsDouble();
    }
    surf.space = revitSpace;

    //
    //Getting the subfaces to determine the BoundaryElement
    //
    var subfaceList = results.GetBoundaryFaceInfo( face );

    if( subfaceList.Count != 0 )
    {
      foreach( SpatialElementBoundarySubface subface
        in subfaceList )
      {
        ElementId elemid = subface.SpatialBoundaryElement
          .HostElementId;

        ElementId typeid = doc.GetElement(
          subface.SpatialBoundaryElement.HostElementId )
            .GetTypeId();

        // Determine the 'assembly' of the bounding Element
        if( myAssemblies.ContainsKey( typeid ) )
        {
          surf.assembly = myAssemblies[typeid];
        }

        // Retrieve the openings in the considered bounding element
        Element elem = doc.GetElement( elemid );
        if( elem is Wall )
        {
          Wall wall = elem as Wall;
          IList<ElementId> op = wall.FindInserts(
            true, false, true, true );

          foreach( ElementId elid in op )
          {
            Element opening = doc.GetElement( elid );
            ElementId openingTypeId = opening.GetTypeId();

            if( opening is FamilyInstance
              && openingTypes.ContainsKey( openingTypeId ) )
            {
              FamilyInstance fi = opening as FamilyInstance;
              XYZ facingDirection = fi.FacingOrientation.Normalize();
              BoundingBoxXYZ bb = fi.get\_BoundingBox( null );
              XYZ midpoint = ( bb.Min + bb.Max ) / 2;

              XYZ testPoint1 = midpoint + facingDirection
                \* MaxWallThickness;

              XYZ testpoint2 = midpoint + -facingDirection
                \* MaxWallThickness;

              if( revitSpace.IsPointInSpace( testPoint1 )
                || revitSpace.IsPointInSpace( testpoint2 ) )
              {
                Parameter height = fi.get\_Parameter( "Height" );
                if( height == null )
                {
                  height = fi.Symbol.get\_Parameter( "Height" );
                }
                Parameter width = fi.get\_Parameter( "Width" );
                if( width == null )
                {
                  width = fi.Symbol.get\_Parameter( "Width" );
                }
                double area = SQFeetToSQMeter(
                  height.AsDouble() \* width.AsDouble() );

                MyOpeningType myOpeningType
                  = openingTypes[openingTypeId];

                MyOpening myOpening = new MyOpening(
                  area, elid );

                myOpening.type = myOpeningType;
                surf.openings.Add( myOpening );
              }
            }
          }
        }
      }
    }
    else
    {
      //
      // The floor has no subfaces (and no boundary-element)
      // from the geometrycalculator
      // We use ray projection to determine the floortype
      //

      // Find a 3D view to use for the
      // ReferenceIntersector constructor
      FilteredElementCollector coll
        = new FilteredElementCollector( doc );

      Func<View3D, bool> isNotTemplate
        = v3 => !( v3.IsTemplate );

      View3D view3D = coll.OfClass( typeof( View3D ) )
        .Cast<View3D>().First<View3D>( isNotTemplate );

      // set originPoint just above the floor
      BoundingBoxUV bb = face.GetBoundingBox();
      XYZ startPoint = FindPointOnFace( bb, edgesPoints, face );
      XYZ originPoint = startPoint + -( surf.normal ) \* D2mm;

      // set the filter to floor
      ElementClassFilter filter
        = new ElementClassFilter( typeof( Floor ) );

      ReferenceIntersector refint
        = new ReferenceIntersector( filter,
          FindReferenceTarget.Face, view3D );

      if( refint != null )
      {
        ReferenceWithContext refwithcont
          = refint.FindNearest( originPoint,
            surf.normal );

        if( refwithcont != null )
        {
          Element foundElem = doc.GetElement(
            refwithcont.GetReference().ElementId );

          ElementId typeid = foundElem.GetTypeId();
          surf.assembly = myAssemblies[typeid];
          if( surf.assembly.FloorOnGround
            && foundElem.get\_Parameter( "BE\_B U waarde" )
              != null )
          {
            surf.Uequiv = foundElem
              .get\_Parameter( "BE\_B U waarde" )
              .AsDouble();
          }
        }
      }
    }
  }
}

public Space FindAdjacentSpaceFromSurface(
  Face originFace,
  List<XYZ> edgesPoints,
  XYZ normalVector )
{
  Space foundSpace = null;

  // Find a suitable point on the face to 'poke' into the nearby space
  // We then propagate by direction of the normal, and poke 1,35 feet into the wall
  // to see if the testpoint is in another space

  BoundingBoxUV bb = originFace.GetBoundingBox();

  // We determine the point by bounding box or by the points of the face polygon
  XYZ startPoint = FindPointOnFace(
    bb, edgesPoints, originFace );

  XYZ testPoint = startPoint + ( normalVector )
    \* MaxWallThickness;

  foreach( Space space in revitSpaceList )
  {
    if( space.IsPointInSpace( testPoint ) )
    {
      foundSpace = space;
      break;
    }
  }
  return foundSpace;
}

public XYZ FindPointOnFace(
  BoundingBoxUV bb,
  List<XYZ> edgesPoints,
  Face originFace )
{
  UV midPoint = ( bb.Min + bb.Max ) / 2;
  IntersectionResult result;
  XYZ startPoint = new XYZ();
  if( originFace.IsInside( midPoint, out result ) )
  {
    startPoint = originFace.Evaluate( midPoint );
  }
  else
  {
    bool isInside = false;
    while( !isInside )
    {
      // This is not a fool-proof method, but it gets the work done for a simple enough case
      // (it will have to be an intricate face-polygon for this method not to work)
      XYZ sumPoint = new XYZ();
      int start = 0;

      for( int i = start; i < start + 3; i++ )
      {
        sumPoint = sumPoint.Add( edgesPoints[i] );
      }

      startPoint = new XYZ( sumPoint.X / 3,
        sumPoint.Y / 3, sumPoint.Z / 3 );

      var intersect = originFace.Project( startPoint );
      UV startp = intersect.UVPoint;
      if( originFace.IsInside( startp, out result ) )
      {
        isInside = true;
      }
      start++;
    }
  }
  return startPoint;
}
```

Here is the complete
[Revit 2013 project file](/a/rvt/jd_heat_load_calculation_space_adjacency.rvt) containing
the calculation macros and simple sample model:

![Sample model 3D](img/jd_heat_load_calculation_house_3d.png)

Here is a cut through the 3D model:

![Sample model 3D cut](img/jd_heat_load_calculation_house_3d_cut.png)

The first level floor plan looks like this:

![Sample model 2D](img/jd_heat_load_calculation_house_2d.png)

There is no Visual Studio solution, since I am working entirely in the SharpDevelop macro environment available within Revit.
As an inexperienced programmer (just started this year, learning it on my own) I found this made it easier to debug, and after getting it to work as a macro, I found the cleaning-up process to make it work as add-in using Visual Studio express minimal.
So actually the code is inside the project file, embedded as a project-level Macro:

![Sample model macros](img/jd_heat_load_calculation_macros.png)

I am already working on the calculations and the export to XML and PDF, but I cleaned up the project to only reflect the data collection part.
If you are interested in the rest too, I can send the whole project over once it is complete.

Many thanks to Jasper for this fruitful discussion, your research and hard work, and the best of luck to you with the rest of the thesis!