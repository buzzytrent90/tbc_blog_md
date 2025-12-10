---
post_number: "0720"
title: "Access Wire Sizes"
slug: "access_wire_sizes"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'python', 'references', 'revit-api', 'windows']
source_file: "0720_access_wire_sizes.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0720_access_wire_sizes.html"
---

### Access Wire Sizes

Here is an interesting question that came up regarding the access to wire sizes.
Using a filtered element collector to retrieve a TemperatureRatingType instance and attempting to read its wire sizes produces a problem that is in fact documented in the Revit API help file RevitAPI.chm, but the workaround is still worth pointing out.

**Question:** I'm trying to access the wire size settings in a Revit model without success.

The following code throws an exception saying "Object reference not set to an instance of an object" from the call to the TemperatureRatingType WireSizes property:
```python
  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .OfClass( typeof( TemperatureRatingType ) );

  IEnumerable<TemperatureRatingType> trEnumerable
    = col.ToElements().Cast<TemperatureRatingType>();

  var TempRatingReturn
    = from rtelement in trEnumerable
      where rtelement.Name == "60"
      select rtelement;

  foreach( TemperatureRatingType rt
    in TempRatingReturn )
  {
    WireSizeSet wsset = rt.WireSizes;
  }
```

**Answer:** You are absolutely right, and this is actually the documented and expected behaviour, although it may not be completely obvious from the API documentation.
The description of the TemperatureRatingType class states the following:

Temperature rating type is defined based on corresponding wire material type.
It includes type information such as wire size, insulation type, correction factor, etc.
Only the temperature rating types which are retrieved from WireMaterialType can work well, so don't retrieve it from Revit document directly.

What this actually means is that the TemperatureRatingType object is not supported when retrieved through an ElementFilter.
It can be retrieved from the document electrical settings WireMaterialTypes collection instead.
In fact, you **must** retrieve it from there in able to access the wire sizes.
This may seem odd, but is currently a fact.
The workaround is thus very simple:
```csharp
  foreach( WireMaterialType wt in doc.Settings
    .ElectricalSetting.WireMaterialTypes )
  {
    foreach( TemperatureRatingType trt in
      wt.TemperatureRatings )
    {
      WireSizeSet ws = trt.WireSizes;
    }
  }
```

I implemented an add-in named ReadWireSizes which demonstrates both the exception being thrown and the workaround to retrieve the desired wire size settings from the electrical settings instead.

Here is the complete implementation of the external command Execute method:
```python
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  // Retrieve TemperatureRatingType from
  // filtered element collector; their
  // wire sizes cannot be accessed:

  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .OfClass( typeof( TemperatureRatingType ) );

  IEnumerable<TemperatureRatingType> trEnumerable
    = col.ToElements().Cast<TemperatureRatingType>();

  var TempRatingReturn
    = from rtelement in trEnumerable
      where rtelement.Name == "60"
      select rtelement;

  // The wire sizes we are searching for:

  WireSizeSet wsset = null;

  // Keep track of the target TemperatureRatingType:

  ElementId rtId = null;

  foreach( TemperatureRatingType rt
    in TempRatingReturn )
  {
    try
    {
      rtId = rt.Id;

      // This call fails:

      wsset = rt.WireSizes;
    }
    catch( NullReferenceException ex )
    {
      Debug.Print( ex.GetType().ToString()
        + ": " + ex.Message );

      Debug.Assert( ex.Message.Equals(
        "Object reference not set to an instance of an object." ),
        "expected null reference exception" );
    }
  }

  // Iterate over the electrical settings
  // wire material types and use the target
  // element id to identify the proper
  // TemperatureRatingType:

  WireMaterialTypeSet wts = doc.Settings
    .ElectricalSetting.WireMaterialTypes;

  foreach( WireMaterialType wt in wts )
  {
    foreach( TemperatureRatingType rt
      in wt.TemperatureRatings )
    {
      if( rtId == rt.Id )
      {
        wsset = rt.WireSizes;
      }
    }
  }

  int n = wsset.Size;

  Debug.Print( string.Format(
    "{0} wire size{1} stored on temperature "
    + "rating type with element id {2}",
    n, ( 1 == n ? "" : "s" ), rtId ) );

  return Result.Succeeded;
```

Running this command in the rme\_advanced\_sample\_project.rvt sample project provided with Revit 2012 MEP generates the following output in the Visual Studio debug output window:

```
A first chance exception of type 'System
  .NullReferenceException' occurred in RevitAPI.dll
System.NullReferenceException: Object reference
  not set to an instance of an object.
28 wire sizes stored on temperature rating type
  with element id 293221
```

Here is
[ReadWireSizes.zip](zip/ReadWireSizes.zip) containing
the complete source code, Visual Studio solution and add-in manifest for this external command.