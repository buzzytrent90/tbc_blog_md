---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.424117'
original_url: https://thebuildingcoder.typepad.com/blog/1159_va3c_resolve_assembly.html
post_number: '1159'
reading_time_minutes: 6
series: general
slug: va3c_resolve_assembly
source_file: 1159_va3c_resolve_assembly.htm
tags:
- csharp
- elements
- filtering
- geometry
- revit-api
- rooms
- views
title: Assembly Resolver
word_count: 1149
---

### Assembly Resolver

One of the
RvtVa3c
[implementation aspects](...#3)
that I mentioned was the fact that we ran into some problems using the standard .NET Microsoft System.Runtime.Serialization.Json.DataContractJsonSerializer class and chose to replace it with
the more reliable [Json.NET](http://james.newtonking.com/json) component instead.

Serialisation is required by our early decision to define and generate the three.js JSON file format by representing all the required objects, their properties and relationships by a set of C# classes.

We instantiate and populate these classes in our custom exporter context implementation and serialise them out to JSON to generate the output file used to represent the BIM for the va3c viewer.

A little bit too late, Ben pointed out that we could have saved ourselves the whole effort of implementing and populating a C# class hierarchy by using
[dynamic JSON and the System.Dynamic.ExpandoObject](https://github.com/va3c/GHva3c/blob/master/GHva3c/GHva3c/va3c_geometry.cs) instead,
as he does in his
[GHva3c Grasshopper va3c exporter](https://github.com/va3c/GHva3c).

Anyway, we continued down the path of C# class definition and serialisation to generate the JSON output file starting from the root three.js scene object.

The original code using a DataContractJsonSerializer to serialise it and thus generate the JSON output looked like this:

```csharp
  using( FileStream stream
    = File.OpenWrite( filename ) )
  {
    DataContractJsonSerializer serialiser
      = new DataContractJsonSerializer(
        typeof( Va3cScene ) );

    serialiser.WriteObject( stream, \_scene );
  }
```

This became too buggy at a certain point.
For instance, when writing to the same file repeatedly, the end of the updated file still contained data from the previous version, obviously corrupting the entire structure.

It can be easily replaced by the following code using Json.NET:

```csharp
  JsonSerializerSettings settings
    = new JsonSerializerSettings();

  settings.NullValueHandling
    = NullValueHandling.Ignore;

  File.WriteAllText( \_filename,
    JsonConvert.SerializeObject( \_scene,
      Formatting.Indented, settings ) );
```

On the first run, though, this caused our external command to fail with an error reporting that the Newtonsoft.Json.dll assembly could not be loaded.

This is due to .NET restrictions requiring proper .NET application behaviour.

[As mentioned](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html),
one way to resolve this issue is to install the entire application in a sub-folder of the Revit.exe directory, which is not always feasible.

If it lives elsewhere, the restriction can be somewhat circumvented using a .NET assembly resolver.

I mentioned this beast in the past, e.g. discussing the RvtUnit project for
[Revit Add-in unit testing](http://thebuildingcoder.typepad.com/blog/2013/07/revit-add-in-unit-testing.html) and
[using REX without the REX framework](http://thebuildingcoder.typepad.com/blog/2013/12/security-framing-cross-section-analyser-and-rex.html#6).

Matt sets up such resolution handlers on a regular basis, so he quickly typed the following code from scratch by heart to implement and register it in the main RvtVa3c exporter method ExportView3D:

```csharp
  /// <summary>
  /// Custom assembly resolver to find our support
  /// DLL without being forced to place our entire
  /// application in a subfolder of the Revit.exe
  /// directory.
  /// </summary>
  System.Reflection.Assembly
    CurrentDomain\_AssemblyResolve(
      object sender,
      ResolveEventArgs args )
  {
    if( args.Name.Contains( "Newtonsoft" ) )
    {
      string filename = Path.GetDirectoryName(
        System.Reflection.Assembly
          .GetExecutingAssembly().Location );

      filename = Path.Combine( filename,
        "Newtonsoft.Json.dll" );

      if( File.Exists( filename ) )
      {
        return System.Reflection.Assembly
          .LoadFrom( filename );
      }
    }
    return null;
  }

  /// <summary>
  /// Export a given 3D view to JSON using
  /// our custom exporter context.
  /// </summary>
  void ExportView3D( View3D view3d, string filename )
  {
    AppDomain.CurrentDomain.AssemblyResolve
      += CurrentDomain\_AssemblyResolve;

    Document doc = view3d.Document;

    Va3cExportContext context
      = new Va3cExportContext( doc, filename );

    CustomExporter exporter = new CustomExporter(
      doc, context );

    // Note: Excluding faces just suppresses the
    // OnFaceBegin calls, not the actual processing
    // of face tessellation. Meshes of the faces
    // will still be received by the context.

    exporter.IncludeFaces = false;

    exporter.ShouldStopOnError = false;

    exporter.Export( view3d );
  }
```

For completeness sake, here is the rest of the external command mainline implementation, prompting for interactive use of the output filename and folder, stored for reuse in subsequent runs in the same session:

```csharp
 #region SelectFile
  /// <summary>
  /// Store the last user selected output folder
  /// in the current editing session.
  /// </summary>
  static string \_output\_folder\_path = null;

  /// <summary>
  /// Return true is user selects and confirms
  /// output file name and folder.
  /// </summary>
  static bool SelectFile(
    ref string folder\_path,
    ref string filename )
  {
    SaveFileDialog dlg = new SaveFileDialog();

    dlg.Title = "JSelect SON Output File";
    dlg.Filter = "JSON files|\*.js";

    if( null != folder\_path
      && 0 < folder\_path.Length )
    {
      dlg.InitialDirectory = folder\_path;
    }

    dlg.FileName = filename;

    bool rc = DialogResult.OK == dlg.ShowDialog();

    if( rc )
    {
      filename = Path.GetFileName( dlg.FileName );

      folder\_path = Path.GetDirectoryName(
        filename );
    }
    return rc;
  }
  #endregion // SelectFile

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    if( doc.ActiveView is View3D )
    {
      string filename = doc.PathName;
      if( 0 == filename.Length )
      {
        filename = doc.Title;
      }
      if( null == \_output\_folder\_path )
      {
        \_output\_folder\_path = Path.GetDirectoryName(
          filename );
      }
      filename = Path.GetFileName( filename ) + ".js";

      if( SelectFile( ref \_output\_folder\_path,
        ref filename ) )
      {
        filename = Path.Combine( \_output\_folder\_path,
          filename );

        ExportView3D( doc.ActiveView as View3D,
          filename );

        return Result.Succeeded;
      }
      return Result.Cancelled;
    }
    else
    {
      TaskDialog.Show( "va3c",
        "You must be in 3D view to export." );
    }
    return Result.Failed;
  }
```

For the custom exporter code populating the objects representing the three.js scene hierarchy and the entire add-in implementation, please refer to the
[RvtVa3c GitHub repository](https://github.com/va3c/RvtVa3c) and the
[Va3cExportContext implementation](https://github.com/va3c/RvtVa3c .... ).

#### External Command Lifecycle

Matt added an additional consideration:

**Question:** I realized one thing that I didn't do here that I sometimes address – but it would make the blog post code a bit more complicated.

Do you know what the lifecycle is of a Revit command? Does it get disposed afterwards, or is it re-used, or left alive? I have a vague feeling that it doesn't “go away” right away.

As such, because we've registered this event on the current appDomain, you might still be getting callbacks into this event after you leave this app and run the next command (which can be confusing).

To address this, you would define the callback event handler first, add it, and then remove the handler at the end when the command ended.

So – it's up to you if that's a complexity that should be addressed or not? Because this example is only loading the Newtonsoft JSON specifically, it's pretty harmless.

**Answer:** A new external command implementation class is instantiated each time you launch the command, as I painfully discovered searching for a
[solution to the RoomEditorApp idling issues](http://thebuildingcoder.typepad.com/blog/2013/11/roomeditorapp-idling-and-benchmarking-timer.html#2):
"each new external command invocation generates a new different class instance."

For this reason, I was unable to unsubscribe from an external event that was subscribed to from an external command, and changed the subscriber to be the external application instead of the external command.