---
post_number: "1389"
title: "Wall Opening Profiles"
slug: "wall_opening_profiles"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'geometry', 'levels', 'parameters', 'python', 'references', 'revit-api', 'selection', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1389_wall_opening_profiles.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1389_wall_opening_profiles.html"
---

### Wall Opening Profiles and Happy Holidays!
Last week, we discussed how to [retrieve wall openings](http://thebuildingcoder.typepad.com/blog/2015/12/retrieving-wall-openings-and-sorting-points.html).
Let's take another fresh look at that, and also at:
- [A walk over Uetliberg](#2)
- [Using FindInserts to retrieve wall openings](#3)
- [CmdGetWallOpeningProfiles](#4)
- [Relationships, independence and consumerism](#5)
#### A Walk over Uetliberg
First, here are some snapshots from an unseasonably warm winter [walk over the Uetliberg](https://www.flickr.com/gp/jeremytammik/i819G8) at Zurich last weekend:
[![Uetliberg Afternoon Walk](https://farm1.staticflickr.com/696/23838892816_e6dea84f27_n.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157662537072231 "Uetliberg Afternoon Walk")
#### Using FindInserts to Retrieve Wall Openings
Last week, we solved the task of retrieving wall opening start and end points by shooting a ray through the wall parallel to the wall location line and asking the ReferenceIntersector for all the wall opening side faces that it hits.
That works fine.
However, in the initial query on [getting wall openings](http://forums.autodesk.com/t5/revit-api/get-wall-openings/m-p/5953513), Eirik implies that `FindInserts` cannot be used because 'it's not given that anything is inserted in the opening'.
That is a false assumption, as proven by the later answer by Scott Wilson, who very succinctly suggests:
> What about just using `Wall.FindInserts(true, false, false, false)`
> Then check the category of each returned element for `BuiltInCategory.OST_Doors`, `BuiltInCategory.OST_Windows` and, for rectangular openings, check for element Type: `Opening`.
> To get the opening faces, loop through all perpendicular faces of the wall using `Wall.GetGeneratingElementIds` to match against your list of openings.
> `FindInserts` used with those parameters will find the windows whether or not anything is 'inserted' in the openings whatever that means.
> I use this method quite frequently.
> You could also just skip the FindInserts Step and just check each face of the wall for those created by a door / window instance or opening element.
> I whipped up a Revit 2015 command as a demo: [GetOpeningProfiles.zip](https://dl.dropboxusercontent.com/u/74965361/GetOpeningProfiles.zip).
#### CmdGetWallOpeningProfiles
Seeing as I already implemented a command for the ray tracing approach, I went ahead and added this alternative of Scott's as well as another external
command [CmdWallOpeningProfiles](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdWallOpeningProfiles.cs) to
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).
I tested both Scott's original Revit 2015 implementation and its new incarnation in Revit 2016.
Scott's implementation includes several noteworthy little details, e.g.:
- Drawing model lines to display the face edges and using temporary hide to display them.
- The proper clean way to convert a built-in category enumeration value to the document category element id:

```
    ElementId catDoorsId = cats.get_Item(
      BuiltInCategory.OST_Doors ).Id;
```

The code to retrieve the wall geometry and extract all faces from all solids that belong to a specific opening is implemented by the helper method `GetWallOpeningPlanarFaces`:

```
  ///
  /// Retrieve all planar faces belonging to the
  /// specified opening in the given wall.
  ///
  static List<PlanarFace> GetWallOpeningPlanarFaces(
    Wall wall,
    ElementId openingId )
  {
    List<PlanarFace> faceList = new List<PlanarFace>();

    List<Solid> solidList = new List<Solid>();

    Options geomOptions = wall.Document.Application.Create.NewGeometryOptions();

    if( geomOptions != null )
    {
      //geomOptions.ComputeReferences = true; // expensive, avoid if not needed
      //geomOptions.DetailLevel = ViewDetailLevel.Fine;
      //geomOptions.IncludeNonVisibleObjects = false;

      GeometryElement geoElem = wall.get_Geometry( geomOptions );

      if( geoElem != null )
      {
        foreach( GeometryObject geomObj in geoElem )
        {
          if( geomObj is Solid )
          {
            solidList.Add( geomObj as Solid );
          }
        }
      }
    }

    foreach( Solid solid in solidList )
    {
      foreach( Face face in solid.Faces )
      {
        if( face is PlanarFace )
        {
          if( wall.GetGeneratingElementIds( face )
            .Any( x => x == openingId ) )
          {
            faceList.Add( face as PlanarFace );
          }
        }
      }
    }
    return faceList;
  }
```

This method is called in a loop iterating over the opening and insertion element ids returned by the `FindInserts` method, so it sets up the geometry options, retrieves and iterates over the entire wall solid anew from scratch for each opening.
You would obviously optimise this a bit in a real-life application.
That method is driven like this by the external command Execute method mainline:

```
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;
    Result commandResult = Result.Succeeded;
    Categories cats = doc.Settings.Categories;

    ElementId catDoorsId = cats.get_Item(
      BuiltInCategory.OST_Doors ).Id;

    ElementId catWindowsId = cats.get_Item(
      BuiltInCategory.OST_Windows ).Id;

    try
    {
      List<ElementId> selectedIds = uidoc.Selection
        .GetElementIds().ToList();

      using( Transaction trans = new Transaction( doc ) )
      {
        trans.Start( "Cmd: GetOpeningProfiles" );

        List<ElementId> newIds = new List<ElementId>();

        foreach( ElementId selectedId in selectedIds )
        {
          Wall wall = doc.GetElement( selectedId ) as Wall;

          if( wall != null )
          {
            List<PlanarFace> faceList = new List<PlanarFace>();

            List<ElementId> insertIds = wall.FindInserts(
              true, false, false, false ).ToList();

            foreach( ElementId insertId in insertIds )
            {
              Element elem = doc.GetElement( insertId );

              if( elem is FamilyInstance )
              {
                FamilyInstance inst = elem as FamilyInstance;

                CategoryType catType = inst.Category
                  .CategoryType;

                Category cat = inst.Category;

                if( catType == CategoryType.Model
                  && ( cat.Id == catDoorsId
                    || cat.Id == catWindowsId ) )
                {
                  faceList.AddRange(
                    GetWallOpeningPlanarFaces(
                      wall, insertId ) );
                }
              }
              else if( elem is Opening )
              {
                faceList.AddRange(
                  GetWallOpeningPlanarFaces(
                    wall, insertId ) );
              }
            }

            foreach( PlanarFace face in faceList )
            {
              Plane facePlane = new Plane(
                face.ComputeNormal( UV.Zero ),
                  face.Origin );

              SketchPlane sketchPlane
                = SketchPlane.Create( doc, facePlane );

              foreach( CurveLoop curveLoop in
                face.GetEdgesAsCurveLoops() )
              {
                foreach( Curve curve in curveLoop )
                {
                  ModelCurve modelCurve = doc.Create
                    .NewModelCurve( curve, sketchPlane );

                  newIds.Add( modelCurve.Id );
                }
              }
            }
          }
        }

        if( newIds.Count > 0 )
        {
          View activeView = uidoc.ActiveGraphicalView;
          activeView.IsolateElementsTemporary( newIds );
        }
        trans.Commit();
      }
    }

    #region Exception Handling

    catch( Autodesk.Revit.Exceptions
      .ExternalApplicationException e )
    {
      message = e.Message;
      Debug.WriteLine(
        "Exception Encountered (Application)\n"
        + e.Message + "\nStack Trace: "
        + e.StackTrace );

      commandResult = Result.Failed;
    }
    catch( Autodesk.Revit.Exceptions
      .OperationCanceledException e )
    {
      Debug.WriteLine( "Operation cancelled. "
        + e.Message );

      message = "Operation cancelled.";

      commandResult = Result.Succeeded;
    }
    catch( Exception e )
    {
      message = e.Message;
      Debug.WriteLine(
        "Exception Encountered (General)\n"
        + e.Message + "\nStack Trace: "
        + e.StackTrace );

      commandResult = Result.Failed;
    }

    #endregion

    return commandResult;
  }
```

Here is the result of using it on a wall with doors, a window and two openings in Revit 2015:
![Wall openings](img/wall_openings_08.png)
The result looks like this:
![Wall openings](img/wall_openings_09.png)
I also ran it on the same three walls as the ray tracing version:
![Wall openings](img/wall_openings_05.png)
The command executes, generates the model lines representing the opening face edges, and temporarily hides everything else:
![Wall openings](img/wall_openings_10.png)
As said, this code is provided by the external
command [CmdWallOpeningProfiles](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdWallOpeningProfiles.cs) in
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples),
and the version presented above is
[release 2016.0.126.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.126.1).
#### An Article on Relationship and Consumerism
As a fitting ending statement for this year, I present my translation from German of a short and succinct article by [Vivian Dittmar](http://viviandittmar.net) that she published in June 2015.
![Vivian Dittmar](img/vifian_dittmar.jpg)
The original German text is entitled \*Warum das Streben nach Unabhängigkeit unsere Beziehungsfähigkeit untergräbt\*.
Here are the PDF versions of the [German original](zip/vifian_dittmar_relationship_de.pdf) and my [English translation](zip/vifian_dittmar_relationship_en.pdf).
I'll also add the full English text right here, since it is not long:
#### Why the Pursuit of Independence Undermines our Relationship Abilities
Most relationship guides proclaim nonsense, stating that only those who are self-sufficient and independent can truly love. These authors maintain that independence is a prerequisite for a healthy partnership. Is independence really the key to happiness in matters of love? The speaker, seminar leader and author Vivian Dittmar speaks about the myth of independence.
Should true love let go of the partner? Are only self-sufficient people capable of loving? Does needing one another invariably lead to the hell of co-dependency?
Independence – sounds great, and fits perfectly with our current collective fixation on individualisation.
From an early age we are taught that each of us is responsible for his own happiness and that dependence on others is something for weaklings who prefer to remain in their own secure little nest. For many, dependence equals vulnerability. Dependence also means that others have power over you. Therefore, dependency is an absolute no-go.
This belief coincides beautifully with the objectives of capitalism. Whoever wants to be independent, will mainly focus on earning sufficient money to take care of themselves.
From this perspective, all too large investments in relationships with other people become a highly dubious matter and are therefore avoided.
#### Consumerism cannot Satisfy the Desire for Relationship
We try to compensate the deficits that we inevitably develop as social beings based on this strategy by satisfaction in the form of consumerism. Advertising helps associate our yearning for relationship with the corresponding products: the deodorant, the car, the drink, the outfit, assure that our desire for togetherness and belonging is finally answered.
These substitute satisfactions frequently lead to addiction, requiring ever-higher doses in ever-shorter intervals to cause the desired effect. However, the primordial human desire for relationship, contact, and true togetherness is not fulfilled.
This ‘real togetherness’ can only happen if we allow ourselves to need each other. I say ‘allow’ because today, more than ever before, it has become our personal decision whether to be involved with someone or not.
#### Isolation as a Symptom of Apparent Independence
People have always shaped and maintained relationships because they were dependent on each other. Today, to an almost frightening degree, we can choose only to need people we never see, without any personal relationship at all. I refer to the employees of the companies that manufacture all our products and try to fulfil all imaginable needs with their services. On one hand, this has great advantages: we can choose the relationships we maintain to an unprecedented degree. Wow, that's really fantastic! However, it also has serious drawbacks and brings a whole new set of challenges.
One is the question of how to create, maintain and grow relations when we are able to cope alone in all areas. Unlike in traditional cultures, relationships are no longer automatic side effects of daily life. In western nations, the trend towards increasing social isolation and loneliness in the midst of society is a symptom of this development.
￼
#### We need one another!
Another side effect is that the old rules that governed our ancestors’ relationship structure over centuries or even millennia suddenly no longer apply. The clearest example of this development is the altered power imbalance between men and women. With increasing emancipation a growing number of women became aware of their independence, decided against classical roles and thus rejected this imbalance.
Today it is obvious for most couples to pursue a relationship between equals. Unfortunately, this does not mean that they know how this can be achieved. If things go badly, both strive to fulfil our cultural ideal of independence and only notice too late that this does not guarantee a successful relationship. In these situations, fighting for power or increasing alienation is inevitable, because neither party is honest and open with their true needs.
We lack the knowledge about and practise in dealing constructively with conflicting needs. We lack role models showing how togetherness on equal terms can succeed, without pretending a perfect world, in which we either always agree or slip into infighting.
A prerequisite for the development of these skills is to start from a clean slate and admit the simple fact: We need each other! Although I can meditate myself out of my human condition or buy all possible satisfaction with just a modest fortune; we humans are and remain social beings needing other people to be happy.
We need to stop avoiding facing this fact before we can begin developing a constructive approach to it. Only then can we completely bid farewell both to the power paradigm, in which all needs in a relationship always represent power potential, and to the consumerism paradigm, in which needs represent buying potential. Then we can move into a new relationship paradigm, in which needs have a completely different meaning.
They represent neither power nor consumerism, but relationship potential.
#### Mature Relationship: Balancing Autonomy and Dependence
The idea that adulthood means not needing anyone else is a fundamental misconception of our culture.
In fact, it is the adolescent who dreams of giving the finger to the world and disappearing into the sunset on a motorbike. Real adulthood means being aware of my self-reliance as well as my dependence and having learned to maintain a healthy balance between them. Specifically, this means that a mature relationship always includes both: needing and not needing, dependence and autonomy.
Only this interplay enables respectful, dignified and thus also fulfilling togetherness. In this respect, there is some truth in the myth of independence as the key to happiness.
But, as so often, that is only half the story.
I hope you enjoyed this little excursion into a totally different realm and send you the best seasonal greetings, wishing a Very Merry Xmas and a Happy New Year to all!