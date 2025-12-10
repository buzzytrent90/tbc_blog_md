---
post_number: "0603"
title: "No Multithreading in Revit"
slug: "no_multithreading"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'revit-api', 'windows']
source_file: "0603_no_multithreading.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0603_no_multithreading.html"
---

### No Multithreading in Revit

I repeatedly hear from developers asking whether multithreading can be used in conjunction with the Revit API.
The short answer is **no**.

To our surprise, we just noticed that the Truss SDK sample does demonstrate a use of multithreading.
That is an oversight and you should not rely on this working properly.

Here is the considered statement of Scott Conover, Software Development Manager of the Revit API, on the current state of multithreading and Revit:

"Revit's internals make use of multiprocessing in only a few select isolated locations.
None of these locations currently encompass the code in the Revit API, or any part of it.
Thus Autodesk does not recommend making any calls to the Revit API from within simultaneously executing parallel threads.
It may be that some part of the Revit API is isolated enough to be able to execute successfully from within such threading code in a test environment; this should not be taken to be a guarantee that the same source code will function for any model or situation, or that a future change in Revit will not cause this code to cease to function."

I hope this clarifies the situation.

#### Truss SDK Sample Enhancement

The Truss SDK sample contains the following code and comments:
```csharp
  // get all the beam types
  // because GetBeamTypes() takes a long
  // time, so call it in a new thread

  Thread newThread = new Thread(
    new ThreadStart( GetBeamTypes ) );

  newThread.Start();
```

Here, GetBeamTypes is using the Revit API methods FilteredElementCollector and parameter access methods.

As said, this is not supported; Revit API access is required to be single threaded.
It is an interesting side note that it apparently works, but this cannot be relied on and this code should definitely not be reused.

Actually, the sample would be faster if it made use of more of the FilteredElementCollector capabilities.
The implementation of this sample predated the introduction of filtered element collectors in Revit 2011:
```csharp
  m\_beamTypes = from elem in
    new FilteredElementCollector( doc )
      .OfClass(typeof(FamilySymbol))
      .ToElements()
    let type = elem as FamilySymbol
    where type != null
      && type.Category != null
      && type.Category.Name == "Structural Framing"
    select type;
```
A better implementation might be:
```csharp
  m\_beamTypes
    = from elem in new FilteredElementCollector(
          m\_activeDocument.Document )
        .OfClass( typeof( FamilySymbol ) )
        .OfCategory( BuiltInCategory.OST\_StructuralFraming )
      let type = elem as FamilySymbol
      select type;
```

This uses a quick category filter and avoids calling ToElements, which is unnecessary, as the filtered element collector itself is already an IEnumerable.

Other examples of using multithreading in perfectly valid ways in conjunction with the Revit API are provided by the APIAppStartup splash window and the AnalysisVisualizationFramework MultithreadedCalculation Revit SDK samples, where the separate threads do not make calls into Revit.