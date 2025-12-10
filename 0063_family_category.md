---
post_number: "0063"
title: "Family Category and Filtering"
slug: "family_category"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'revit-api']
source_file: "0063_family_category.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0063_family_category.html"
---

### Family Category and Filtering

So here we are in a brand new year, 2009.
Welcome back!
Here is an interesting question regarding families, their categories, and filtering, raised last year by Martin Schmid of Autodesk.

**Question:** In the following code, I attempt to use Revit API filtering to select all families with a built-in category value of BuiltInCategory.OST\_MechanicalEquipment:

```csharp
Document doc = app.ActiveDocument;
List<Element> famsInCat = new List<Element>();

Autodesk.Revit.Creation.Filter cf
  = app.Create.Filter;

TypeFilter typeFilter
  = cf.NewTypeFilter( typeof( Family ) );

CategoryFilter catFilter
  = cf.NewCategoryFilter( bic );

LogicAndFilter andFilter
  = cf.NewLogicAndFilter( typeFilter, catFilter );

doc.get\_Elements( andFilter, famsInCat );
return famsInCat;
```

Unfortunately, this does not work as expected, even when some mechanical equipment families have been loaded into the model.

Instead, this more cumbersome iterative approach can be used successfully:

```csharp
Category categoryMechanicalEquipment
  = doc.Settings.Categories.get\_Item( bic );

ElementId categoryId
  = categoryMechanicalEquipment.Id;

ElementIterator it
  = doc.get\_Elements( typeof( Family ) );

while( it.MoveNext() )
{
  Family family = it.Current as Family;
  bool categoryMatches
    = family.FamilyCategory.Id.Equals( categoryId );

  if( categoryMatches )
  {
    famsInCat.Add( family );
  }
}
return famsInCat;
```

Do you see any errors in my logic?

**Answer:** The reason it does not work is because the Family class does not implement the Category property for all family files.
Although the property is provided by the Family class, it sometimes returns null.
The filter is probably using this property internally, but to no avail.

Therefore, you have to use some other method to check the category of a family.
On some family instances, you can use FamilyCategory instead of Category, like you are doing in your working code snippet.
Unfortunately, even this work-around is not reliable for all families.
On some families, the FamilyCategory property is not implemented either.
The most reliable method is to check the Category property on any one symbol contained in the family.
Mostly, it is practical to just use the first symbol defined in the family.
Here is some updated code demonstrating this approach:

```csharp
Category categoryMechanicalEquipment
  = doc.Settings.Categories.get\_Item( bic );

ElementId categoryId
  = categoryMechanicalEquipment.Id;

ElementIterator it
  = doc.get\_Elements( typeof( Family ) );

while( it.MoveNext() )
{
  Family family = it.Current as Family;
  foreach( Symbol s in family.Symbols )
  {
    if( s.Category.Id.Equals( categoryId ) )
    {
      famsInCat.Add( family );
    }
    break; // only need to look at first symbol
  }
}
return famsInCat;
```

Unfortunately, as Martin points out above, this does indeed mean that you have to add some own slightly cumbersome code after the filtered element retrieval to check for the desired category.