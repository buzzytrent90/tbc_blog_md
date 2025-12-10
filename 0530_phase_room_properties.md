---
post_number: "0530"
title: "Phase Dependent Room Properties"
slug: "phase_room_properties"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'family', 'revit-api', 'rooms', 'windows']
source_file: "0530_phase_room_properties.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0530_phase_room_properties.html"
---

### Phase Dependent Room Properties

Here is a recent query from a developer on the family instance FromRoom and ToRoom properties. He asked:

**Question:** I am unable to retrieve the room associated with certain door and window instances.

I attached a Revit model where the family instance room properties return "null", even though the doors are associated with a room.

Why do the ToRoom and FromRoom family instance properties return "null" for these doors?

**Answer:** I analysed your model and see no issues with the functionality of the inconsistent property values.
Here is the explanation:

It seems that the cause of this problem is phasing.
Your model defines three phases: existing, new construction, and optional.
No rooms are defined in the optional phase.

The FamilyInstance.FromRoom property returns the room from which the door is opened at the last phase of the project.
I tried calling the FamilyInstance.FromRoom[Phase] method with a proper phase and it works well.

**Response:** I tried using FamilyInstance.FromRoom[Phase] as you suggested, and I am still unable to get the room.
Can you kindly send me the sample code which you used?

**Answer:** Please try the code snippet below.

As said, your model defines the following three phases: Existing, New Construction, and Optional-in order.

The rooms' phasing is "New Construction".

The doors' phase created is "New Construction" and the phase demolished is "None".

The result of running the following code is:

- Input phase "Existing" throws an exception, because the door does not exist in the given phase.- Input phase "New Construction" returns the rooms.- Input phase "Optional" returns null, because no rooms are defined in this phase.

```csharp
  foreach( Phase phase in doc.Phases )
  {
    try
    {
      Room froom = door.get\_FromRoom(phase);
      Room troom = door.get\_ToRoom(phase);
    }
    catch( System.Exception ex )
    {
      ...
    }
  }
```

**Response:** It is working and I am able to retrieve the room now.

Just one more thing to be aware of... Watch your phases!