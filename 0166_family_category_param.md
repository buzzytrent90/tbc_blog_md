---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: code_example
optimization_date: '2025-12-11T11:44:13.473306'
original_url: https://thebuildingcoder.typepad.com/blog/0166_family_category_param.html
post_number: '0166'
reading_time_minutes: 2
series: elements
slug: family_category_param
source_file: 0166_family_category_param.htm
tags:
- elements
- family
- parameters
- python
- revit-api
- transactions
title: Get and Set Family Category and Parameters
word_count: 392
---

### Get and Set Family Category and Parameters

I am receiving a lot of questions about parameters in family documents, so I will be posting several solutions in this area in the next few days.
The first one is short and simple and suited for a rapid Saturday post.
There is so much news coming that I do want to cram it in right away.
This is a question from Jose Fandos of
[Andekan LLC](http://www.andekan.com):

**Question:**
How can we get and set the family category and the family parameters seen in the "Family Category and Parameters" Revit dialogue box?
This dialogue can be displayed by selecting Create > Category and Parameters in the family editor:

![Family Category and Parameters dialogue](img/family_category_and_params.png)

The parameters include the OmniClass number, for instance.
We are not looking for the type or symbol parameters, and we need this access from the family editor, not from the project editor.

**Answer:**
I used the RvtMgdDbg tool to explore a family file to answer your query.
Using that, I can step into RvtMgdDbg > Snoop Application... > Application > Active Document > Owner Family > Parameters to see the family parameters, including the OmniClass number with the built-in parameter enumeration value OMNICLASS\_CODE:

![Snoop OmniClass number](img/snoop_omniclassnumber.png)

The family category is also available as a property on this owner family object, but it is read-only.

Here is the external command Execute method that I implemented to demonstrate read access to both of these properties and write access to the OmniClass number:

```python
BuiltInParameter \_bip = BuiltInParameter.OMNICLASS\_CODE;

public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  if( !doc.IsFamilyDocument )
  {
    message
      = "Please run this command in a family document.";
  }
  else
  {
    Family f = doc.OwnerFamily;
    Category c = f.FamilyCategory;
    Parameter p = f.get\_Parameter( \_bip );

    Debug.Print(
      "Category '{0}', OmniClassNumber {1}",
      c.Name, p.AsString() );

    p.Set( "Jeremy" );

    Debug.Print( "Modified OmniClassNumber {0}",
      f.get\_Parameter( \_bip ).AsString() );
  }
  return IExternalCommand.Result.Failed;
}
```

Note that I am returning Failed from the Execute method, so the transaction that Revit is managing for my command will be aborted, so the changes I made will not actually be committed.

Next week I plan to discuss work-arounds for the challenges surrounding shared parameters in family files, so stay tuned.
Happy weekend to all!