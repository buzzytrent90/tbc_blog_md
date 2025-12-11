---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.808563'
original_url: https://thebuildingcoder.typepad.com/blog/1341_type_catalogue.html
post_number: '1341'
reading_time_minutes: 5
series: elements
slug: type_catalogue
source_file: 1341_type_catalogue.htm
tags:
- family
- parameters
- revit-api
- selection
- views
- elements
title: Type Catalogues and FireRating in the Cloud
word_count: 988
---

### Type Catalogues and FireRating in the Cloud

Let's veer away from the Revit API for a moment and take a look at family type catalogues, since questions on those pop up regularly in connection with programming as well:

- [Programmatic type catalogue creation](#2)
- [Tweaking type catalogue behaviour](#3)
- [Type catalogue parameters](#4)
- [Type catalogue units](#5)

#### Programmatic Type Catalogue Creation

**Question:**
Is there any API support (function) to create type catalogues?

**Answer:**
As far as I know, there are two ways to create type catalogues, and both are programmatically accessible:

- Create a TXT file associated with the family listing the possible combinations of dimensioning parameters.
- Create an MEP specific type catalogue for MEP parts.

I assume you are interested in the latter?

I tried searching the Revit API help for 'catalog', and that does indeed not return any relevant results.

However, searching the various
[What's New in the Revit API](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html) overviews,
I found the following
[for Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html):

#### CSV Fitting Parameter Removal

Because CSV files are no longer used to drive MEP fitting parameters, Revit supports a new set of APIs to manage fitting parameters through several classes:

- FamilySizeTableManager – manages importing and exporting of legacy CSV data and size tables.
- FamilySizeTable – manages specific sizes of fittings.
- FamilySizeTableColumn – manages a specific dimension of a given size in a size table
- FamilySizeTableErrorInfo – reports any errors when importing an file with CSV size table into a FamilySizeTable

Is this possibly what you are looking for?

So maybe what you are after is the FamilySizeTable class and all its associated methods and properties.

**Response:**
Actually we are interested in the first item, creating a TXT file associated with the family listing the possible combinations of dimensioning parameters.

Is there a way to create this txt file using Revit API?

**Answer:**
That case is quite trivial, as far as I know:

Simply create a TXT file with the same name as the family definition RFA, and it will be picked up automatically by Revit.

I am not aware of any specific Revit API to achieve this or support it in any way.

To create the file, you can use standard .NET functionality, or anything else you like.

Basically this approach is simply making use of the standard family definition user interface functionality.

#### Tweaking Type Catalogue Behaviour

**Question:**
Is there any support for TXT family type catalogue files inside Revit API?

If I use one of the LoadFamily methods and there is a TXT file along with the RFA, will the type selection dialogue pop up automatically?

Is there any way to tweak or suppress this behaviour?

**Answer:**
While there isn't any support to handle the family type catalogue TXT files using the API, I was wondering if, instead of using LoadFamily, using LoadFamilySymbol to load up the specific symbols would work for you.

Alternatively, you can also load any single type from the family, programmatically duplicate it and modify the new type's parameters as you like.

#### Type Catalogue Parameters

**Question:**
I have three questions related to type catalogues:

1. The type catalogues created by Revit (using the UI button) have on the first line parameter names plus some extra information, e.g., `Type##OTHER##` or `Thickness##LENGTH##MILLIMETERS`.
   What is the logic how that extra information is added and from where?
2. When I look at the family parameters accessible from the family manager, I see some that are not present in the type catalogue. How do we know which parameters are and are not written to the type catalogue?
3. Is there a validation mechanism for a type catalogue? How to check if is valid or not?

**Answer:**
This information is about standard end user content creation and not at all Revit API related.

Therefore, you can probably find the answers yourself by researching those topics from an end user point of view.

One place to start is obviously the related
[documentation on type catalogues and their syntax](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-FFA71D72-D4C5-416D-BF65-1757657C3CE9) – by the way, interestingly enough, the same help id also applies for
[LT](http://help.autodesk.com/view/RVTLT/2016/ENU/?guid=GUID-FFA71D72-D4C5-416D-BF65-1757657C3CE9).

I think this should help with your issues #1 and #2.

Regarding #3: it is indeed possible to create a poorly defined type catalogue.

That will get some sort of warning or error when you try to load such a family.

I am not aware of any other way to check or validate via API or otherwise.

Loading a family with a type catalog displays the dialogue 'Specify Types':

![Specify Types](img/specify_types.png)

The grid under the types section is driven directly from the type catalog.
If the type catalog includes additional parameters that are not in the family, a warning is displayed and they are ignored.

#### Type Catalogue Units

**Question:**
we're trying to create type catalogue in txt file for Radiator family. We were trying to define a parameter in watt units, with no success. How to define header for this parameter?

**Answer:**
Look at page 15 of Martin Schmid's Autodesk University
class handout on *Creating Revit MEP Content for Engineering Coordination*.

Here is a sample Exhaust Fan [RFA](zip/Exhaust_Fan.rfa) and [TXT](zip/Exhaust_Fan.txt) file for you to play with.

They don't use Watts, but should give the idea if you are not already familiar with type catalogues.

Also, the families guide provides a list of the parameter types, namely, for electrical\_power: watts, kilowatts, british\_thermal\_units\_per\_second, british\_thermal\_units\_per\_hour, calories\_per\_second, kilocalories\_per\_second, volt\_amperes, kilovolt\_amperes, horsepower.

Thus, the header definition for such a parameter would be:

```
[paramname]##electrical_power##watts
```