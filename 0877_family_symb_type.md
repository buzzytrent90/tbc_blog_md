---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.9
content_type: qa
optimization_date: '2025-12-11T11:44:14.783609'
original_url: https://thebuildingcoder.typepad.com/blog/0877_family_symb_type.html
post_number: 0877
reading_time_minutes: 4
series: elements
slug: family_symb_type
source_file: 0877_family_symb_type.htm
tags:
- elements
- family
- filtering
- parameters
- revit-api
- selection
- views
title: Family Symbols versus Types and SelectionFilterElement
word_count: 797
---

### Family Symbols versus Types and SelectionFilterElement

Here is a question that has been asked a couple of times recently and thus seems worthwhile clarifying:

**Question:** If I access the OwnerFamily.Symbols in a family document, i.e. an RFA file, it returns an empty collection.
If I load that same family into a project and access the Family.Symbols property, it returns the correct expected family types.
Why does this property not work in the family editor?

Put differently, the following code snippet will show zero in the message box:

```
  Document fdoc = app.OpenDocumentFile(
    "C:/Family.rfa" );

  Family fam = fdoc.OwnerFamily;

  MessageBox.Show(fam.Symbols.Size.ToString());
```

Loading the same family into a project and then querying the symbol count returns the correct number:

```
  Document doc = app.OpenDocumentFile(
    "C:/Project.rvt" );

  Family fam = null;

  doc.LoadFamily( "C:/Family.rfa",
    new FamilyOption(), out fam );

  MessageBox.Show(fam.Symbols.Size.ToString());
```

Again, why does the Family.Symbols property not work in the family context?

**Answer:** Why things are the way they are is often quite difficult to answer.
For that situation, I quite like the adage "Mine is not to wonder why, mine is but to do or die."

However, if I am allowed to rephrase you question constructively and ask "How can I access the symbols defined by a family in the family editor context?", the answer is clear and easy:

Use the FamilyManager class.
The family manager object is used to manage the family types and parameters within a family document.
You can access it through the Document.FamilyManager property on the family document object.

Getting back to the why and wherefore, it is helpful to understand that the family does not contain any symbols.
It only contains types.
The types can be compared to records in a database.
The family parameters define the database fields, i.e. columns, and the types represent the records, i.e. rows.
You could go even further and compare the individual family parameters with the database fields.

The symbols do not exist in the RFA.
They only spring into existence when the family is loaded into a project.

Here is a slightly more detailed explanation:

Family types and symbols are completely different objects with different purposes.

There can be FamilySymbols in a family document.
If so, these are the symbols defined by other families that are loaded into the document.

Family types live in the family where they are defined, whereas family symbols in a family always come from some other family loaded into it.

- Types can add parameters, formulas, etc. Symbols are limited to the definition in the source family.- Types can be added and deleted at run time. Symbols cannot, other than by special operations like Duplicate.- Symbols can be instantiated in the document they live in. Types cannot.

The FamilyType class in a family is not inherited from the Revit Element class, so types are not elements.
It offers the ability to access parameters either for read or write in conjunction with the FamilyManager.

For certain operations in a family, such as getting an image from a type, you can set the type to be active using the FamilyManager.CurrentType property, then generate an image from a view of the family using the Document.Export method and an appropriate ImageExportOptions argument.

#### SelectionFilterElement

Two posts of Harry Mattison's on the SelectionFilterElement introduced in the Revit 2013 API just caught my attention and also seem worthwhile pointing out here.

The SelectionFilterElement is a filter element that stores an explicit list of ElementIds.
Only elements who's ElementIds are contained in this list will pass the filter.

I mentioned the
[existence of this class](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#6)
briefly and rather cryptically in the initial overview of the
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html).

Harry now presents two running snippets of sample code using a SelectionFilterElement to
[save a set of elements](http://boostyourbim.wordpress.com/2012/12/11/save-a-set-of-elements-with-selectionfilterelement) and
subsequently
[retrieve and highlight them](http://boostyourbim.wordpress.com/2012/12/15/retrieving-and-selecting-elements-in-a-selectionfilterelement).

Definitely worth keeping in mind if you have a specific set of elements that you wish to access quickly.

#### Dynamo and Vasari Holiday Hacking

If the above is not enough for you, here is an invitation to venture further afield, into the realm of the Dynamo visual programming add-in for Revit:

- [Dynamo holiday project overview](http://autodeskvasari.com/profiles/blogs/dynamo-caution-elves-at-work)
- [Dynamic Relaxation via Dynamo](http://autodeskvasari.com/profiles/blogs/dynamo-holiday-relaxation-aid)
- [Youtube video of results](http://autodeskvasari.com/profiles/blogs/dynamo-new-video-relaxation-part-deux)