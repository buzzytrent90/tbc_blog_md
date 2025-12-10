---
post_number: "1032"
title: "Programmatic Access to Duct Sizes"
slug: "duct_sizes"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'windows']
source_file: "1032_duct_sizes.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1032_duct_sizes.html"
---

### Programmatic Access to Duct Sizes

Here is a nice little MEP related blog post on accessing duct sizes through the API to round off the week.

It was provided by our MEP expert Martin Schmid, who says:

> I recently heard from a couple of developers that they had difficulties accessing the Duct Size settings in Revit.
>
> In the user interface, you can see them by selecting Manage > MEP Settings > Mechanical Settings > Duct Settings > Round/Oval/Rectangular.
>
> I threw together a macro to check for myself... maybe this would be helpful to publish so the results turn up in web searches...

I think this is a great idea and completely agree, so I implemented an external command ListDuctSizes calling Martin's method of the same name to share with you:

```csharp
  /// <summary>
  /// List all duct sizes, thus proving that the duct
  /// size settings available in the Revit UI through
  /// Manage > MEP Settings > Mechanical Settings >
  /// Duct Settings > Round/Oval/Rectangular are
  /// indeed available via the API.
  /// </summary>
  void ListDuctSizes( Document doc )
  {
    DuctSizeSettings settings
      = DuctSizeSettings.GetDuctSizeSettings( doc );

    foreach( KeyValuePair<DuctShape, DuctSizes> pair
      in settings )
    {
      Debug.Print( pair.Key.ToString() );

      foreach( MEPSize size in pair.Value )
      {
        string value = FormatUtils.Format( doc,
          UnitType.UT\_HVAC\_DuctSize,
          size.NominalDiameter );

        Debug.Print(
          "  {0}: used in size lists/sizing: {1}/{2}",
          value,
          size.UsedInSizeLists.ToString(),
          size.UsedInSizing.ToString() );
      }
    }
  }
```

Here is some of the output that it generates in the Visual Studio debug output window:

```
Round
  3": used in size lists/sizing: True/True
  4": used in size lists/sizing: True/True
  4": used in size lists/sizing: True/True
  ...
  88": used in size lists/sizing: True/True
  90": used in size lists/sizing: True/True
Rectangular
  3": used in size lists/sizing: True/True
  4": used in size lists/sizing: True/True
  4": used in size lists/sizing: True/True
  ...
  94": used in size lists/sizing: True/True
  96": used in size lists/sizing: True/True
Oval
  3": used in size lists/sizing: True/True
  4": used in size lists/sizing: True/True
  5": used in size lists/sizing: True/True
  ...
  143": used in size lists/sizing: True/True
  144": used in size lists/sizing: True/True
```

For the sake of completeness, here is
[ListDuctSizes.zip](zip/ListDuctSizes.zip) containing
the full source code, Visual Studio solution and add-in manifest for this simple command.

Many thanks to Martin for sharing this, and a happy weekend to all!

####