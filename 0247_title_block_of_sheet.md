---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.1
content_type: qa
optimization_date: '2025-12-11T11:44:13.617445'
original_url: https://thebuildingcoder.typepad.com/blog/0247_title_block_of_sheet.html
post_number: '0247'
reading_time_minutes: 6
series: views
slug: title_block_of_sheet
source_file: 0247_title_block_of_sheet.htm
tags:
- elements
- family
- parameters
- references
- revit-api
- schedules
- sheets
- transactions
- views
title: Title Block of Sheet
word_count: 1147
---

### Title Block of Sheet

Here is a question that crops up now and then in different variations, on how to access the title block information of a sheet:

**Question:** My drawing list routine exports various sheet view information to a spreadsheet.
I would like to include the name of the title block family that is associated with the view.
What is the best way to find it?
Do I need to loop through family instances, and do a reverse lookup?

My snooping tool freaks out whenever I select a title block family from the list of family instances, so I don't know if the host of that family instance will tie it to the specific sheet view?

**Answer:** If your version of
[RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html) fails
to display some information that you are interested in, I suggest you
[debug it and fix the problem](http://thebuildingcoder.typepad.com/blog/2009/08/fixing-rvtmgddbg-for-mep-connectors.html),
e.g. start it in the debugger and add an exception handler to the offending code.

I am confused by your statement 'name of the title block family that is associated with the view'.
Do you mean sheet instead of view?
In general, each sheet has a title block associated with it, and then hosts multiple views.

To see which title block is added when I add a new sheet to the model, I use a technique to
[track all new elements added](http://thebuildingcoder.typepad.com/blog/2009/02/locked-dimensioning.html).
This technique uses the Revit API introduction labs Lab 2-1 element lister.
I create a file containing a list of all elements in the BIM before and after adding the sheet and then compare them.
Here is the result, in this case the list of elements created by adding a new sheet to the project:

```
C:\tmp\ >diff RevitElementsBeforeSheet.txt RevitElementsAfterSheet.txt
2581a2582,2595
> Id=137766; Class=ViewSheet; Category=Sheets; Name=Unnamed; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a26
> Id=137767; Class=Element; Category=?; Name=Unnamed; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a27
> Id=137768; Class=Element; Category=Viewports; Name=Title w Line; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a28
> Id=137769; Class=Element; Category=?; Name=A101 - Unnamed; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a29
> Id=137770; Class=Element; Category=Work Plane Grid; Name=Unnamed; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a2a
> Id=137771; Class=SketchPlane; Category=?; Name=Unnamed; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a2b
> Id=137772; Class=FamilyInstance; Category=Title Blocks; Name=A0 metric; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a2c
> Id=137773; Class=View; Category=?; Name=; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a2d
> Id=137774; Class=Element; Category=?; Name=; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a2e
> Id=137775; Class=Element; Category=Viewports; Name=Title w Line; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a2f
> Id=137776; Class=Element; Category=?; Name=; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a30
> Id=137777; Class=Element; Category=?; Name=; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a31
> Id=137778; Class=Element; Category=?; Name=; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a32
> Id=137779; Class=Element; Category=Schedule Graphics; Name=; UniqueId=7e3fda1f-b209-4185-9037-575bba9df262-00021a33
```

If these lines are truncated in the browser view, you can copy and paste them to an editor to see the full text.

From this you can see that the new sheet has the element id 137766, and the associated title block is a FamilyInstance with id 137772.

Once the new elements have been identified, the Revit API introduction labs
[built-in parameter checker](http://thebuildingcoder.typepad.com/blog/2009/04/deeper-parameter-exploration.html) can
be used to explore their data.
In the Revit user interface, I can select the sheet via Modify > Element ID > Select by ID > 137766 > OK and then invoke the built-in parameter checker to explore its parameters.

Looking at the sheet parameters sorted by type and going through all parameters whose value is an element id, I see that there is no reference to 137772, so the sheet does not have a parameter explicitly specifying the title block by its element id, as one might have hoped.

Looking at the title block parameters yields better results, among others the family that you are asking for and the sheet number:

```
ELEM_FAMILY_AND_TYPE_PARAM
  Family and Type
  ElementId 28295
  Title Blocks 'A0 metric'

SHEET_NUMBER
  Sheet Number
  String
  A101
```

This means that although we have not found a pointer from the sheet to the title block, we have found information about the inverse relationship from the title block to the sheet.
We can therefore use the
[relationship inverter algorithm](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html) to
identify the title block from the sheet, if needed.

I also had an idea for another, completely different approach: I tried deleting the sheet again after it had been added and checked the resulting list of elements after that operation:

```
C:\tmp\ >diff
  RevitElementsAfterSheetDelete.txt
  RevitElementsBeforeSheet.txt

Files are identical.
```

Happily, as it turns out, the new list of elements is exactly the same as it was before the sheet was added.
This means that the deletion of the sheet caused the associated title block to be deleted as well.
This means that we can probably make use of the
[doc.Delete method to detect dependent elements](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html).
Using that approach, you delete the sheet inside a transaction, save the list of element ids returned by the doc.Delete method, and abort the transaction to undo the deletion operation. The list of ids will then include element ids of both the sheet and the associated title block.

I hope this resolves your issue twice over.

#### Title Block Modification

By the way,
[Ricardo Cardial](http://autocad-revit-arquitetura.blogspot.com)
raised a related
[question](http://thebuildingcoder.typepad.com/blog/2009/08/shared-parameter-visibility.html#comment-6a00e553e1689788330120a5189ca2970b) on
how to modify the text labels defined in the title block, which we will partially repeat here in order to ensure that these useful ideas do not get lost:

**Question:** How can we change the information of the scale label shown in title blocks?
When we have two different scales on the sheet, the value is 'As indicated'.
I would like to change that, for instance to translate it to French or Italian.

**Answer:** Look at the original comment for the full answer, which consists of two parts.
They can be summarised like this:

- [Read the data through the built-in parameter SHEET\_SCALE](http://thebuildingcoder.typepad.com/blog/2009/08/shared-parameter-visibility.html#comment-6a00e553e1689788330120a57141eb970c) with
  the display name 'Scale'.
  It is a string-valued parameter and is unfortunately marked as read-only.- [Modify the family file](http://thebuildingcoder.typepad.com/blog/2009/08/shared-parameter-visibility.html#comment-6a00e553e1689788330120a5715f8c970c) used
    to define the title block of the sheet.