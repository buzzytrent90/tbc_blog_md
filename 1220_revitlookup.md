---
post_number: "1220"
title: "Past, Future, Frameworks, RevitLookup and Hackathon"
slug: "revitlookup"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'parameters', 'references', 'revit-api', 'selection', 'views']
source_file: "1220_revitlookup.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1220_revitlookup.html"
---

### Past, Future, Frameworks, RevitLookup and Hackathon

Today, let me ask you to test my [RevitLookup future-proofing update](#2) and mention a discussion on the use of the [.NET 4.5 framework for Revit 2013, 2014 and 2015](#3).

I am sitting here writing this among the swans and gulls, squinting in the autumn sunshine on the shore of the Zurich Lake.

Later, I visited the nearby
[Kronenhalle](http://www.kronenhalle.ch),
a rather exclusive restaurant highly praised year in and year out by gourmet Christian Seiler, e.g.
[2011](http://www.christianseiler.com/zurichs-legende.html),
[2012](http://www.christianseiler.com/das-tor-zur-schweiz.html) and
[2014](http://blog.dasmagazin.ch/2014/09/20/kronenhalle-der-schoepfung).

With all that praise I had to take a look and have a last quick bite before diving into the
[HackZurich](http://hackzurich.com) hackathon
for the rest of the weekend.

It turned out not to be my cup of tea, though; dark, too carnivorous, no vegetarian dishes.
I ended up having just a tomato soup, dessert and coffee :-)

#### Future Proofing RevitLookup

I updated RevitLookup to eliminate the compiler warnings due to deprecated and obsolete Revit API usage.

These are the warnings I addressed, with the Selection.Elements one occurring in numerous locations:

- Autodesk.Revit.Creation.Document.NewViewDrafting() is obsolete: This method is obsolete in Revit 2015. Use ViewDrafting.Create() instead.
- Autodesk.Revit.DB.Definitions.Create(string, Autodesk.Revit.DB.ParameterType, bool) is obsolete: This method is deprecated in Revit 2015. Use Create(Autodesk.Revit.DB.ExternalDefinitonCreationOptions) instead
- Autodesk.Revit.DB.Family.Symbols is obsolete: This property is obsolete in Revit 2015. Use Family.GetFamilySymbolIds() instead.
- Autodesk.Revit.DB.FormatOptions.GetName() is obsolete: This method is deprecated in Revit 2015. Use UnitUtils.GetTypeCatalogString(DisplayUnitType) instead.
- Autodesk.Revit.UI.Selection.SelElementSet is obsolete: This class is deprecated in Revit 2015. Use Selection.SetElementIds() and Selection.GetElementIds() instead.
- Autodesk.Revit.UI.Selection.Selection.Elements is obsolete: This property is deprecated in Revit 2015. Use GetElementIds() and SetElementIds instead.

The complete updated source code, Visual Studio solution and add-in manifest is provided in the
[RevitLookup GitHub repository](https://github.com/jeremytammik/RevitLookup),
and the current future-proofed version is stored there as
[release 2015.0.0.2](https://github.com/jeremytammik/RevitLookup/releases/tag/2015.0.0.2).

By the way, I am very surprised that nobody at all has reported any bugs or improvements on the last version.

What's up, guys 'n gals?

Please download, install and use this version to ensure that I did not introduce any errors is it.

And please let me know about any problems, solutions and enhancements.

Thank you!

I had another interesting conversation related to future (and past-) proofing and supporting multiple version add-ins in the past few days:

#### Compiling Using .NET 4.5 for Revit 2013, 2014 and 2015

**Question:** The SDK documentation for Revit 2013 explicitly requires compiling to .NET Framework 4.0, while the Revit 2015 SDK explicitly requires Framework 4.5.

As far as I can tell, the .NET framework 4.5 is
[backwards compatible](http://msdn.microsoft.com/en-us/library/ff602939(v=vs.110).aspx) with
apps built in Framework 4.0.

Therefore, is there any issue with compiling an app for all three Revit versions, 2013, 2014 and 2015, using the 4.5 framework?

**Answer:** It is probably possible to compile a Revit add-in with the .NET 4.5 framework and load it into all three versions of Revit, provided that there have been no changes in any of the Revit API interfaces that it uses.

In fact, I demonstrated a
[multi-version add-in](http://thebuildingcoder.typepad.com/blog/2012/07/multi-version-add-in.html) that
was compiled for both Revit 2012 and 2013.

It even uses reflection to check which of the two it is running in, and access new Revit 2013 API functionality, if available, reverting to implementing equivalent functionality for itself if not.

However, I can give you no guarantees whatsoever, and you are doing this completely at your own risk.

In a way, the question is pointless, since you will have to try it out for yourself anyway.

There is a significant risk that you will run into Revit API functionality that you cannot avoid using and whose signature has changed between these three releases.

Such changes can be handled: every such access will require the use of .NET reflection to determine which API version you need to address and access it appropriately.

Two releases can be easily bridged. Three will be a bit more work.

The standard solution used by developers supporting several versions of Revit is to recompile the add-in for each version, but use one single code base and possibly even define all the different Revit versions as separate targets within one single Visual Studio solution.

The
[multi-version wizard](http://thebuildingcoder.typepad.com/blog/2013/11/multi-version-visual-studio-revit-add-in-wizard.html) generates
a complete Visual Studio solution supporting several Revit target releases for you with a single click.

**Response:** Thanks for your help. Indeed it seems to be working just fine in all three versions when I target 4.5.

It does make use of version-specific Revit API assembly DLLs, since it uses a handful of commands that changed across versions:

- Doc.ActiveView.IsolateElementsTemporary(iCollIds) vs doc.ActiveView.IsolateElementsTemporary(listIds)
- Element.GetMaterialArea(materialId, false) vs element.GetMaterialArea(material)
- Element.GetMaterialVolume(materialId) vs element.GetMaterialVolume(material)
- Element.GetMaterialIds vs element.Materials
- LinkInstance.GetLinkDocument().Title vs ModelPathUtils.ConvertModelPathToUserVisiblePath(linkInstance.GetExternalFileReference().GetAbsolutePath())

What I was more concerned with was whether I needed to change nuget packages as well based on different target .NET frameworks.

Apparently, given my testing thus far, the add-ins for Revit 2013, 2014 and 2015 API can all be compiled, loaded and executed correctly targeting the .NET 4.5 framework.

This is obviously not a 100% sure thing, given that there may be some edge cases from which conflicts arise.

I found it helpful to set up a static class with regions called “Revit2013”, “Revit2014”, and “Revit2015”, each of which contains static methods that are appropriate only to that particular Revit API version. That way, when I change out which Revit API version is referenced, I can just comment in and comment out a few lines. Since it’s just a handful of methods that vary, I can the vast majority of my code base is version agnostic and hence much easier to manage.

**Answer:** Yes, when targeting several different versions with small variations, it is definitely important to isolate and manage the differences well and effectively.

You could also use pre-processor pragmas, e.g. #define REVIT\_2015\_TARGET and #if REVIT\_2015\_TARGET, to switch on and off certain sections of code.

Commenting and uncommenting them sounds a bit error prone to me, or anyway like a manual and thus potentially fallible approach.

The
[multi-version add-in](http://thebuildingcoder.typepad.com/blog/2012/07/multi-version-add-in.html) compiled
for Revit 2012 and 2013 avoids making use of compile-time pragmas, because it differentiates at run time and sets a Boolean variable instead.

The
[multi-version Visual Studio solution wizard](http://thebuildingcoder.typepad.com/blog/2013/11/multi-version-visual-studio-revit-add-in-wizard.html) does
use them, though.

Possibly, a combination of the static classes you describe with pre-processor pragmas could provide the most efficient solution.

As soon as you notice these pragmas being sprinkled around the code all too liberally, it is best to implement a method that isolates the respective code differences into one of the static version-specific helper classes.