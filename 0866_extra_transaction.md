---
post_number: "0866"
title: "Extra Transaction or Regeneration Required"
slug: "extra_transaction"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'python', 'revit-api', 'selection', 'transactions', 'views']
source_file: "0866_extra_transaction.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0866_extra_transaction.html"
---

### Extra Transaction or Regeneration Required

Whenever your add-in modifies the model in any way and you wish to query the Revit database, you need to pay careful attention to ensure that you do not retrieve stale or invalid data.

If anything unexpected whatsoever happens, one of the first things to consider is the possible need for a document regeneration or additional separate transactions between steps.

These kinds of issues arise frequently.

I discussed adding an extra transaction to
[modify beam parameters and add openings](http://thebuildingcoder.typepad.com/blog/2010/01/extra-transaction-required.html) to
it and to
[create an opening](http://thebuildingcoder.typepad.com/blog/2012/07/creating-an-opening.html) in
a slab.

Sometimes a simple
[document regeneration](http://thebuildingcoder.typepad.com/blog/2010/06/to-regenerate-or-not-to-regenerate.html) is
also sufficient, e.g. to
[set the width of a newly created TextNote](http://thebuildingcoder.typepad.com/blog/2010/11/setting-text-width-requires-regen.html).

Here are two more typical examples:

- [Extra transaction required after loading tag](#2)
- [Extra regeneration required to populate materials](#3)

#### Extra Transaction Required After Loading Tag

Here is an issue raised by Max, [Maciej Szlek](http://maciejszlek.pl), who already contributed code to help
[distinguish](http://thebuildingcoder.typepad.com/blog/2011/03/distinguishing-mep-element-shape.html)
[MEP fitting shapes](http://thebuildingcoder.typepad.com/blog/2011/05/improved-mep-element-shape-and-mount-ararat.html):

**Question:** I have a duct system and want to attach my tag to its elements, using the code to
[set a tag type](http://thebuildingcoder.typepad.com/blog/2010/06/set-tag-type.html).
I tried something like this:
```csharp
doc.LoadFamilySymbol(
[my\_tag\_family.rfa],
[type\_name],
out tagType);
IndependentTag tag = doc.Create.NewTag(
doc.ActiveView,
element,
true,
TagMode.TM\_ADDBY\_CATEGORY,
TagOrientation.Horizontal,
XYZ.Zero);
tag.ChangeTypeId(tagType.Id);
```

This code works fine in some projects, whereas in others Revit throws an InvalidOperationException saying that there is no tag available in the call to the NewTag method.
The behaviour seems to vary between different projects and on different elements.

How can that be?

**Answer:** Have you tried either regenerating the document or placing the tag symbol loading and the tag creation calls in separate transaction, committed individually?

**Response:** Problem solved.
As it turns out, the Revit project needs to have some suitable tag type loaded before calling NewTag.
Your transaction hint was very helpful.
My earlier code was not enough as it stands.
It actually requires separate transactions to load the tag type and add a new tag if there was no other tag loaded before.
I did not previously know what Revit meant by 'there is no tag available'.

Furthermore, I was initially using automatic transaction mode for this command, thinking that would free me from worrying about any transaction handling myself at all.
I now learned that the automatic transaction mode does not enclose each action in a separate transaction, but wraps all executed functions in the whole external command into one single one.
In this case, such an approach is not usable at all.

Thank you for a very useful API lesson :))

**Answer:** Yes, I already mentioned that automatic transaction mode is unofficially deprecated, and manual mode is recommended for all cases nowadays.
[Automatic transaction mode is considered obsolete](http://thebuildingcoder.typepad.com/blog/2012/05/read-only-and-automatic-transaction-modes.html).

Arnošt Löbel's AU class on
[Core Frameworks in the Revit API](http://thebuildingcoder.typepad.com/blog/2012/11/au-classes-on-python-ui-server-and-framework-apis.html#4) provides
more details on this topic.

Many thanks to Max for raising this issue and verifying that the solution works!

#### Extra Regeneration Required to Populate Materials

One of the issues we took a look at during the DevLab at AU last Tuesday was the list of materials attached to a newly duplicated symbol.

At first glance, it appeared that a duplicated family symbol had zero entries in its list of materials, whereas the original symbols material list was correctly populated.

On further exploration, we discovered that this is another case of retrieving stale data after a model modification.

The behaviour is easily rectified by a call to regenerate the model after the symbol duplication.

Here is the final implementation of the external command implementing to test the issue.
It expects a structural steel column to be pre-selected, which is a family instance whose associated symbol has exactly one entry in its materials list.
We duplicate the symbol.
This requires defining a new name.
To simplify repeated testing, we generate a new name by simply appending the clock tick counter to the original one.
Directly after the call to Duplicate, the new symbol's material list has zero entries, which makes no sense for this kind of symbol.
A call to Regenerate fixes that:

```python
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;
    Selection sel = uidoc.Selection;
    FamilyInstance inst = null;

    if( 1 == sel.Elements.Size )
    {
      foreach( Element e in sel.Elements )
      {
        inst = e as FamilyInstance;

        break;
      }
    }

    if( null == inst )
    {
      message = "Please select one "
        + "single structural column";

      return Result.Failed;
    }

    using( Transaction tx
      = new Transaction( doc ) )
    {
      tx.Start( "Duplicate Symbol" );

      FamilySymbol symbol = inst.Symbol;

      string name = symbol.Name
        + DateTime.Now.Ticks.ToString();

      int nMaterialsBefore
        = symbol.Materials.Size;

      symbol = inst.Symbol.Duplicate( name )
        as FamilySymbol;

      // The model is in a temporary state that does
      // not make sense. Regenerate to clean this up.

      int nMaterialsAfter
        = symbol.Materials.Size;

      doc.Regenerate();

      nMaterialsAfter = symbol.Materials.Size;

      Debug.Assert(
        nMaterialsAfter.Equals( nMaterialsBefore ),
        "why does the material get lost, please?" );

      tx.Commit();
    }
    return Result.Succeeded;
  }
}
```

For completeness' sake, here is
[DuplicatedSymbolLosesMaterial.zip](zip/DuplicatedSymbolLosesMaterial.zip) containing
the complete source code, Visual Studio solution and add-in manifest of this little test command.