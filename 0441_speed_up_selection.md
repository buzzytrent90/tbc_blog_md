---
post_number: "0441"
title: "Speed Up Selection"
slug: "speed_up_selection"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'revit-api', 'selection', 'transactions', 'views']
source_file: "0441_speed_up_selection.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0441_speed_up_selection.html"
---

### Speed Up Selection

I am now back in Switzerland again after all my various travels, and looking forward to diving into the Revit API in depth again...

Here is a really surprising result on how to speed up programmatic selection of a large number of elements, triggered by a question from Joe Offord of
[Enclos Corp](http://www.enclos.com):

**Question:** I have a situation where I'd like to select a large number of curtain panels (over 1000) and add them to my UIDocument.Selection set.
These curtain panel families have a number of nested, shared components in them, numbers ranging from 5 to 30.
I'm using the following code in a loop:
```csharp
  Selection.Elements.Add(Panel)
  If Panel.SubComponents IsNot Nothing Then
    For Each subComp As FamilyInstance \_
      In Panel.SubComponents
      Selection.Elements.Add(subComp)
    Next
  End If
```

This process is very time-consuming and is taking about 5-10 minutes for just this portion of the code (including looping) to run.
Is there a better way to create my selection that would speed up things?

What I want when it's all done is to be able to see what elements I've selected.
I would normally select just the panel and not the subcomponents, but for some reason Revit will not highlight the subcomponents when I tried this.
When I click on the panel outside the API it will highlight the nested components too but I can't replicate this behaviour using the API.

**Answer:** Generally, the way to speed up the addition of a large number of elements to a collection consists of trying
to add all of them or as many as possible at once.

Unfortunately, the only methods I can see on the SelElementSet class to add elements to the collection just
take one element at a time:

- Add to add an element into the selected element set.- Insert to insert the specified element into the set.

There are no other useful methods that I can see on the ElementSet base class either, unfortunately.

One thing that you might be able to try would be to create your own ElementSet instance, let's say in a
variable 'a', and add the desired elements to that one by one using the Add method, but keeping the
ElementSet 'a' independent, not associated with the Revit user interface or current selection for the time
being.

Once you have populated 'a' with the elements you are interested in, you might be able to say
```csharp
  app.ActiveUIDocument.Selection.Elements = a;
```

This may possibly avoid some overhead performed by the Revit application when you manipulate the current
selection set directly, element by element. Or it may optimise that overhead by doing whatever it is doing
to all of them in one go. Or it may make no difference at all.

I cannot think of anything else to suggest, though.

**Response:** I tried passing in a new ElementSet into the Selection.Elements as you suggested but it got rejected.
However, I found a way to create a new SelElementSet and used that to modify the Selection.
The code looks something like this:
```csharp
  Dim newCollection As Selection.SelElementSet \_
    = Selection.SelElementSet.Create
  ' ~ code to add elements to newCollection ~
  m\_selection.Elements = newCollection
```

It is remarkable how much time this saves.
My selection of 1000+ panels went from roughly 5 minutes to 15 seconds.

Thanks for the excellent advice, as always.

**Answer:** A colleague of mine asked a relevant question in this context, although I don't really have an idea of whether his assumption is correct or not:

"I guess this was using AUTO transaction, right? I wonder if in case of manual transaction, placing the update/regenerate at the very end would have the same fast result even when adding the entities to the selection set one by one?"

So, just to clarify, what transaction mode and regeneration options are you actually using?

Did you do any experiments with these settings before asking us for a solution?

Thank you in advance for the clarification!

**Response:** I used Manual Transactions for both methods I tried. I made a new Transaction specifically for this selection routine and called Document.Regenerate only once, right before the call toTransaction.Commit. This was the same for both the single addition method (using Selection.Elements.Add) and the group selection method (using new SelElementSet).

In my biggest selection experiment I had 15,294 Generic Model elements with 1706 Curtain Panel elements selected. The fast way took about 15 seconds. The slow way (using the code I posted first) was about 15% complete after 3 minutes before I shut it down. I was running both commands in a 3D view of the entire model.

**Answer:** Thank you very much for the clarification! It does indeed help a lot, since it indicates that the improvement we discovered cannot be achieved by simply optimising the transaction and regeneration approaches.

Very many thanks to Joe for raising this interesting question!