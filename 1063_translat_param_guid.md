---
post_number: "1063"
title: "Translated Shared Parameter GUID Consolidation"
slug: "translat_param_guid"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'revit-api', 'transactions']
source_file: "1063_translat_param_guid.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1063_translat_param_guid.html"
---

### Translated Shared Parameter GUID Consolidation

Obviously, a language independent application is much easier to implement and maintain than a language dependent one, once you have the internationalisation framework for it in place.

By the way, I took this opportunity to add the new category
[I18n](http://en.wikipedia.org/wiki/Internationalization_and_localization)
to this blog to collect these kind of topics.

The Revit API offers support for creating language independent add-ins, e.g. by using the built-in parameter and category enumeration values instead of localised strings.

For the inverse operation, the LabelUtils and other classes can be used to retrieve localised strings to display in the user interface.

Unfortunately, the whole system of parameters was conceived directly for end users, not applications, and provides little language independence support.

Here is a description of an attempt to address one aspect of this, an arising issue and its solution by Jasper Desmet, who recently also described the
[heat load calculation space adjacency algorithm](http://thebuildingcoder.typepad.com/blog/2013/07/football-and-space-adjacency-for-heat-load-calculation.html) for
his BIM based master thesis.

Situation of the problem: the subject is an application that translates a library of families, according to a translation map (extracted from an Excel-file, but this bears no relevance).
To fully translate the families, all language dependent names in the shared parameter file need translation as well, e.g., to replace the Dutch shared parameter labels with English or French ones.

This is the root of the trouble: the translated shared parameter file is using the same GUIDs for the parameters, so only the Name of the shared parameter changes (this has to do with tagging, that goes awry otherwise).
But as the GUID of the translated shared parameter is the same, we found that the Document.FamilyManager.ReplaceParameter method did ***not*** replace the old parameter with the new translated one.
It was only later we found out this was due to the GUID being the same.
I expect that the ReplaceParameter method checks the GUID before replacing, and if it’s the same it assumes the parameter to be the same.

We constructed a workaround, where the shared parameter file is translated twice: once with other GUID’s (a dummy shared parameter file), and once with the same GUID’s.
This enables us to replace the FamilyParameter with the dummy shared parameter first, then replace that with the translated shared parameter with the same GUID.

So when a parameter needs adaptation, but keeping the same GUID for some reason, this workaround is satisfactory.
I have no idea as to repercussions for performance.

Code:

```csharp
  private static void AdaptSharedParameterFile(
    Application app,
    DefinitionFile originalFile )
  {
    // Build your new (translated/adapted)
    // Definition file  using the Definitions.Create(
    // String, ParameterType, bool, GUID) method to
    // create the shared parameter(s) and the
    // Definition.GUID to obtain the current Shared
    // Parameter GUID
  }

  private static void DummySharedParameterFile(
    Application app,
    DefinitionFile originalFile )
  {
    // Build your dummy Definition file, with
    // different GUIDs using the Definitions.Create(
    // String, ParameterType) method to create the
    // shared parameter(s)
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    Application app;
    Document doc;
    DefinitionFile originalFile;

    AdaptSharedParameterFile( app, originalFile );
    DummySharedParameterFile( app, originalFile );

    using( Transaction t = new Transaction( doc ) )
    {
      t.Start( "Replace Parameter" );

      // Query the required Family Parameter

      FamilyParameter fp = myFamilyParameter;

      // Open the dummy shared parameter file and
      // retrieve the required external definition

      app.SharedParametersFilename
        = sharedParameterFile;

      DefinitionFile dummyDefFile
        = app.OpenSharedParameterFile();

      ExternalDefinition dummyDef
        = dummyDefFile.Groups.get\_Item( myGroup.Name )
          .Definitions.get\_Item( myAdaptedName )
            as ExternalDefinition;

      // Replace the parameter with the dummy
      // parameter to change the GUID

      doc.FamilyManager.ReplaceParameter( fp,
        dummyDef, fp.Definition.ParameterGroup,
        fp.IsInstance );

      // Open the adapted shared parameter file

      app.SharedParametersFilename
        = sharedParameterFile;

      DefinitionFile targetDefFile
        = app.OpenSharedParameterFile();

      ExternalDefinition targetDef
        = targetDefFile.Groups.get\_Item( myGroup.Name )
          .Definitions.get\_Item( myAdaptedName )
            as ExternalDefinition;

      // Replace the dummy-parameter with the
      // adapted parameter with the same GUID
      // as the old parameter

      doc.FamilyManager.ReplaceParameter( fp,
        targetDef, fp.Definition.ParameterGroup,
        fp.IsInstance );

      t.Commit();
  }
```

Many thanks to Jasper for this interesting exploration and workaround!