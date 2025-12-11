---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: code_example
optimization_date: '2025-12-11T11:44:14.616454'
original_url: https://thebuildingcoder.typepad.com/blog/0799_running_language.html
post_number: 0799
reading_time_minutes: 4
series: general
slug: running_language
source_file: 0799_running_language.htm
tags:
- csharp
- elements
- geometry
- python
- revit-api
- views
title: Running Language Code and More Exporters
word_count: 767
---

### Running Language Code and More Exporters

An optimal add-in consists of a completely language independent core application with additional localised resources.

Localised resources for different languages can be identified by language tags, e.g. the hexadecimal LCID or language code identifier used by the
[Revit Product GUID](http://thebuildingcoder.typepad.com/blog/2011/04/product-guids-for-revit-2012-and-forever-more.html).
Here is a more
[complete overview](http://en.wikipedia.org/wiki/BCP_47) of
several different language tag systems.

**Question:** Is it possible to determine the language of Revit via the API at run-time?

For example, when you click F1 for help when hovering over a ribbon control, it takes you to a URL made up of the command help ID, application, year, and language code, e.g.:

- [http://wikihelp.autodesk.com/**enu**?adskContextId=HID\_SITE\_TOPO\_SURFACE&product=Revit&release=2013&language=**enu**](http://wikihelp.autodesk.com/enu?adskContextId=HID_SITE_TOPO_SURFACE&product=Revit&release=2013&language=enu)

I would like to determine the correct language code for the running version of Revit in order to provide version and language specific help for my own add-in.

How can this be achieved, please?

**Answer:** There is no direct API to obtain the language code, but you can query the corresponding LanguageType enumeration value from the ControlledApplication Language property. The ControlledApplication instance is provided by the UIControlledApplication ControlledApplication property in the external application OnStartup method.

Here is a helper method to convert from the LanguageType enumeration value to the corresponding three-letter language code:
```csharp
string LanguageCode( LanguageType lt )
{
  switch( lt )
  {
    case LanguageType.Chinese\_Simplified:
      return "CHS";

    case LanguageType.Chinese\_Traditional:
      return "CHT";

    case LanguageType.Czech:
      return "CSY";

    case LanguageType.German:
      return "DEU";

    case LanguageType.English\_USA:
      return "ENU";

    case LanguageType.Spanish:
      return "ESP";

    case LanguageType.French:
      return "FRA";

    case LanguageType.Hungarian:
      return "HUN";

    case LanguageType.Italian:
      return "ITA";

    case LanguageType.Japanese:
      return "JPN";

    case LanguageType.Korean:
      return "KOR";

    case LanguageType.Polish:
      return "PLK";

    case LanguageType.Brazilian\_Portuguese:
      return "PTB";

    case LanguageType.Russian:
      return "RUS";

    case LanguageType.Dutch:
      return "NLD";
  }
  return null;
}
```

A minimal OnStartup method making use of it might look like this:
```python
public Result OnStartup(
  UIControlledApplication a )
{
  LanguageType lt
    = a.ControlledApplication.Language;

  string language\_code = LanguageCode( lt );

  Debug.Print( "Revit language code is "
    + language\_code + "." );

  return Result.Succeeded;
}
```

This prints the following message on my English version of Revit:

```
Revit language code is ENU.
```

For completeness' sake, here is
[RunningLanguage.zip](zip/RunningLanguage.zip) containing
the full source code, Visual Studio solution and add-in manifest of an albeit trivial little add-in exercising this.

Thank you to Martin Schmid and Scott Conover for exploring this issue.

#### More Revit Model Exporters

To follow up the discussion of my quick and dirty Revit model
[OBJ exporter](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-transparency-support.html),
I want to point out that [Adam Nagy](http://adndevblog.typepad.com/aec/adam-nagy.html) completed
his series of posts on the
[AEC DevBlog](http://adndevblog.typepad.com/aec) on
a Revit model exporter and viewer for iOS using an even more minimalistic custom data format for uploading to the cloud and viewing on an iOS mobile device:

- [Part 1](http://adndevblog.typepad.com/aec/2012/06/revit-model-viewer-for-ios-part-1.html): Revit add-in to upload geometry data to a storage service
- [Part 2](http://adndevblog.typepad.com/aec/2012/06/revit-model-viewer-for-ios-part-2.html): An iOS application to download and display the model using OpenGL
- [Part 3](http://adndevblog.typepad.com/aec/2012/06/revit-model-viewer-for-ios-part-3.html): Interactive view orientation and manipulation using gestures

While the OBJ format I looked at is more heavy-weight than Adam's minimal custom format, my implementation includes some other enhancements which make it quite effective as well.

I have also heard of other home-grown viewer implementations with some support for switchback, individual element tagging and object identification based on VRML and on the
[Unity gaming engine](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-devlab.html).

If you are interested in a finely tuned exporter with more complete coverage and control over what gets exported, you might want to take a look at the open source
[STL exporter](http://thebuildingcoder.typepad.com/blog/2011/06/revit-2012-stl-exporter-released-as-open-source.html).

Finally, for high-end exporter requirements, the
[Revit IFC exporter](http://thebuildingcoder.typepad.com/blog/2011/09/revit-ifc-exporter-released-as-open-source.html) is
also open source.

So you have lots of options here!