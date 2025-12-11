---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.926512'
original_url: https://thebuildingcoder.typepad.com/blog/1393_idling_dmu_doc_change.html
post_number: '1393'
reading_time_minutes: 10
series: transactions
slug: idling_dmu_doc_change
source_file: 1393_idling_dmu_doc_change.md
tags:
- csharp
- elements
- levels
- parameters
- python
- references
- revit-api
- rooms
- sheets
- transactions
- views
- walls
title: Idling Dmu Doc Change
word_count: 1974
---

### Idling, DMU, DocumentChanged and Need for Regen
Today we have two ever-recurring topics to revisit: a discussion on how to react to a document modification and yet another example of the need for regeneration:
- [Idling and external events, DMU and DocumentChanged](#2)
- [Need for regen in an updater](#3)
#### Idling and external events, DMU and DocumentChanged
We have repeatedly compared the various ways of reacting to Revit model changes, e.g. in the discussion on [DocumentChanged versus Dynamic Model Updater](http://thebuildingcoder.typepad.com/blog/2012/06/documentchanged-versus-dynamic-model-updater.html).
Let's take a fresh look at this, with some important input once again from Arnošt Löbel:
\*\*Question:\*\* Is there any way to quickly retrieve an `ApplicationServices.Application` object when I don't have a Document to work with? I don't see a way to it in the `OnStartUp` event such that I could just cache it somewhere. I also assume using the following is bad:

```
  var app = new Application();
```

Often this is the case when I'm in an Idling event handler but not always. I need the application in order to get at the DocumentSet via Application.Documents.
\*\*Answer:\*\* If you are in an Idling event handler, are you sure that is the right place to be?
I recommend avoiding the Idling event in preference of external events, if you are using it to interact with the outside world.
The only time I think an Idling event is more useful than an external event is when you want it to be called immediately and once only, e.g., because you are in a situation in which you would like to start a transaction to modify the document and cannot do so right away.
Anyway, apart from that: if you indeed are inside an Idling event handler, you do in fact have extremely easy and direct access to the Application object.
The `sender` argument of various events, such as `ApplicationInitialized` and `Idling`, provide the UIApplication object to you straight away:

```
  private void OnIdling(
    object sender,
    IdlingEventArgs e )
  {
    var uiApp = sender as UIApplication;

    // . . .
  }
```

\*\*Response:\*\* Thanks for the info. My code is from previous versions of Revit where external events did not exist (I think) and that's why I was using the Idling event. I'll give these IExternalEventHandler classes a go and test it out.
Meanwhile can you confirm this is an acceptable workflow:
1. Create an IUpdater and set it trigger when a parameter value has changed.
2. When IUpdater.Execute is called and I need to make changes a Document I then invoke ExternalEvent.Create(IExternalEventHandler).
3. Inside of the IExternalEventHandler.Execute method, I create transactions and make more changes to the Document. This will not trigger the IUpdater to fire again.
Last time I remember transactions were not permitted inside of IUpdater.Execute. Please correct me if I'm wrong.
\*\*Answer:\*\* Yes, your workflow looks pretty OK to me.
Yes, you cannot start a new transaction inside the updater Execute method, because that is executed inside the already running transaction that triggered the updater in the first place.
For that usage, I would assume that a one-off Idling event handler is almost exactly equivalent to an external event, so you can use either.
\*\*Answer from Arnošt:\*\* Sorry, Jeremy, I have to chime in.
This workflow actually does not look good to me. Although it is possible to do it that way, it is neither an efficient way nor the safest way. You already hinted the solution in your second statement, though; that one is correct: If you need to make changes (as a reaction to model changes that triggered the updater), you should make them right there in the IUpdater.Execute method. It is possible because the model is already in a transaction and thus is open for changes made by external applications.
In the suggested workflow, the biggest problem I see (beside that it is rather inefficient) is that the new transaction will be separated from the transaction that triggered the updater. That is wrong and dangerous. The danger is in the possibility that the end user undoes one transaction but leaves the other one in which the updater run. That would leave the document in a state where changes were made to which you wanted to react, but the reaction (in the second transaction) is removed. From the point of view of the application the model would thus be invalid and corrupt.
\*\*Response:\*\* I agree there's danger although very limited. Let me explain what I'm trying to do and maybe you can direct me to the most efficient process.
I have 4 generic annotation families. I need to track all instances of these families, by their corresponding View, in our external database. Our database is not tracking individual instances of the generic annotations, just overall counts, by part number, per view and cannot be changed. So the events I need to capture from Revit are:
\* When these annotation instances are created
\* When these annotation instances are modified
\* When these annotation instances are deleted
\* Transaction is undone/redone
When any of these events occur I need launch a synchronization event which does the following:
\* Delete the existing bill of materials (model group) in the corresponding View
\* Validate the parameters of generic annotation instance in that View (if they are invalid I need to delete those instances)
\* Upload the new data to the external database
\* Recreate the bill of materials table (model group)
I'm using a model group with detail lines and text for the table. To be efficient, I only want to execute a synchronization event once per View.
The IUpdaters are good, however I haven't found a way for them to tell me what View the deleted element ids came from. I'm starting to question whether I should just do everything in the DocumentSaved events rather than bothering with the IUpdaters. I'm open to suggestions if you feel otherwise.
\*\*Answer:\*\* I agree that you are up to a challenge. Tracking deleted elements is never easy because one cannot get any properties of the elements from Revit. It is not because Revit does not want to give them; we do not have them either in the internal model database. Or rather, we do have them still in some form (for eventual undoing), but it is dangerous accessing them, for they have been disconnected from the rest of the model. This is an internal implementation detail, of course; the point is that one cannot get any info for deleted elements other than the elements' Ids.
You have two choices, the way I see it. Either you have to store more info in your database, or you try a different approach altogether. You suggested performing a synchronization only once when saving a (modified) file. If that is a viable approach for your needs, I would recommend it over the Updater approach.
There is one other thing also suggesting against the use of an updater, and it is also quite important one. You mentioned that: "I need to track all instances of these families, by their corresponding View, in our external database". If that is what I think it is, you should not use updaters at all for the purpose of syncing with the database. It is because updaters are called for new changes only; they are never called for undone or redone transactions, which is a huge problem for external tracking. It means that - in your case - if any creation of the annotations are undone, your database will not have correct records. Therefore, when there is a need for a sync between elements in a Revit model and external databases, the programmer must use the DocumentChanged event. The rule is as follow:
- If an app needs to react to model changes by making further changes to the same modes, then an Updater is the way to go.
- If an app needs to react to model changes by making changes to external data, then the DocumentChanged must be used.
However, the DocumentChanged event suffers from the same problem: one cannot get any data about already deleted elements. Sorry about that.
Good luck!
#### Need for Regen in an Updater
We have already looked at a large number of examples of how important it is to be aware of the Revit model modifications your add-in causes.
If you change anything in the model, beware of querying the model afterwards without forcing a regeneration, or you will receive stale and possibly conflicting data.
The Building Coder dedicates a whole topic group to the [need to regenerate](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.33).
Now we have another entry to add to it, prompted
by [Amr](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/3289713)'s
Revit API discussion thread asking why on earth
the [IUpdater is triggered and run with no errors but model changes are undone](http://forums.autodesk.com/t5/revit-api/iupdater-is-triggered-and-run-with-no-errors-but-model-changes/m-p/5980294),
and answered by Amr himself:
\*\*Question:\*\* IUpdater is triggered and run with no errors but model changes are undone for no obvious reason.
Heeelp, I'm pulling my hair out since morning.
I'm creating an application that searches the document for rooms and then uses the room boundary to create a new floor.
I created an IUpdater so that if the user moves a wall it changes the rooms boundary and therefore uses the new room boundary to create a new floor and delete the old one.
What happens, however, is this: when I move a wall, the updater is triggered and the code runs with no errors, but then I find the wall back in its original location.
I really don't know why and I can't connect any dots!!!
Here's my code for the IUpdater Execute method:

```
  try
  {
    foreach (ElementId elemId in data.GetModifiedElementIds())
    {
      List<Floor> floorsByRoomsList = new List<Floor>();
      Room r = doc.GetElement(elemId) as Room;
      if (r != null)
      {
        SpatialElementBoundaryOptions sb = new SpatialElementBoundaryOptions();
        sb.SpatialElementBoundaryLocation = SpatialElementBoundaryLocation.CoreBoundary;
        IList<IList<BoundarySegment>> loops = r.GetBoundarySegments(sb);

        List<Floor> floorsInDoc = new List<Floor>();
        floorsInDoc = GetAllFloors(doc);

        foreach(Floor existFloor in floorsInDoc)
        {
          string floorMark = existFloor.get_Parameter(BuiltInParameter.ALL_MODEL_MARK).AsString();
          if (floorMark == r.Id.ToString())
          {
            Floor newFloor = null;
            Opening newOpening = null;
            if (loops.Count == 1)
            {
              #region the new floor properties
              CurveArray floorprofile = Data.ObtainCurveArray(r);
              FloorType fl = existFloor.FloorType;
              Level level = r.Level;
              bool bl;
              if (existFloor.GetAnalyticalModel() == null)
                bl = false;
              else
                bl = true;
              #endregion
              newFloor = doc.Create.NewFloor(floorprofile, fl, level, bl);
              floorsByRoomsList.Add(newFloor);
            }
            else if (loops.Count > 1)
            {
              #region the new floor & opening properties
              CurveArray floorprofile = Data.ObtainCurveArray(r);
              FloorType fl = existFloor.FloorType;
              Level level = r.Level;
              bool bl;
              if (existFloor.GetAnalyticalModel() == null)
                bl = false;
              else
                bl = true;

              CurveArray openingProfile = Data.ObtainOpeningCurveArray(r);
              bool blO = true;
              #endregion
              newFloor = doc.Create.NewFloor(floorprofile, fl, level, bl);
              floorsByRoomsList.Add(newFloor);
              newOpening = doc.Create.NewOpening(newFloor, openingProfile, blO);
            }
          } // if the floor is not located inside room and its name is not equal to room id
          else
          {
            continue;
          }
        }
      } // if the modified element is not a room
      else
      {
        continue;
      }
      foreach (Floor fl in floorsByRoomsList)
        fl.get_Parameter(BuiltInParameter.ALL_MODEL_MARK).Set(r.Id.ToString());
    }
  }
```

\*\*Answer:\*\* After various trials and errors I found out that the problem is that Revit doesn't update and reflect the changes instantly, so adding the `NewOpening` method right after `NewFloor` didn't work because somehow the floor wasn't yet written to the model. You have to write it manually by adding a call to `doc.Regenerate` between the calls to `NewFloor` and `NewOpening`. This will reflect the latest changes to the model and therefore the `NewOpening` method will work nicely.
Thank you very much, Amr, for discovering this the hard way and sharing it with us.