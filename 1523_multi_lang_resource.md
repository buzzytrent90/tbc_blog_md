---
post_number: "1523"
title: "Multi Lang Resource"
slug: "multi_lang_resource"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'sheets']
source_file: "1523_multi_lang_resource.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1523_multi_lang_resource.html"
---

### Multiple Language RESX Resource Files
A large contribution today from Andrey Bushman, and a couple of upcoming Forge events:
- [Supporting multiple language resource files](#2)
- [Creating and using localised resource `RESX` files](#3)
- [Upcoming Forge accelerators](#4)
#### Supporting Multiple Language Resource Files
Andrey Bushman raised another interesting issue in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [why the '/language' key switches `CurrentCulture` instead of `CurrentUICulture`](http://forums.autodesk.com/t5/revit-api-forum/why-the-language-key-switches-currentculture-instead-of/m-p/6837625)
In the course of our discussion, Andrey pointed out his highly interesting blog and shared a fully functional Revit add-in demonstrating how to support multi-language resources with all conceivable frills, bells and whistles:
- [Andrey's Revit programming notes blog](https://revit-addins.blogspot.ru/2017/01/revit-201711.html) (in Russian)
- [RevitMultiLanguageAddInExample GitHub repository](https://github.com/Andrey-Bushman/RevitMultiLanguageAddInExample) (in C# .NET)
If, like me, your command of Russian is limited, it helps to switch
on [automatic translation](https://chrome.google.com/webstore/detail/google-translate/aapbdbdomjkkjkaonfhkkikfgjllcleb) for
the Russian blog :-)
By the way, this add-in obviously also makes use of
Andrey's [NuGet Revit API package](http://thebuildingcoder.typepad.com/blog/2016/12/nuget-revit-api-package.html),
now updated to support the recent additional Revit 2017.X.Y releases.
Andrey's reason for raising the thread in the first place was a weird behaviour setting the UI culture in Revit 2017.1.1, which I passed on to the development team for further exploration.
However, Andrey provides a workaround for that too, in the module
[RevitPatches.cs](https://github.com/Andrey-Bushman/RevitMultiLanguageAddInExample/blob/master/RevitMultiLanguageAddInExample/RevitPatches.cs):
```csharp
span style="color:blue;">public static class RevitPatches
{
///
/// This patch fixes the bug of localization switching
/// for Revit 2017.1.1. It switches the 'Thread
/// .CurrentThread.CurrentUICulture' and 'Thread
/// .CurrentThread.CurrentCulture' properties according
/// the UI localization of Revit current session.
/// summary>
/// The target language.param>
public static void PatchCultures( LanguageType lang )
{
if( !Enum.IsDefined( typeof( LanguageType ), lang ) )
{
throw new ArgumentException( nameof( lang ) );
}
string language;
switch( lang )
{
case LanguageType.Unknown:
language = "";
break;
case LanguageType.English_USA:
language = "en-US";
break;
case LanguageType.German:
language = "de-DE";
break;
case LanguageType.Spanish:
language = "es-ES";
break;
case LanguageType.French:
language = "fr-FR";
break;
case LanguageType.Italian:
language = "it-IT";
break;
case LanguageType.Dutch:
language = "nl-BE";
break;
case LanguageType.Chinese_Simplified:
language = "zh-CHS";
break;
case LanguageType.Chinese_Traditional:
language = "zh-CHT";
break;
case LanguageType.Japanese:
language = "ja-JP";
break;
case LanguageType.Korean:
language = "ko-KR";
break;
case LanguageType.Russian:
language = "ru-RU";
break;
case LanguageType.Czech:
language = "cs-CZ";
break;
case LanguageType.Polish:
language = "pl-PL";
break;
case LanguageType.Hungarian:
language = "hu-HU";
break;
case LanguageType.Brazilian_Portuguese:
language = "pt-BR";
break;
default:
// Maybe new value of the enum hasn't own
// `case`...
throw new ArgumentException( nameof( lang ) );
}
CultureInfo ui_culture = new CultureInfo( language );
CultureInfo culture = new CultureInfo( language );
Thread.CurrentThread.CurrentUICulture = ui_culture;
Thread.CurrentThread.CurrentCulture = culture;
}
}
```
Here is a summary of our discussion consisting mainly of Andrey's explanation:
My external application (add-in) has two localizations: English (by default) and Russian. I now noticed that my external application (add-in) uses wrong localization on some computers. One of those computers has a 'clear' Revit without any other installed custom add-ins.
If I launch `revit.exe` with the `/language RUS` command line option, then the Revit UI is Russian, but my add-in UI still uses the default localization (i.e. "en") instead of "ru".
I extract the localized resources through the `ResourcesManager` class in my code. That is the native way to work with resources in .NET. I know that `ResourcesManager` chooses the localized resources based on the value of `Thread.CurrentThread.CurrentUICulture`.
Therefore, I assumed that this value isn't `"ru"` or `"ru-RU"`, for some reason, despite the `/language RUS` key. I checked and confirmed that assumption (look my code below): the `Thread.CurrentThread.CurrentUICulture` property is set to `"en"` instead of `"ru"`. Also, I see that `Thread.CurrentThread.CurrentCulture` uses `"ru-RU"` instead of `"en"`.
Why does the `/language` key not switch the setting of `Thread.CurrentThread.CurrentUICulture` responsible for the user interface (UI)?
Is it bug?
```csharp
Result IExternalApplication.OnStartup(
UIControlledApplication application )
{
// ...
// I use the `/language RUS` key for revit.exe
// But I see that my add-in user the 'default'
// localization instead of 'ru'. Hm...
//
// Ok,I will check the CurrentCulture and
// CurrentUICulture values...
//
// Oops... Localization was changed by the
// `/language RUS` key, but not for that thread
// which shall to have this change! Why???
CultureInfo n = Thread.CurrentThread.CurrentCulture; // ru-RU
CultureInfo m = Thread.CurrentThread.CurrentUICulture; // en
// I can fix this problem myself:
// The CurrentUICulture switching fixes the
// problem, but why such problem occurs in Revit?
CultureInfo k = new CultureInfo( "ru" );
Thread.CurrentThread.CurrentUICulture = k;
// Now my add-in uses right localization.
// ...
}
```
Later, I found that this problem exists in Revit 2017.1.1, but this problem is absent in Revit 2017.
For demonstration of this problem I created the project and published it on bitbucket,
in [Andrey-Bushman/research_of_revit_culture_switching_problem](https://bitbucket.org/Andrey-Bushman/research_of_revit_culture_switching_problem).
You can fix this problem in Revit 2017.1.1 by using a class like this:
```csharp
public sealed class UICultureSwitcher : IDisposable
{
CultureInfo previous;
public UICultureSwitcher()
{
CultureInfo culture = new CultureInfo( Thread
.CurrentThread.CurrentCulture.Name );
previous = Thread.CurrentThread.CurrentUICulture;
Thread.CurrentThread.CurrentUICulture = culture;
}
void IDisposable.Dispose()
{
Thread.CurrentThread.CurrentUICulture = previous;
}
}
```
Place code of methods of each external command and external application inside a `using` block with this class member initializing, and the problem will be fixed.
Expanded info and the patch are provided in the [Russian blog post on Revit 2017.1.1](https://revit-addins.blogspot.ru/2017/01/revit-201711.html).
I also created a multi-language add-in example, published in
the [RevitMultiLanguageAddInExample GitHub repository](https://github.com/Andrey-Bushman/RevitMultiLanguageAddInExample).
When you use the `/language RUS` key the UI of my add-in, its tooltip, expanded tooltip, and help file displayed on pressing F1 are in Russian.
When you use `/language ENU` key the UI of my add-in, its tooltip, expanded tooltip, and help file displayed on pressing F1 are in English.
The same applies for the TaskDialog content.
The expected correct behaviour is shown in the
[two-minute video on Revit lang bug](https://www.youtube.com/watch?v=f1IBVgP7T1I):

Many thanks to Andrey for this beautiful and illuminating sample!
#### Creating and Using Localised Resource `RESX` Files
Andrey just published
an [eight-minute video tutorial on resx using](https://www.youtube.com/watch?v=DKCm3p9Gp9M) explaining
the simplest way to create and make use of localized add-in resources.

Additional information and expanded code examples are available from his Russian blog:
- [Binding commands to the help system](https://revit-addins.blogspot.ru/2017/01/blog-post_28.html)
- [Fix Revit 2017.1.1 localisation issue](https://revit-addins.blogspot.ru/2017/01/revit-201711.html)
#### Upcoming Forge Accelerators
We have several [Forge accelerators](http://autodeskcloudaccelerator.com/) coming up
in the [next couple of months](http://autodeskcloudaccelerator.com/prague-2/):
- San Francisco, USA – March 6-10
- Gothenburg, Sweden – March 27-30
- Barcelona, Spain – June 12-16
![Upcoming Forge accelerators](img/2017-02_upcoming_accelerators.png)
I am planning on attending the two European ones and would love to see you there too.
Before that, however, the San Francisco accelerator provides the very next chance to attend – and the early bird gets the worm – so grab you chance while you can!
![Bird with worm](img/bird_with_worm.png)