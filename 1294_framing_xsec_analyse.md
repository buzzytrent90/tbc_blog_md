---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.702659'
original_url: https://thebuildingcoder.typepad.com/blog/1294_framing_xsec_analyse.html
post_number: '1294'
reading_time_minutes: 9
series: general
slug: framing_xsec_analyse
source_file: 1294_framing_xsec_analyse.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- parameters
- references
- revit-api
- selection
- views
- walls
- windows
title: Framing Cross Section Analyser and REX in Revit 2015
word_count: 1857
---

### Framing Cross Section Analyser and REX in Revit 2015

Back in December 2013, I discussed
[structural cross section analysis](http://thebuildingcoder.typepad.com/blog/2013/12/security-framing-cross-section-analyser-and-rex.html),
i.e. determination of the cross section profile of beam, columns, braces, etc., and
[several completely different approaches](http://thebuildingcoder.typepad.com/blog/2013/12/security-framing-cross-section-analyser-and-rex.html#3) one
can take to achieve that.

I also demonstrated how to make use of the powerful functionality provided by the REX toolkit without building the entire add-in on top of the REX framework.

By the way, here are some other earlier discussions of REX:

- [The REX SDK](http://thebuildingcoder.typepad.com/blog/2011/04/the-rex-sdk.html)
- [Extensions for Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/06/extensions-for-revit-2012.html)
- [REX Content Generator](http://thebuildingcoder.typepad.com/blog/2011/12/rex-content-generator.html)

Before getting to this Revit Structure stuff, let me also mention that I made one of my rather infrequent contributions to the
[AEC DevBlog](http://adndevblog.typepad.com/aec) today as well, to explain that about
[using AutoCAD MEP object enablers in AcCoreConsole](http://adndevblog.typepad.com/aec/2015/03/using-autocad-mep-objects-in-accoreconsole.html).

Anyway, back to the cross section analysis; that discussion led to the following question:

**Question:**
I would like to create a Revit 2015 REX add-in without using the REX add-in wizard.

I looked at your
[FramingXsecAnalyzer add-in](https://github.com/jeremytammik/FramingXsecAnalyzer)
and
[the post](http://thebuildingcoder.typepad.com/blog/2013/12/security-framing-cross-section-analyser-and-rex.html) describing
it, but don't understand how to start and create a new such project for my own use.

Do you have any sample project I could start with?

**Answer:**
The starting point is a completely normal Revit add-in project.

You add the required additional references to that and ensure that the REX framework is found and loaded dynamically.

To migrate the application to a new version, you can start by looking at the 2012 and 2014 versions in the
[download section](http://thebuildingcoder.typepad.com/blog/2013/12/security-framing-cross-section-analyser-and-rex.html#10) of that post.

You can use the GitHub comparison tools to determine what changed between them, e.g.
[FramingXsecAnalyzer/compare/2012.0.0.0...2014.0.0.0](https://github.com/jeremytammik/FramingXsecAnalyzer/compare/2012.0.0.0...2014.0.0.0).

I migrated the project to Revit 2015 for you.

One important step was obviously to increment the version number in the assembly resolver.

At the beginning of the external command Execute method, we register it like this:

```csharp
  AppDomain.CurrentDomain.AssemblyResolve
    += new ResolveEventHandler( OnAssemblyResolve );
```

The resolver specifies the assembly version to load:

```csharp
  static System.Reflection.Assembly OnAssemblyResolve(
    object sender,
    ResolveEventArgs args )
  {
    Assembly a = Assembly.GetExecutingAssembly();

    return Autodesk.REX.Framework.REXAssemblies
      .Resolve( sender, args, "2015", a );
  }
```

The code actually making use of the REX functionality remains completely unchanged:

```csharp
  /// <summary>
  /// Use REX to analyse element cross section.
  /// This requires a reference to
  /// REX.ContentGeneratorLT.dll and prior
  /// initialisation of the REX framework.
  /// The converter initialisation must reside in
  /// a different method than the subscription to
  /// the assembly resolver OnAssemblyResolve.
  /// </summary>
  void RexXsecAnalyis(
    ExternalCommandData commandData,
    Element e )
  {
    // Initialise converter

    RVTFamilyConverter rvt = new RVTFamilyConverter(
      commandData, true );

    // Retrieve family type

    REXFamilyType fam = rvt.GetFamily( e,
      ECategoryType.SECTION\_PARAM );

    // Retrieve section data

    REXFamilyType\_ParamSection paramSection = fam
      as REXFamilyType\_ParamSection;

    REXSectionParamDescription parameters
      = paramSection.Parameters;

    // Extract dimensions, section type, tapered
    // predicate, etc.
    // If different start and end sections are
    // required, use DimensionsEnd as well.

    REXSectionParamDimensions dimensions = parameters
      .Dimensions;

    ESectionType sectionType = parameters
      .SectionType;

    bool tapered = parameters.Tapered;

    bool start = true;

    Contour\_Section contour = parameters.GetContour(
      start );

    List<ContourCont> shape = contour.Shape;

    Util.InfoMessage( string.Format(
      "The selected structural framing element "
      + "cross section REX section type is "
      + "{0}.", sectionType ) );
  }
```

As always, the current version is available from the
[FramingXsecAnalyzer GitHub repository](https://github.com/jeremytammik/FramingXsecAnalyzer),
and the version discussed here is
[release 2015.0.0.0](https://github.com/jeremytammik/FramingXsecAnalyzer/releases/tag/2015.0.0.0).

This kind of interactive parameter selection and filtering may obviously be very useful for many other applications as well.

I tested it on the following I column:

![I cross section](img/framing_xsec_i_2015.png)

It reports:

```
The selected structural framing element cross section
section view cut plane face has 1 loop and is thus 'open'.

The selected structural framing element cross section
REX section type is I.
```

The REX modules are demand loaded between these two log messages.

Looking in the Visual Studio debug output window, you can see the following activity going on in the background:

```
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Windows\Microsoft.Net\assembly\GAC_MSIL\Autodesk.REX.Framework\v4.0_2014.0.0.0__51e16e3b26b42eda\Autodesk.REX.Framework.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Windows\assembly\GAC_MSIL\Autodesk.Common.AResourcesControl\1.0.0.0__ff3304d4f320ee59\Autodesk.Common.AResourcesControl.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Components\AREXContentGenerator\REX.ContentGeneratorLT.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Windows\Microsoft.Net\assembly\GAC_MSIL\Autodesk.REX.Framework\v4.0_2015.0.0.0__51e16e3b26b42eda\Autodesk.REX.Framework.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Foundation.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Foundation.Forms.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Products\Revit\AREXRevitMngr.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Geometry.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.Engine.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.Preferences.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.System.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.UI.WPF.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.UI.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.UI.Forms.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\en-US\REX.UI.resources.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\en-US\REX.System.resources.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.Mathematics.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Geometry.Structure.dll'
'Revit.exe' (Managed (v4.0.30319)): Loaded 'C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Geometry.Revit.dll'
```

Copy to a text editor or view the source to see the truncated lines in full.

Here is a list of the modules that were loaded:

- C:\Windows\Microsoft.Net\assembly\GAC\_MSIL\Autodesk.REX.Framework\v4.0\_2014.0.0.0\_\_51e16e3b26b42eda\Autodesk.REX.Framework.dll
- C:\Windows\assembly\GAC\_MSIL\Autodesk.Common.AResourcesControl\1.0.0.0\_\_ff3304d4f320ee59\Autodesk.Common.AResourcesControl.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Components\AREXContentGenerator\REX.ContentGeneratorLT.dll
- C:\Windows\Microsoft.Net\assembly\GAC\_MSIL\Autodesk.REX.Framework\v4.0\_2015.0.0.0\_\_51e16e3b26b42eda\Autodesk.REX.Framework.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Foundation.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Foundation.Forms.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Products\Revit\AREXRevitMngr.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Geometry.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.Engine.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.Preferences.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.System.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.UI.WPF.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.UI.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.UI.Forms.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\en-US\REX.UI.resources.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\en-US\REX.System.resources.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Engine\REX.Mathematics.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Geometry.Structure.dll
- C:\Program Files\Common Files\Autodesk Shared\Extensions 2015\Framework\Foundation\REX.Geometry.Revit.dll

In a second test, I presented it with a tubular column:

![Tube cross section](img/framing_xsec_tube_2015.png)

In this case, it reports:

```
The selected structural framing element cross section
section view cut plane face has 2 loops and is thus 'closed'.

The selected structural framing element cross section
REX section type is TUBE.
```

I hope this helps.

Have fun!

---

### Using AutoCAD MEP Objects in AcCoreConsole

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

**Question:**
Using AcCoreConsole I'd like to move an object via AcCoreConsole.

The object type in question happens to be an AutoCAD MEP MVBlock.

Is this possible?

Would AcCoreConsole recognize the object as valid or would it be a zombie, i.e. a proxy object?

**Answer:**
I checked with the development team and received several confirmations saying that this should work:

AcCoreConsole does recognize MVBlock, and the object can be moved in the same way as in AutoCAD.

The AutoCAD MEP object enablers get loaded on demand in AcCoreConsole.exe.

Here is a test suite proving it:

1. accoreconsole.exe: Just run accoreconsole.exe, following modules were loaded. Please note AEC modules are not loaded:

![AcCoreConsole Object Enablers](img/accoreconsole_1.jpg)

2. accoreconsole.exe /i oneBox.dwg: Open a dwg with only one box. A few more AutoCAD modules get loaded, but no AEC modules:

![AcCoreConsole Object Enablers](img/accoreconsole_2.jpg)

3. accoreconsole.exe /i oneWall.dwg: Open a dwg with only one wall. AEC modules are loaded. Also note that AECB modules are loaded, even though there are no MEP objects in the DWG:

![AcCoreConsole Object Enablers](img/accoreconsole_3.jpg)

Following is the output of list command:

![AcCoreConsole Object Enablers](img/accoreconsole_3b.jpg)

4. accoreconsole.exe /i oneDuct.dwg: Open a dwg with only one duct. AEC and AECB module are loaded:

![AcCoreConsole Object Enablers](img/accoreconsole_4.jpg)

Following is the output of list command:

![AcCoreConsole Object Enablers](img/accoreconsole_4b.jpg)

I hope this helps.