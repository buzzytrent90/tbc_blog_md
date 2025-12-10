---
post_number: "0826"
title: "DivideParts in F#"
slug: "divide_parts_fsharp"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'references', 'revit-api', 'selection', 'transactions', 'views', 'walls']
source_file: "0826_divide_parts_fsharp.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0826_divide_parts_fsharp.html"
---

﻿

### DivideParts in F#

My internet connection died last night.
After worrying and testing quite a bit last night and this morning, I laid a temporary cable from my neighbour's router.
Shortly after I finished, my own connection came back up again, after about 14 hours downtime.
[Murphy's law](http://en.wikipedia.org/wiki/Murphy%27s_law) remains valid, as always.
Very reassuring :-)
There is lots of building work going on nearby, so probably they broke it and fixed it with no explanations given.
Anyway, I am back to normal now and
[keeping my fingers crossed](http://en.wikipedia.org/wiki/Crossed_fingers).

Last week, I presented a C# sample by Piotr Zurek of
[CADPRO Systems Ltd](http://www.cadpro.co.nz/) exercising the
[PartUtils class and its DivideParts method](http://thebuildingcoder.typepad.com/blog/2012/09/parts-assemblies-partutils-and-divideparts.html).

As an experiment and for comparison purposes, Piotr decided to port this to F# as well.
In his own words:

As an exercise, I decided to port this example to F# to see how it works and whether the F# syntax offers any benefits over C#.

#### Lessons Learned

Here are some of the lessons learned:

##### Let Not Use

Initially, the F# version of the command ran fine only the first time I launched it, and failed in the call to PickObject with an InvalidOperationException saying "The managed object is not valid" on the second call.

After isolating the call to PickObject, I changed the following four initialisation statements:
```csharp
  use uiApp = commandData.Application
  use app  = uiApp.Application
  use uiDoc = uiApp.ActiveUIDocument
  use doc  = uiDoc.Document
```

It turns out that this is wrong, since the F# 'use' statement is just like 'using' in C#.

So when these objects fall out of scope, Dispose will be called on them.
Since they belong to Revit, they don't like being disposed of by external code.
Changing 'use' to 'let' fixed that problem:
```csharp
  let uiApp = commandData.Application
  let app  = uiApp.Application
  let uiDoc = uiApp.ActiveUIDocument
  let doc  = uiDoc.Document
```

Thanks to Kean Walmsley for this explanation.

##### Pattern Matching Versus :?

At one point, I made a little bit of use of F# pattern matching to implement the WallSelectionFilter AllowElement method like this:
```csharp
      member x.AllowElement(element) =
        match element with
        | :? Wall as element -> true
        | \_ -> false
```

However, on further consideration, it is clearer and more succinct to use the F# ':?' operator, which is an equivalent of 'is' in C#, like this:
```csharp
type WallSelectionFilter() =
  class
    interface ISelectionFilter with
      member x.AllowElement(element) = element :? Wall
      member x.AllowReference(reference, xyz) = false
  end
```

##### Implementing an Exception Handler

I initially removed the 'try-catch' exception handler around the call to PickObject to handle a user cancelling the pick operation.
On consideration, I found that it translates to 'try-with' in F#.
The piping symbol '|' means pretty much 'data received from the last operation':
```csharp
  let reference =
    try
      sel.PickObject(
        ObjectType.Element,
        new WallSelectionFilter(),
        "Select a wall to split into panels")
    with
      | :? Autodesk.Revit.Exceptions.
        OperationCanceledException -> null
```

Exiting the mainline execution requires an if-else statement as well: if reference is null, return Result.Canceled.
The best solution is probably more
[functional](http://en.wikipedia.org/wiki/Functional_programming),
though.

#### Final Implementation

Here is the final complete F# implementation of this external command:
```csharp
namespace FSharpPanelBuilder

open System
open System.Collections.Generic

open Autodesk.Revit
open Autodesk.Revit.UI
open Autodesk.Revit.Attributes
open Autodesk.Revit.DB
open Autodesk.Revit.UI.Selection

type WallSelectionFilter() =
  class
    interface ISelectionFilter with
      member x.AllowElement(element) = element :? Wall
      member x.AllowReference(reference, xyz) = false
  end

[<Transaction(TransactionMode.Manual)>]
type Command() =
  class
    interface IExternalCommand with
      member this.Execute(commandData,
                          message : string byref,
                          elements ) =

        try
          let uiApp = commandData.Application
          let app  = uiApp.Application
          let uiDoc = uiApp.ActiveUIDocument
          let doc  = uiDoc.Document
          let sel = uiDoc.Selection

          let reference =
            try
              sel.PickObject(
                ObjectType.Element,
                new WallSelectionFilter(),
                "Select a wall to split into panels")
            with
              | :? Autodesk.Revit.Exceptions.
                OperationCanceledException -> null

          if reference = null then
            TaskDialog.Show(
              "F# Panel Builder",
              "Operation was canceled") |> ignore
            Autodesk.Revit.UI.Result.Cancelled
          else
            let wall =
              doc.GetElement(reference.ElementId) :?> Wall

            let locationCurve =
              wall.Location :?> LocationCurve

            let line = locationCurve.Curve :?> Line

            use transaction = new Transaction(doc)

            let status = transaction.Start("Building panels")

            let wallList = new List<ElementId>(1)
            wallList.Add(reference.ElementId)

            PartUtils.CreateParts(doc, wallList)
            doc.Regenerate()

            let parts =
              PartUtils.GetAssociatedParts(
                doc, wall.Id, false, false)

            let divisions = 15
            let origin = line.Origin

            let delta =
              line.Direction.Multiply(
                line.Length / (float)divisions)

            let shiftDelta =
              Transform.get\_Translation(delta)

            let rotation =
              Transform.get\_Rotation(
                origin, XYZ.BasisZ, 0.5 \* Math.PI)

            let wallWidthVector =
              rotation.OfVector(
                line.Direction.Multiply(2. \* wall.Width))

            let mutable intersectionLine =
              app.Create.NewLineBound(
                origin + wallWidthVector,
                origin - wallWidthVector)

            let curveArray = new List<Curve>()

            for i = 1 to divisions do
              intersectionLine <-
                intersectionLine.get\_Transformed(
                  shiftDelta) :?> Line

              curveArray.Add(intersectionLine)

            let divisionSketchPlane =
              doc.Create.NewSketchPlane(
                new Plane(XYZ.BasisZ, line.Origin))

            let intersectionElementsIds =
              new List<ElementId>()

            let partMaker =
              PartUtils.DivideParts(
                doc, parts, intersectionElementsIds,
                curveArray, divisionSketchPlane.Id)

            doc.ActiveView.PartsVisibility <-
              PartsVisibility.ShowPartsOnly

            transaction.Commit() |> ignore

            Result.Succeeded

        with ex ->
          TaskDialog.Show(
            "F# Panel Builder",
            ex.Message) |> ignore

          Result.Failed

  end
```

The code is a straight translation from C# into F#, almost without making any use of F# specific concepts.

Later on, it would be nice to rewrite it to be a bit more
[functional](http://en.wikipedia.org/wiki/Functional_programming).

I really like how expressive and concise F# is, but switching between F# and C# causes me horrible headaches. :-)

#### Git and GitHub Source Code Management

I placed the entire project on
[GitHub](https://github.com):

- <https://github.com/pzurek/FSharpPanelBuilder>.

GitHub a source code collaboration platform based on the version control system git, a fast, efficient, distributed system for collaborative software development.
It enables you to easily fork, send pull requests and manage all your public and private git repositories.

I love it.
Here at CADPRO I don't get too much chance to use it collaboratively, since I am usually the only person working on a project.

When playing with open source projects, that's another story and git is a brilliant tool for it.
Some people claim that is it hard and too complicated, but for me that was the first distributed source control system I used and I didn't find it too hard to grasp.

The thing that makes Git amazing is GitHub.
They just put a nice desktop and web interface on it and make it nice and easy to use.

Besides looking at the
[main project page](https://github.com/pzurek/FSharpPanelBuilder),
you can
[download the whole project as a zip file](https://github.com/pzurek/FSharpPanelBuilder/zipball/master).

GitHub also provides a little tool that help with publishing source code on the Internet.
Gist.github.com allows you to create snippets of code and then embed them in other websites.

As an example, I pasted the
[command.fs file from the F# sample](https://gist.github.com/3702963).

Many thanks to Piotr for all this information, research and comparison!

#### Retrieving Line Styles

Mikako Harada published sample code showing how to
[retrieve all line styles](http://adndevblog.typepad.com/aec/2012/09/retrieving-a-list-of-line-styles.html) through
the CurveElement GetLineStyleIds property.

If you don't have a curve element handy to invoke the property on, you can simply create a temporary one, for instance a detail line in the currently active view.
Created in a separate transaction which is rolled back afterwards, it enables access to the line styles with no effect on the database.