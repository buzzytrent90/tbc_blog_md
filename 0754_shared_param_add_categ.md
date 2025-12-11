---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: code_example
optimization_date: '2025-12-11T11:44:14.525219'
original_url: https://thebuildingcoder.typepad.com/blog/0754_shared_param_add_categ.html
post_number: '0754'
reading_time_minutes: 4
series: general
slug: shared_param_add_categ
source_file: 0754_shared_param_add_categ.htm
tags:
- csharp
- elements
- family
- parameters
- python
- revit-api
- vbnet
title: Adding a Category to a Shared Parameter
word_count: 814
---

### Adding a Category to a Shared Parameter

Today is the first day of the Munich Revit API training.
While I am getting ready for that, and as a break from the new-fangled 2013 related stuff, here is an important question and and answer with a useful utility method by Miroslav Schonauer for handling shared parameters:

**Question:** I am using VB.NET to iterate through a set of families within a project, add shared parameters to types, and defining the category to apply the parameter to the family using the following code:
```csharp
  'create a category set with the Element category in it
  Dim categories As Autodesk.Revit.DB.CategorySet
  categories = app.Create.NewCategorySet
  Dim LCategory As Autodesk.Revit.DB.Category
  LCategory = doc.Settings.Categories.Item( \_
    iCategory.Name.ToString)
  categories.Insert(LCategory)
  'create a new Type binding for the Symbol categories
  Dim TypeBinding As Autodesk.Revit.DB.TypeBinding
  TypeBinding = app.Create.NewTypeBinding(categories)
  'Bind the parameter
  doc.ParameterBindings.Insert( \_
    sharedParameterDefinition, TypeBinding)
```

This works fine for the first family with a specific category, such as say Furniture; all furniture families then have the parameter applied and updated.

When I try and apply the same parameter to another family from another category, e.g. Special Equipment, I get no error, but the code fails to apply to the parameter.

How do I add more categories of the families to the parameter?

**Answer:** Just make a collection of all the categories that you wish to bind your shared parameter to, put them all into the same CategorySet, and make one single call to NewTypeBinding with the entire set containing all the categories in one go.

**Response:** The problem is that I do not know the categories when I first create the shared parameter, and these may change at any point in the future.
The add-in is reading data from an external source and then creating and updating all parameters and values on the fly.

Can I retrieve the existing categories that apply to a parameter, add my new category to the CategorySet, and then create a NewTypeBinding, or can you only add the categories at initial creation?

**Answer:** You can add additional categories later on as well.
Here is some code which does what you need, and much more besides:
```python
public static BindSharedParamResult BindSharedParam(
  Document doc,
  Category cat,
  string paramName,
  string grpName,
  ParameterType paramType,
  bool visible,
  bool instanceBinding )
{
  try // generic
  {
    Application app = doc.Application;

    // This is needed already here to
    // store old ones for re-inserting

    CategorySet catSet = app.Create.NewCategorySet();

    // Loop all Binding Definitions
    // IMPORTANT NOTE: Categories.Size is ALWAYS 1 !?
    // For multiple categories, there is really one
    // pair per each category, even though the
    // Definitions are the same...

    DefinitionBindingMapIterator iter
      = doc.ParameterBindings.ForwardIterator();

    while( iter.MoveNext() )
    {
      Definition def = iter.Key;
      ElementBinding elemBind
        = (ElementBinding) iter.Current;

      // Got param name match

      if( paramName.Equals( def.Name,
        StringComparison.CurrentCultureIgnoreCase ) )
      {
        // Check for category match - Size is always 1!

        if( elemBind.Categories.Contains( cat ) )
        {
          // Check Param Type

          if( paramType != def.ParameterType )
            return BindSharedParamResult.eWrongParamType;

          // Check Binding Type

          if( instanceBinding )
          {
            if( elemBind.GetType() != typeof( InstanceBinding ) )
              return BindSharedParamResult.eWrongBindingType;
          }
          else
          {
            if( elemBind.GetType() != typeof( TypeBinding ) )
              return BindSharedParamResult.eWrongBindingType;
          }

          // Check Visibility - cannot (not exposed)
          // If here, everything is fine,
          // ie already defined correctly

          return BindSharedParamResult.eAlreadyBound;
        }

        // If here, no category match, hence must
        // store "other" cats for re-inserting

        else
        {
          foreach( Category catOld
            in elemBind.Categories )
              catSet.Insert( catOld ); // 1 only, but no index...
        }
      }
    }

    // If here, there is no Binding Definition for
    // it, so make sure Param defined and then bind it!

    DefinitionFile defFile
      = GetOrCreateSharedParamsFile( app );

    DefinitionGroup defGrp
      = GetOrCreateSharedParamsGroup(
        defFile, grpName );

    Definition definition
      = GetOrCreateSharedParamDefinition(
        defGrp, paramType, paramName, visible );

    catSet.Insert( cat );

    InstanceBinding bind = null;

    if( instanceBinding )
    {
      bind = app.Create.NewInstanceBinding(
        catSet );
    }
    else
    {
      bind = app.Create.NewTypeBinding( catSet );
    }

    // There is another strange API "feature".
    // If param has EVER been bound in a project
    // (in above iter pairs or even if not there
    // but once deleted), Insert always fails!?
    // Must use .ReInsert in that case.
    // See also similar findings on this topic in:
    // http://thebuildingcoder.typepad.com/blog/2009/09/adding-a-category-to-a-parameter-binding.html
    // - the code-idiom below may be more generic:

    if( doc.ParameterBindings.Insert(
      definition, bind ) )
    {
      return BindSharedParamResult.eSuccessfullyBound;
    }
    else
    {
      if( doc.ParameterBindings.ReInsert(
        definition, bind ) )
      {
        return BindSharedParamResult.eSuccessfullyBound;
      }
      else
      {
        return BindSharedParamResult.eFailed;
      }
    }
  }
  catch( Exception ex )
  {
    MessageBox.Show( string.Format(
      "Error in Binding Shared Param: {0}",
      ex.Message ) );

    return BindSharedParamResult.eFailed;
  }
}
```

It is pretty old and includes a few interesting notes in the comments.
It has continued working fine over many releases now.

For completeness' sake, here is the
[entire HelperParams.cs C# module](zip/HelperParams.cs).

**Response:** That resolved my problem and everything is working great.

The problem was that I was using the doc.ParameterBindings.Insert method.
Since the binding already exists, I have to use the ReInsert method instead.

Ok, now I am off to give this API training.
Wish me luck!