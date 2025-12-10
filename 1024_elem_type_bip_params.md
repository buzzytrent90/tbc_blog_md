---
post_number: "1024"
title: "10.000.000.000th Post and Element Type Parameters"
slug: "elem_type_bip_params"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'revit-api', 'selection', 'views', 'walls']
source_file: "1024_elem_type_bip_params.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1024_elem_type_bip_params.html"
---

### 10.000.000.000th Post and Element Type Parameters

This is the 1024th post on The Building Coder, just to top off our recent
[5-year and 1000th post celebration](http://thebuildingcoder.typepad.com/blog/2013/08/happy-birthday-dear-building-coder.html).

The decimal number 1024 equals 2^10, i.e. 10.000.000.000 in binary format, hence the large number of zeroes in the title :-)

I am also still away on
[vacation](http://thebuildingcoder.typepad.com/blog/2013/09/appstore-advice-and-zooming-in-a-preview-control.html#5) and
this is the second post in my absence – or 10th, in binary format :-) – dealing with
[retrieving ElementType parameters](#2), the
[ADN Xtra labs built-in parameter checker](#3) and a
[BipChecker update for Revit 2014](#4).

Meanwhile, I hope you are enjoying the break as much as I am :-)

#### Retrieving ElementType Parameters

I want to present a small enhancement to the built-in parameter checker included in the ADN Xtra labs.

The reason for looking at it again is to answer the following frequently recurring question:

**Question:** I know how to retrieve the element properties form an object, e.g. a column instance, using the Element.Parameters collection.

However, how can I access the column type properties, please?

**Answer:** For a given element E, you can ask for the element id of its type T by calling the GetTypeId method.
Pass that in to the document GetElement method, access the T object instance itself, and retrieve the Element.Parameters collection from that.

#### The ADN Xtra Labs Built-in Parameter Checker

The ADN Xtra labs built-in parameter checker loops over all defined BuiltInParameter enumeration entries and checks to see whether a value can be retrieved for each of the corresponding parameters on a selected element.

Please be aware that an enhanced version of this built-in parameter checker was published as a separate
[BipChecker add-in](http://thebuildingcoder.typepad.com/blog/2011/09/unofficial-parameters-and-bipchecker.html) back
in 2011.
We'll take another look at that below.

The user is prompted to select an element using the
[GetSingleSelectedElementOrPrompt](http://thebuildingcoder.typepad.com/blog/2010/05/pre-post-and-pick-select.html) method,
which supports all conceivable selection facilities, including:

- Pre-selection before launching the command.
- Post-selection after launching the command.
- Entering a numeric element id to select an invisible element.

It achieves this by presenting a small prompt message:

![Element selection prompt](img/bip_select_msg.png)

The prompt is obviously only displayed if no pre-selection was made.

The code also initialises the isSymbol flag to false:

```csharp
  Element e
    = LabUtils.GetSingleSelectedElementOrPrompt(
      uidoc );

  bool isSymbol = false;
```

The previous code was implemented before the introduction of the Element.GetTypeId method, so it just checked for a family instance like this:

```csharp
  //
  // for a family instance, ask user whether to
  // display instance or type parameters;
  // in a similar manner, we could add dedicated
  // switches for Wall --> WallType,
  // Floor --> FloorType etc. ...
  //
  if( e is FamilyInstance )
  {
    FamilyInstance inst = e as FamilyInstance;
    if( null != inst.Symbol )
    {
      string symbol\_name
        = LabUtils.ElementDescription(
          inst.Symbol, true );

      string family\_name
        = LabUtils.ElementDescription(
          inst.Symbol.Family, true );

      string msg =
      "This element is a family instance, so it "
      + "has both type and instance parameters. "
      + "By default, the instance parameters are "
      + "displayed. If you select 'No', the type "
      + "parameters will be displayed instead. "
      + "Would you like to see the instance "
      + "parameters?";

      if( !LabUtils.QuestionMsg( msg ) )
      {
        e = inst.Symbol;
        isSymbol = true;
      }
    }
  }
```

I updated the code to be more generic and handle all kinds of element type relationships by checking whether the GetTypeId method returns a valid element type id like this:

```csharp
  ElementId idType = e.GetTypeId();

  if( ElementId.InvalidElementId != idType )
  {
    // The selected element has a type; ask user
    // whether to display instance or type
    // parameters.

    ElementType typ = doc.GetElement( idType )
      as ElementType;

    Debug.Assert( null != typ,
      "expected to retrieve a valid element type" );

    string type\_name = LabUtils.ElementDescription(
      typ, true );

    string msg =
      "This element has an ElementType, so it has "
      + "both type and instance parameters. By "
      + "default, the instance parameters are "
      + "displayed. If you select 'No', the type "
      + "parameters will be displayed instead. "
      + "Would you like to see the instance "
      + "parameters?";

    if( !LabUtils.QuestionMsg( msg ) )
    {
      e = typ;
      isSymbol = true;
    }
  }
```

If an element that has a valid type assigned to it is selected, e.g. a wall, the code detects this and prompts the user to choose whether to display its instance or type properties:

![Element type message](img/bip_element_type_msg.png)

If instance properties are chosen, the following list of parameters on the wall itself is displayed:

![List of instance parameters](img/bip_instance_params.png)

If type properties are chosen, the parameters are retrieved from the wall type instead:

![List of element type parameters](img/bip_type_params.png)

Here is
[version 2014.0.0.3](zip/adn_labs_2014_3.zip) of
the ADN Training Labs for Revit 2014 including the updated built-in parameter checker.

#### BipChecker Update for Revit 2014

I went on planning to implement the same enhancement in the stand-alone BipChecker add-in, only to discover two things:

1. It has not been updated since its original publication in the year 2011, for Revit 2012.
2. It has already implemented a more sophisticated check than the one I describe above.

To see the more sophisticated check for various kinds of element types implemented by BipChecker, please search for 'CanHaveTypeAssigned' in the
[initial BipChecker publication](http://thebuildingcoder.typepad.com/blog/2011/09/unofficial-parameters-and-bipchecker.html).

I updated it for Revit 2014, fixing some compilation errors and
[disabling the architecture mismatch warning](http://thebuildingcoder.typepad.com/blog/2013/07/recursively-disable-architecture-mismatch-warning.html);
here is
[BipChecker\_2014.zip](zip/BipChecker_2014.zip) containing the new version.

Back to my vacation again...
Meanwhile, I wish you a wonderful time as well!