---
post_number: "1193"
title: "Setting the Phase of a View"
slug: "set_view_phase"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'levels', 'parameters', 'revit-api', 'rooms', 'views']
source_file: "1193_set_view_phase.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1193_set_view_phase.html"
---

### Setting the Phase of a View

Here is a quick summary of one of the many issues being discussed on the
[Revit API forum](http://forums.autodesk.com/t5/Revit-API/bd-p/160),
on
[changing the phase of a view](http://forums.autodesk.com/t5/Revit-API/change-phase-of-a-view/m-p/5206419),
with a quick and happy conclusive result:

**Question:** Is it possible to change the phase of a view? Something like view.phase?

I've tried with the created phase but it doesn't work.

Thank for your help!

**Answer:** Does this old discussion on
[creating a room on a level in a phase](http://thebuildingcoder.typepad.com/blog/2009/03/create-room-on-level-in-phase.html) help?

Please look specifically at the reply to most recent
[comment](http://thebuildingcoder.typepad.com/blog/2009/03/create-room-on-level-in-phase.html#comment-6a00e553e16897883301a511f1f07a970c) by Jared.

**Response:** I tried what you explain on the blog but I don't know why it doesn't work:

```csharp
  View active = commandData.Application
    .ActiveUIDocument.ActiveGraphicalView;
  foreach( Phase ii in phase )
  {
    Parameter p = active.get\_Parameter(
      BuiltInParameter.VIEW\_PHASE );
    ElementId iiId = ii.Id;
    p.SetValueString( iiId );
  }
```

**Answer:** How is your variable 'phase' defined?

Have you checked what storage type the built-in parameter VIEW\_PHASE is expecting?

You can use RevitLookup to explore it.

Whatever storage type it is expecting, I am pretty sure that you cannot set it using SetValueString.

I would expect the storage type to be ElementId, in which case you need to set it using the element id directly, e.g. like this:

```csharp
p.Set( iiId );
```

I hope this helps.

**Response:** Yeah it works!

phase is defined like this: `PhaseArray phase = doc.Phases;`

Here is the working code to set it:

```csharp
  ElementId iiId = ii.Id;

  Parameter p = active.get\_Parameter(
    BuiltInParameter.VIEW\_PHASE );

  p.Set( iiId );
```

with:

- ii – a Phase
- active – a view

Thank a lot!