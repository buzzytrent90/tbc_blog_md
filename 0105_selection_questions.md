---
post_number: "0105"
title: "Selection Questions"
slug: "selection_questions"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'rooms', 'selection', 'windows']
source_file: "0105_selection_questions.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0105_selection_questions.html"
---

### Selection Questions

A whole bunch of questions on selection of elements and points came in during the last day or two.
Here are a few of them:

- How can I force the user to select an element when my external command begins?
  [Answer...](#1248008)
- How can I invoke an existing Revit command when my external command begins?
  [Answer...](#1248007)
- In the discussion of the
  [Revit window handle and modeless dialogues](http://thebuildingcoder.typepad.com/blog/2009/02/revit-window-handle-and-modeless-dialogues.html)
  you list selected elements in a modeless dialogue box.
  Could the reverse operation be implemented, displaying the selected elements in a list box with check marks next to each, and unselecting them in the Revit user interface if the user removes the check mark?
  [Answer...](#1247912)
- I'd like to be able to click at a point on a drawing and get its coordinates.
  Is this possible without actually selecting an existing element?
  [Answer...](http://thebuildingcoder.typepad.com/blog/2008/10/picking-a-point.html)
- I call PickOne repeatedly, asking the user to pick elements on the screen.
  However, when I iterate the selection set, the order is different.
  How can I retain the selection order when repeatedly calling PickOne?
  [Answer...](#1248049)

#### Selecting an element

This functionality is supported by the Revit API and its use is pretty straightforward.
Two interactive user selection methods can be used while an external command is active, PickOne and WindowSelect.
You can call PickOne in a loop until the user selects the specific element that you require.
The basics of using PickOne are discussed in
[this post](http://thebuildingcoder.typepad.com/blog/2008/12/pickone.html).
We also provide an example demonstrating how to
[pick a specific element type](http://thebuildingcoder.typepad.com/blog/2009/01/verona-revit-api-training.html).
Finally, here is a more complex example
[presenting the current contents of the selection set in a modeless dialogue](http://thebuildingcoder.typepad.com/blog/2009/02/revit-window-handle-and-modeless-dialogues.html)
and discussing how more complex analysis of the current selection could be added.

#### Invoking a Revit command

Unfortunately the Revit API does not yet provide access for an external command to invoke any other existing Revit commands.
Even though the official API does not provide this access to the command, you might be able to use the Revit journaling mechanism or the Win32 API to simulate user input to provide this functionality.
Please note that both of these approaches are completely unsupported and must be researched, tested and maintained by you completely at your own risk.

The basics of the API access to the journaling mechanism are demonstrated by the Revit SDK Journaling sample.

Using the Win32 API, you can simply send Windows messages simulating user input to either select the corresponding Revit menu entry.
For example, to invoke the Room command, you could simulate selecting the Revit menu entry Drafting > Room or typing in the keyboard shortcut RM.
This is discussed in the post on
[driving Revit from outside](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html).
An alternative method to using Win32 API calls directly yourself to simulate user input is also available:
[AutoHotKey](http://thebuildingcoder.typepad.com/blog/2009/01/autohotkey.html).
If you do decide to use the Win32 API, you may also be interested in the discussion on the
[Revit window handle and modeless dialogues](http://thebuildingcoder.typepad.com/blog/2009/02/revit-window-handle-and-modeless-dialogues.html).

#### Deselecting elements through a checkmark in a modeless dialogue

Yes, that is perfectly possible. In the
[modeless dialogue](http://thebuildingcoder.typepad.com/blog/2009/02/revit-window-handle-and-modeless-dialogues.html)
we presented, the selected elements are simply listed in a text label.
They could also be displayed in a list box instead, with a check mark for each selected item.
In that case, you could implement code to unselect the element in the Revit user interface by unchecking its corresponding check box in the modeless dialogue.

This is possible because the Document Selection property is a Selection object which has a property Elements, which is a SelElementSet instance, which provides methods named Erase and Remove. This class is derived from ElementSet and Erase is inherited from that class. I am not sure how Erase affects the current selection in the Revit UI; you might like to test that.
Anyway, according to the help file, Remove will "remove the specified from the selected element set, just like de-select it from the UI".
Although the formulation is rather strange, I would say that calling the method with a given selected element will unselect it in the document selection set and in the Revit user interface.
I am not sure whether Erase will have exactly the same effect as Remove.
I might try it out as soon as I have some time.

#### Picking a Point

This issue and the meagre workarounds I can think of were already discussed in the post on
[picking a point](http://thebuildingcoder.typepad.com/blog/2008/10/picking-a-point.html).

#### Retaining the element selection order

The reason that there is no inherent order in the Document Selection collection is that its Elements constitute a set and as such are unordered.
If you wish to keep track of the original selection order, you would have to do so yourself. You could achieve this by iterating over the entire selection set after each call to PickOne and determining which new element has been added by the last call. You could then keep track of the elements added in an own ordered list, in parallel with the unordered set maintained by Revit.