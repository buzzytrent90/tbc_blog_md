---
post_number: "1331"
title: "CNC Direct – Export Wall Parts to DXF and SAT"
slug: "cnc_direct"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'levels', 'parameters', 'references', 'revit-api', 'selection', 'walls']
source_file: "1331_cnc_direct.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1331_cnc_direct.html"
---

### CNC Direct – Export Wall Parts to DXF and SAT

I implemented the Revit ExportCncFab add-in back in 2013 to export Revit wall parts to DXF or SAT for CNC fabrication.

The full source code, Visual Studio solution and add-in manifest lives in the
[ExportCncFab GitHub repository](https://github.com/jeremytammik/ExportCncFab).

The project was prompted by William Spier, who recently published a nice 12-minute YouTube video
[Revit to CNC Direct](https://www.youtube.com/watch?v=uNJ9RTppqoU) describing and demonstrating its practical use and application:

In William's words: "Asked Jeremy to write a DLL that allows users to export multiple objects as discrete, individually named DXF files.
So doing means you can export native Revit geometry to DXF, resulting in fabrication ready files like gyp wall board, structural steel gussets, or anything you can conceive of that has a level of accuracy that allows it to be CNC ready, as it exists in the Revit model."

Due to popular request, I now migrated this add-in from Revit 2014 to Revit 2015 and on to 2016, documenting all issues encountered en route.

Before getting to that, here are the previous discussions of this project and a couple of its implementation aspects:

- [Export Wall Parts Individually to DXF](http://thebuildingcoder.typepad.com/blog/2013/03/export-wall-parts-individually-to-dxf.html)
- [ExportCncFab on GitHub](http://thebuildingcoder.typepad.com/blog/2013/10/exportcncfab-on-github-and-revitlookup-update.html)
- [Driving CNC Fabrication and Shared Parameters](http://thebuildingcoder.typepad.com/blog/2013/12/driving-cnc-fabrication-and-shared-parameters.html)

Now, let's look at the two migrations in more detail:

#### Migrating ExportCncFab from Revit 2014 to Revit 2015

I performed the standard steps to migrate a Revit add-in from Revit 2014 to Revit 2015:

- Changed Revit API assembly references
- Changed the .NET framework version from 4.0 to 4.5
- Updated the copyright message and version information assembly properties defined in AssemblyInfo.cs
- Compiled, generating [zero errors and six warnings](zip/export_cnc_fab_migr_2015_a.txt)
- Published flat migration as [release 2015.0.0.0](https://github.com/jeremytammik/ExportCncFab/releases/tag/2015.0.0.0)

The good news is that the flat migrated add-in compiles successfully.

The warnings are not really bad news, either; they just indicate usage of some deprecated API functionality, which is completely normal.

I fixed them right away in order to simplify to continued migration to the next major release.

The six warnings are actually just three, with some repetitions:

- `Autodesk.Revit.UI.Selection.Selection.Elements` is obsolete: This property is deprecated in Revit 2015. Use GetElementIds() and SetElementIds instead.
- `Autodesk.Revit.DB.Element.get_Parameter(string)` is obsolete: This property is obsolete in Revit 2015, as more than one parameter can have the same name on a given element. Use Element.Parameters to obtain a complete list of parameters on this Element, or Element.GetParameters(String) to get a list of all parameters by name, or Element.LookupParameter(String) to return the first available parameter with the given name.
- `Autodesk.Revit.DB.Definitions.Create(string, Autodesk.Revit.DB.ParameterType, bool)` is obsolete: This method is deprecated in Revit 2015. Use Create(Autodesk.Revit.DB.ExternalDefinitonCreationOptions) instead.

Let's take a closer look at these three warnings and how to fix them:

- [Replacing Selection.Elements by GetElementIds](#2015.1)
- [Replacing Element.get\_Parameter by Element.GetParameters](#2015.2)
- [Using ExternalDefinitonCreationOptions](#2015.3)

#### Replacing Selection.Elements by GetElementIds

Instead of accessing a collection of the currently selected elements directly, Revit now provides a list of their element ids.

The code updated to compile with no warnings in Revit 2015 looks like this, with the obsolete code commented out:

```csharp
  // Iterate over all pre-selected parts

  List<ElementId> ids = null;

  Selection sel = uidoc.Selection;

  ICollection<ElementId> selIds = sel.GetElementIds(); // 2015

  //if( 0 < sel.Elements.Size ) // 2014

  if( 0 < selIds.Count ) // 2015
  {
    //foreach( Element e in sel.Elements ) // 2014

    foreach( ElementId id in selIds ) // 2015
    {
      Element e = doc.GetElement( id );

      if( !( e is Part ) )
      {
        ErrorMsg( "Please pre-select only gyp wallboard"
          + " parts before running this command." );
        return Result.Failed;
      }

      Part part = e as Part;

      // . . .
```

#### Replacing Element.get\_Parameter by Element.GetParameters

Multiple parameters of the same name can be attached to a single Revit element.

In order to force developers to handle this possibility in a fool-proof way, the get\_Parameter method taking a parameter name is deprecated in Revit 2015 and can be replaced by the GetParameters method that returns a collection of all the parameters of the given name instead.

Here is an example of some Revit 2014 code that causes the corresponding warning:

```csharp
  /// <summary>
  /// Return the parameter definition from
  /// the given element and parameter name.
  /// </summary>
  static Definition GetDefinition(
    Element e,
    string parameter\_name )
  {
    Parameter p = e.get\_Parameter( parameter\_name );

    Definition d = ( null == p )
      ? null
      : p.Definition;

    return d;
  }
```

Here is possibly way of fixing this to compile cleanly in Revit 2015:

```csharp
  /// <summary>
  /// Return the parameter definition from
  /// the given element and parameter name.
  /// </summary>
  static Definition GetDefinition(
    Element e,
    string parameter\_name )
  {
    IList<Parameter> ps = e.GetParameters( parameter\_name );

    int n = ps.Count;

    Debug.Assert( 1 >= n,
      "expected maximum one shared parameters "
      + "named " + parameter\_name );

    Definition d = ( 0 == n )
      ? null
      : ps[0].Definition;

    return d;
  }
```

#### Using ExternalDefinitonCreationOptions

This is the code used to define the ExportCncFab shared parameters in Revit 2014 using the DefinitionGroup.Definitions.Create method:

```csharp
  // Create the category set for binding

  CategorySet catSet = app.Create.NewCategorySet();

  Category cat = doc.Settings.Categories.get\_Item(
    BuiltInCategory.OST\_Parts );

  catSet.Insert( cat );

  Binding binding = app.Create.NewInstanceBinding(
    catSet );

  // Retrieve or create shared parameter group

  DefinitionGroup group
    = f.Groups.get\_Item( \_definition\_group\_name )
    ?? f.Groups.Create( \_definition\_group\_name );

  // Retrieve or create the three parameters;
  // we could check if they are already bound,
  // but it looks like Insert will just ignore
  // them in that case.

  Definition definition
    = group.Definitions.get\_Item( \_is\_exported )
    ?? group.Definitions.Create( \_is\_exported,
      ParameterType.YesNo, true );

  doc.ParameterBindings.Insert( definition, binding,
    BuiltInParameterGroup.PG\_GENERAL );

  definition
    = group.Definitions.get\_Item( \_exported\_first )
    ?? group.Definitions.Create( \_exported\_first,
      ParameterType.Text, true );

  doc.ParameterBindings.Insert( definition, binding,
    BuiltInParameterGroup.PG\_GENERAL );

  definition
    = group.Definitions.get\_Item( \_exported\_last )
    ?? group.Definitions.Create( \_exported\_last,
      ParameterType.Text, true );

  doc.ParameterBindings.Insert( definition, binding,
    BuiltInParameterGroup.PG\_GENERAL );
```

In Revit 2015, this causes deprecated API usage warnings prompting to use the ExternalDefinitonCreationOptions class instead.

To avoid fixing this three times over and using extremely long wrapped lines to do so, I defined the local static CreateNewDefinition helper method to handle this for 2015 and replace the calls to group.Definitions.Create above:

```csharp
  static Definition CreateNewDefinition(
    DefinitionGroup group,
    string parameter\_name,
    ParameterType parameter\_type )
  {
    //return group.Definitions.Create(
    //  parameter\_name, parameter\_type, true ); // 2014

    return group.Definitions.Create(
      new ExternalDefinitonCreationOptions(
        parameter\_name, parameter\_type ) ); // 2015
  }
```

It now compiles with zero errors and zero warnings.

This state of things is published as [release 2015.0.0.2](https://github.com/jeremytammik/ExportCncFab/releases/tag/2015.0.0.2).

#### Migrating ExportCncFab from Revit 2015 to Revit 2016

The flat migration from Revit 2015 to Revit 2016 generates one trivial but nonetheless irritating [error CS0246](zip/export_cnc_fab_migr_2016_a.txt) affecting the CreateNewDefinition method that we just introduced above:

- The type or namespace name 'ExternalDefinitonCreationOptions' could not be found (are you missing a using directive or an assembly reference?)

This is simply due to a typo in the original Revit 2015 API class name that has been rectified in Revit 2016.

The fix is equally trivial and obvious: add the missing letter 'i'.

The new implementation including commented obsolete code now looks like this:

```csharp
  static Definition CreateNewDefinition(
    DefinitionGroup group,
    string parameter\_name,
    ParameterType parameter\_type )
  {
    //return group.Definitions.Create(
    //  parameter\_name, parameter\_type, true ); // 2014

    //return group.Definitions.Create(
    //  new ExternalDefinitonCreationOptions(
    //    parameter\_name, parameter\_type ) ); // 2015

    return group.Definitions.Create(
      new ExternalDefinitionCreationOptions(
        parameter\_name, parameter\_type ) ); // 2016
  }
```

Once that was fixed, no more errors or warnings were generated.

This initial Revit 2016 version is published as [release 2016.0.0.0](https://github.com/jeremytammik/ExportCncFab/releases/tag/2016.0.0.0).

I hope you find this useful and look forward to hearing how you are making use of it!