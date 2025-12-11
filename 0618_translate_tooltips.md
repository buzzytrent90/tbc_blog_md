---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.275033'
original_url: https://thebuildingcoder.typepad.com/blog/0618_translate_tooltips.html
post_number: 0618
reading_time_minutes: 5
series: general
slug: translate_tooltips
source_file: 0618_translate_tooltips.htm
tags:
- csharp
- elements
- revit-api
- transactions
- windows
title: Translate Revit Tooltips
word_count: 954
---

### Translate Revit Tooltips

Kean Walmsley recently started exploring the exciting and useful idea of hooking up machine translation to AutoCAD, initially to
[auto-translate tooltips](http://through-the-interface.typepad.com/through_the_interface/2011/07/automatic-translation-of-autocad-tooltips-using-net.html).
As Kean points out, you can get all AutoCAD tooltips translated into one of 35 different languages, which is a great help for people learning or using the product in regions for which we don't currently localize.
Here's a tooltip in Arabic, for instance:

![Arabic AutoCAD tooltip](img/transtips_arabic.png)

Kean quickly followed up the original proof of concept with an
[improved implementation](http://through-the-interface.typepad.com/through_the_interface/2011/07/caching-translations-of-autocad-tooltips-using-net.html) including caching and other features.

Thanks to the fact that many Autodesk products are now sharing a number of basic platform tools, it is possible to use the same tooltip translation mechanism for Revit.

This tool mainly uses AdWindows.dll, the .NET assembly providing the Autodesk.Windows namespace, which provides part of the ribbon user interface implementation.

As Rudolf Honke already pointed out in a
[number of posts on UI Automation](http://thebuildingcoder.typepad.com/blog/automation), the .NET framework provides a lot of powerful functionality for manipulating and hooking into the ribbon.

To catch the tooltips being displayed, we can subscribe to the Autodesk.Windows.ComponentManager.ToolTipOpened event, which is fired for all tooltips.

Here is a snapshot of an initial prototype of the tooltip translation, running in an English version of Revit and causing it to display its tooltips in Italian:

![Italian Revit tooltip](img/transtips_revit_it.png)

The current prototype implementation makes use of a very simplistic front end in which you explicitly specify the source and target languages:

![Source and target language form](img/transtips_revit_dlg.png)

Obviously, in a more foolproof and user friendly version, the source language would be determined automatically from the Revit application, and the target version would at least default to the OS language.

Currently, this prototype for Revit is working quite well.
The source has only small differences to the AutoCAD implementation.
Revit's way to work with tooltips differs slightly from AutoCAD, in that most of the ribbon items need to be resolved on the first run.

We are planning to clean this up some more, which should be pretty straightforward.
Having a separate source file included in both projects is probably enough, and there is no need for a separately compiled component.

Reporting errors is currently handled in AutoCAD by posting to the command line, and in Revit by displaying a task dialogue.
That will need to be resolved, presumably by defining a 'ProcessWebException' or 'display error' call-back, or something.

Obviously, this tool has little to do with the Revit API, actually, and the functionality may possibly even be achievable from a stand-alone external executable.
It does have to load and access the functionality in the AdWindows.dll .NET assembly, though.
The easiest (and maybe only?) way to do that is to implement it as an add-in.

You can translate back into English by setting the target language to English.
The AutoCAD version implements a 'None' option which also does this.

The source and target languages are managed by storing the language code in global class variables.
We have not implemented indexing into the arrays of language names and codes, because these indices would become invalid in offline mode with just the pre-existing or deployed translations on disk to work from.

So, without further ado, let's take a look at the source code for this utility in its current untested form.
First of all, here is the very simple external command implementation:
```csharp
namespace TransTipsRvt
{
  [Transaction( TransactionMode.ReadOnly )]
  public class Command : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      TransTips.Run();

      return Result.Succeeded;
    }
  }
}
```

All the add-in does is run the TransTips class static method Run.
Since it does not touch the Revit database at all, it can use a read-only transaction mode.

The TransTips Run method queries the web service for the available languages, displays the form prompting for the source and target language, and then hijacks the tooltips for translation.
The latter is identical to Kean's implementation:
```csharp
public static void Run()
{
  List<string> codes = GetLanguageCodes();

  List<string> names =
    new List<string>( GetLanguageNames( codes ) );

  // Get the name corresponding to a language code

  string srcName =
    names[codes.FindIndex( x => x.Equals( \_srcLang ) )];

  string trgName =
    names[codes.FindIndex( x => x.Equals( \_trgLang ) )];

  TransTipsForm dlg =
    new TransTipsForm( names, srcName, trgName );

  if( DialogResult.OK == dlg.ShowDialog() )
  {
    \_srcLang = codes[ names.FindIndex(
      x => x.Equals( dlg.SourceLanguage ) ) ];

    \_trgLang = codes[ names.FindIndex(
      x => x.Equals( dlg.TargetLanguage ) ) ];
  }
  HijackTooltips();
}
```

Here is the
[complete Visual Studio solution](zip/TransTipsRvt02.zip), as well as a
[compiled DLL](zip/TransTipsRvt.dll) and
[add-in manifest file](zip/TransTipsRvt.addin),
in case you wish to test this without compiling it yourself.

Here is a last snapshot from my final test run before posting, displaying the 'Modify' tooltip of an English Revit in Korean:
![Korean Revit tooltips](img/transtips_revit_ko.png)

**Addendum:** Please also look at Kean Walmsley's updated single solution which will build
[TransTips for both AutoCAD and Revit](http://through-the-interface.typepad.com/through_the_interface/2011/07/translating-tooltips-in-both-autocad-and-revit-using-net.html) (including
built versions of the DLLs),
[TransTips for AutoCAD, Inventor and Revit](http://through-the-interface.typepad.com/through_the_interface/2011/07/translating-tooltips-inside-autocad-inventor-and-revit-using-net.html), and later for
[AutoCAD, Inventor, Revit and 3ds Max](http://through-the-interface.typepad.com/through_the_interface/2011/08/translating-tooltips-in-autocad-inventor-revit-and-3ds-max-using-net.html).