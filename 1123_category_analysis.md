---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.8
content_type: code_example
optimization_date: '2025-12-11T11:44:15.347500'
original_url: https://thebuildingcoder.typepad.com/blog/1123_category_analysis.html
post_number: '1123'
reading_time_minutes: 2
series: elements
slug: category_analysis
source_file: 1123_category_analysis.htm
tags:
- family
- parameters
- python
- revit-api
- views
- elements
title: Category Analysis with and without Python
word_count: 466
---

### Category Analysis with and without Python

Here is a useful approach for analysing categories by Alexander Ignatovich of
[Investicionnaya Venchurnaya Companiya](http://www.iv-com.ru),
originating from the following query:

**Question:** How can I check that a given family is an annotation?

For example, I have a family with BuiltInCategory.OST\_GridHeads family category.

Revit knows that this is an annotation category and shows it inside the corresponding group in the project browser.

However, there are no annotation "super groups" in the BuiltInCategory enumerable, and family.FamilyCategory.Parent is null.

**Answer:** I am not aware of any way to achieve this in Revit 2014.

As you certainly thought of yourself, you could implement a workaround by hard-coding a list enumerating and manually classifying all the categories that you are interested in.

For that, it might also be helpful to keep in mind that instances of annotation categories are generally view-specific, and model categories are generally not. This requires instances to be present that you can interrogate.

**Response:** It is actually more important for me to know, that a family is **not** an annotation.

I created the following hack to help determine this.

Instead of looking at more than 700 categories one by one, I do the following:

- Manually create a project parameter.
- Add check marks for it to all the categories I am interested in.
- Iterate over the project parameter bindings.
- For every binding of my project parameter, for every category bound to it, list the built-in category enumeration name and underlying integer value.

The latter two steps are quickly implemented by this IronPython code in the RevitPythonShell:

```
  from System import *
  bindings = doc.ParameterBindings
  it = bindings.ForwardIterator()
  while it.MoveNext():
    if it.Key.Name == 'my': # project parameter name
      for cat in it.Current.Categories:
        print Enum.ToObject(BuiltInCategory, cat.Id.IntegerValue)
```

For instance, I wanted to get the list of all "model" categories; there are about 800 categories in the BuiltInCategory enumeration. Project parameters can be applied to all "model" categories and also to some system and analytical model categories. It is really good way to extract the desired category list: just select all categories and deselect system and analytical model categories. This is especially helpful when you are working with a localized version of the program, because sometimes the translation is not perfect :-)

Here is a picture to clarify:

![Selected project parameter categories](img/project_param_categories.png)

Many thanks to Alexander for sharing this powerful idea!

It shows once again how handy and effective it is to use the tools and functionality provided by Revit as far as possible, which is a long way, and enhance them with
[interactive Python for in-depth Revit database exploration](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html).