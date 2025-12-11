---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.638702'
original_url: https://thebuildingcoder.typepad.com/blog/0809_iterate_then_delete.html
post_number: 0809
reading_time_minutes: 3
series: general
slug: iterate_then_delete
source_file: 0809_iterate_then_delete.htm
tags:
- elements
- family
- revit-api
- rooms
- transactions
title: Do Not Delete During Iteration
word_count: 635
---

### Do Not Delete During Iteration

Here is a type of question that crops up much too regularly, and that I am hoping this little example will help you avoid.

When you are iterating over a collection, deleting the elements in the collection at the same time can wreak havoc in many ways.

Here is an example:

**Question:** Since my last plug-in was such a huge success here in the office I have been tasked with writing another. However, I've run into an issue. When I run the plug-in Revit crashes with no error message or anything. Revit simply disappears. How do I even begin to debug an issue like this?

The plug-in that I have created counts light fixtures and the room the light fixture is in and writes that data to an excel file so that I can use that data in an eQUEST energy model. If the light fixture is found in a room, and only 1 room, then the light fixture is deleted from the project. This is so I can print out the lighting plans and have a visual representation of the lights that I need to count by hand.

I have tried adding a counter to the code to see where it crashes, however it does not crash on the same light fixture every time. On the sample project I have attached it crashes in the 50s. On a real project it crashes around 640 fixtures. Additionally I have commented out segments of code to fixture out where the issue is. If I comment out the "element delete code", then the plug in runs all the way and counts the fixtures to excel, but does not create the visual representation of "problem fixtures". I've tried setting the transaction mode to manual and wrapping the "element delete code" in a manually created transaction.

I've thought about creating a new family that I would import into the project with a fixture that is a single inch in size, and instead of deleting the fixture, setting the fixture to this really small family. However with that solution I feel like I'm fixture the symptom and not the problem.

I've attached my code and a sample project where it crashes. Is there any insight you can provide about this problem?

Thanks again for your time.

**Answer:** Congratulations on the great success of your first add-in!

Regarding your question: you are iterating over the elements and deleting them at the same time.

This is not allowed.

Iterate first and collect a list of elements to delete.

Delete them in a second separate step.

If this helps and you get it to work, this might make a nice blog article, if you are willing to share it, including the mail you sent me already, describing what a great success your first add-in was in your office.

I hope this next add-in (and the following ones?) work out as well as the first.

**Response:** Your solution worked.

I just iterated through and created a list and then deleted that list.

Here is the code I used:
```vbnet
  ' Loop over rooms...
  If Not roomname = "Room not found" \_
    And Not roomname = "Fixture found in multiple rooms" \_
  Then
    idstodelete.Add(e.Id)
  End If
  ' End loop.

  ' Delete the elements here

  Dim id As ElementId
  For Each id In idstodelete
    Dim e As Element
    e = doc.GetElement(id)
    doc.Delete(e)
  Next
```

**Answer:** Glad it helped.

Your code can be further improved by using the Document Delete method overload taking an ElementId collection argument instead of the one taking an element itself.
That provides two advantages:

- No need to open the element from its element id.- Delete all of the elements in the list in one fell swoop.