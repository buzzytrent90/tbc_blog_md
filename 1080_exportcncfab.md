---
post_number: "1080"
title: "Driving CNC Fabrication and Shared Parameters"
slug: "exportcncfab"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'parameters', 'python', 'references', 'revit-api', 'selection', 'transactions', 'views', 'walls', 'windows']
source_file: "1080_exportcncfab.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1080_exportcncfab.html"
---

### Driving CNC Fabrication and Shared Parameters

The topic of CNC fabrication of Revit BIM elements is continuing to grow in popularity.

As you already know, I published the ExportWallboard add-in to automatically isolate and
[export wall parts individually to DXF](http://thebuildingcoder.typepad.com/blog/2013/03/export-wall-parts-individually-to-dxf.html) for
CNC fabrication,
then enhanced, renamed and published it on GitHub as
[ExportCncFab](http://thebuildingcoder.typepad.com/blog/2013/10/exportcncfab-on-github-and-revitlookup-update.html) for
William Spier's Autodesk University class on
[Design to Fabrication](http://thebuildingcoder.typepad.com/blog/2013/12/au-day-2-worksharing-and-revit-2014-api-roundtables.html#3).

Those discussions were explicitly related to CNC fabrication up front.
Numerous other topics here are also useful in that context.

I was unable to highlight and discuss the newly added CNC add-in enhancements in detail before AU, so let's make up for that now, as well as look at some funny examples of [genius problem simplifications](#7).

#### ExportCncFab Functionality

ExportCncFab now sports the following features:

- [Exporting wall parts individually to DXF:](http://thebuildingcoder.typepad.com/blog/2013/03/export-wall-parts-individually-to-dxf.html)

- [Exporting individual compound wall parts](http://thebuildingcoder.typepad.com/blog/2013/03/export-wall-parts-individually-to-dxf.html##2)
- [Handling and dismissing a warning message](http://thebuildingcoder.typepad.com/blog/2013/03/export-wall-parts-individually-to-dxf.html##3)
- [Adding support for both pre- and post- part selection](http://thebuildingcoder.typepad.com/blog/2013/03/export-wall-parts-individually-to-dxf.html##4)
- [Handling temporary transactions and regeneration](http://thebuildingcoder.typepad.com/blog/2013/03/export-wall-parts-individually-to-dxf.html##5)

- Support for [export to SAT](#3) as well as DXF
- CNC fabrication export history tracking and creation of [shared parameters](#4)
- A beautiful little [external application](#5) to define a nice user interface

I already presented the details of the DXF export, so now let's take a look at the other three items.

By the way, for the sake of completeness, let me also mention this previous example of
[exporting walls and floors to SAT](http://thebuildingcoder.typepad.com/blog/2012/01/export-walls-and-floors-to-sat.html),
before the advent of parts, and this
[implementation of saving a solid to a SAT file](http://thebuildingcoder.typepad.com/blog/2013/09/saving-a-solid-to-a-sat-file-implementation.html).

#### Adding Export to SAT Functionality

Since I want to present the wall parts export to DXF or SAT functionality as two separate options in the user interface, they each need to be implemented as separate external commands.

On the other hand, I obviously avoid duplicating the code, which is mostly identical.

One way to achieve that is implement the common DXF and SAT export functionality in a separate method CmdDxf.Execute2 that can be called from both commands and takes an argument to toggle between the DXF and SAT export.

With that in place, I can implement the new SAT export external command like this:

```python
  [Transaction( TransactionMode.Manual )]
  public class CmdSat : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      return CmdDxf.Execute2( commandData, true );
    }
  }
```

The DXF export command is almost identical, except for providing a 'false' argument to Execute2.

Both commands are forced to use manual transaction mode instead of read-only for two reasons: (i) they perform temporary modifications in the database to isolate the parts to export one by one, and (ii) they save the export history in shared parameters, requiring database update for storage.

Besides the pure export and history recording functionality, the Execute2 implementation is complicated further still, since it also handles a warning dialogue displayed by Revit to ask whether to "really print or export temp view modes".
It does so by temporarily subscribing to the DialogBoxShowing event and providing the following OnDialogBoxShowing event handler for it:

```csharp
  static void OnDialogBoxShowing(
    object sender,
    DialogBoxShowingEventArgs e )
  {
    TaskDialogShowingEventArgs e2
      = e as TaskDialogShowingEventArgs;

    if( null != e2 && e2.DialogId.Equals(
      "TaskDialog\_Really\_Print\_Or\_Export\_Temp\_View\_Modes" ) )
    {
      int cmdLink
        = (int) TaskDialogResult.CommandLink2;

      e.OverrideResult( cmdLink );
    }
  }
```

Besides suppressing and handling that message, the Execute2 method also has to jump through a hoop or two to store the export history in the shared parameters, and check that they exist before doing so.

All together, it performs the following steps:

- Check we are in a valid document.
- Check we are in a valid 3D view.
- Check that parts visibility is turned on.
- Determine the elements to export, either via pre- or post-selection.
- Check that the export history shared parameters have been defined.
- Prompt to select the export target directory.
- Encapsulate the following actions in a transaction group that is ultimately rolled back, leaving the document unmodified by the part export.
- Within the transaction group, iterate over the selected parts one by one.
- For each part:

- Disable the view temporary isolate mode if it was previously active.
- Temporarily isolate the current element.
- Export the current view displaying only one element to DXF or SAT, respectively.
- Cache the export history for the current element.

- Roll back the transaction group.
- Store the export history in the shared parameters in a separate transaction that is obviously not rolled back and therefore needs to reside outside the previous transaction group.

This slightly convoluted process evolved step by step as the needs expanded.
Surprisingly, it works completely reliably.

Here is the code implementing this in all its glory, including some interesting comments referring to the process itself and the migration from Revit 2013 to 2014:

```csharp
  public static Result Execute2(
    ExternalCommandData commandData,
    bool exportToSatFormat )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    if( null == doc )
    {
      ErrorMsg( "Please run this command in a valid"
        + " Revit project document." );
      return Result.Failed;
    }

    View view = doc.ActiveView;

    if( null == view || !( view is View3D ) )
    {
      ErrorMsg( "Please run this command in a valid"
        + " 3D view." );
      return Result.Failed;
    }

    if( PartsVisibility.ShowPartsOnly
      != view.PartsVisibility )
    {
      ErrorMsg( "Please run this command in a view"
        + " displaying parts and not source elements." );
      return Result.Failed;
    }

    // Define the list of views to export,
    // including only the current 3D view

    List<ElementId> viewIds = new List<ElementId>( 1 );

    viewIds.Add( view.Id );

    // Iterate over all pre-selected parts

    List<ElementId> ids = null;

    Selection sel = uidoc.Selection;

    if( 0 < sel.Elements.Size )
    {
      foreach( Element e in sel.Elements )
      {
        if( !( e is Part ) )
        {
          ErrorMsg( "Please pre-select only gyp wallboard"
            + " parts before running this command." );
          return Result.Failed;
        }

        Part part = e as Part;

        ICollection<LinkElementId> lids
          = part.GetSourceElementIds();

        if( 1 != lids.Count )
        {
          ErrorMsg( "Gyp wallboard part has multiple"
            + " source elements." );
          return Result.Failed;
        }

        LinkElementId lid = lids.First<LinkElementId>();
        ElementId hostId = lid.HostElementId;
        ElementId linkedId = lid.LinkedElementId;
        ElementId parentId = hostId;
        ElementId partId = e.Id;

        // Determine parent category

        Element parent = doc.GetElement( parentId );
        Category cat = parent.Category;

        ICollection<ElementId> cids
          = part.GetSourceElementOriginalCategoryIds();

        if( 1 != cids.Count )
        {
          ErrorMsg( "Gyp wallboard part has multiple"
            + " source element categories." );
          return Result.Failed;
        }

        ElementId cid = cids.First<ElementId>();

        //cat = doc.GetElement( id ) as Category;

        // Expected parent category is OST\_Walls

        BuiltInCategory bic
          = (BuiltInCategory) cid.IntegerValue;

        if( BuiltInCategory.OST\_Walls != bic )
        {
          ErrorMsg( "Please pre-select only "
            + " gyp wallboard parts." );

          return Result.Failed;
        }

        if( null == ids )
        {
          ids = new List<ElementId>( 1 );
        }

        ids.Add( partId );
      }

      if( null == ids )
      {
        ErrorMsg( "Please pre-select only gyp wallboard"
          + " parts before running this command." );
        return Result.Failed;
      }
    }

    // If no parts were pre-selected,
    // prompt for post-selection

    if( null == ids )
    {
      IList<Reference> refs = null;

      try
      {
        refs = sel.PickObjects( ObjectType.Element,
          new WallPartSelectionFilter(),
          "Please select wall parts." );
      }
      catch( Autodesk.Revit.Exceptions
        .OperationCanceledException )
      {
        return Result.Cancelled;
      }
      ids = new List<ElementId>(
        refs.Select<Reference, ElementId>(
          r => r.ElementId ) );
    }

    if( 0 == ids.Count )
    {
      ErrorMsg( "No valid parts selected." );

      return Result.Failed;
    }

    // Check for shared parameters
    // to record export history

    ExportParameters exportParameters
      = new ExportParameters(
        doc.GetElement( ids[0] ) );

    if( !exportParameters.IsValid )
    {
      ErrorMsg( "Please initialise the CNC fabrication "
        + "export history shared parameters before "
        + "launching this command." );

      return Result.Failed;
    }

    if( !Util.BrowseDirectory( ref \_folder, true ) )
    {
      return Result.Cancelled;
    }

    try
    {
      // Register event handler for
      // "TaskDialog\_Really\_Print\_Or\_Export\_Temp\_View\_Modes"
      // dialogue

      uiapp.DialogBoxShowing
        += new EventHandler<DialogBoxShowingEventArgs>(
          OnDialogBoxShowing );

      object opt = exportToSatFormat
        ? (object) new SATExportOptions()
        : (object) new DXFExportOptions();

      //opt.FileVersion = ACADVersion.R2000;

      string filename;

      using( TransactionGroup txg = new TransactionGroup( doc ) )
      {
        txg.Start( "Export Wall Parts" );

        foreach( ElementId id in ids )
        {
          Element e = doc.GetElement( id );

          Debug.Assert( e is Part,
            "expected parts only" );

          Part part = e as Part;

          ICollection<LinkElementId> lids
            = part.GetSourceElementIds();

          Debug.Assert( 1 == lids.Count,
            "unexpected multiple part source elements." );

          LinkElementId lid = lids.First<LinkElementId>();
          ElementId hostId = lid.HostElementId;
          ElementId linkedId = lid.LinkedElementId;
          ElementId parentId = hostId;
          ElementId partId = e.Id;

          filename = string.Format( "{0}\_{1}",
            parentId, partId );

          Element host = doc.GetElement( hostId );

          Debug.Assert( null != host, "expected to be able to access host element" );
          //Debug.Assert( ( host is Wall ), "expected host element to be a wall" );
          Debug.Assert( ( host is Wall ) || ( host is Part ), "expected host element to be a wall or part" );
          Debug.Assert( null != host.Category, "expected host element to have a valid category" );
          //Debug.Assert( host.Category.Id.IntegerValue.Equals( (int) BuiltInCategory.OST\_Walls ), "expected host element to have wall category" );
          Debug.Assert( host.Category.Id.IntegerValue.Equals( (int) BuiltInCategory.OST\_Walls ) || host.Category.Id.IntegerValue.Equals( (int) BuiltInCategory.OST\_Parts ), "expected host element to have wall or part category" );
          Debug.Assert( ElementId.InvalidElementId != host.LevelId, "expected host element to have a valid level id" );

          if( ElementId.InvalidElementId != host.LevelId )
          {
            Element level = doc.GetElement( host.LevelId );

            filename = level.Name.Replace( ' ', '\_' )
              + "\_" + filename;
          }

          if( view.IsTemporaryHideIsolateActive() )
          {
            using( Transaction tx = new Transaction( doc ) )
            {
              tx.Start( "Disable Temporary Isolate" );

              view.DisableTemporaryViewMode(
                TemporaryViewMode.TemporaryHideIsolate );

              tx.Commit();
            }

            Debug.Assert( !view.IsTemporaryHideIsolateActive(),
              "expected to turn off temporary hide/isolate" );
          }

          using( Transaction tx = new Transaction( doc ) )
          {
            tx.Start( "Export Wall Part "
              + partId.ToString() );

            // This call requires a transaction.

            view.IsolateElementTemporary( partId );

            //List<ElementId> unhideIds = new List<ElementId>( 1 );
            //unhideIds.Add( partId );
            //view.UnhideElements( unhideIds );

            //doc.Regenerate(); // this is insufficient

            tx.Commit();
          }

          if( exportToSatFormat )
          {
            //ViewSet viewSet = new ViewSet();
            //
            //foreach( ElementId vid in viewIds )
            //{
            //  viewSet.Insert( doc.GetElement( vid )
            //    as View );
            //}
            //
            //doc.Export( \_folder, filename, viewSet,
            //  (SATExportOptions) opt ); // 2013

            doc.Export( \_folder, filename, viewIds,
              (SATExportOptions) opt ); // 2014
          }
          else
          {
            doc.Export( \_folder, filename, viewIds,
              (DXFExportOptions) opt );
          }

          // Update CNC fabrication
          // export shared parameters -- oops,
          // cannot do this immediately, since
          // this transaction group will be
          // rolled back ... just save the
          // element id and do it later
          // searately.

          //exportParameters.UpdateExportHistory( e );
          exportParameters.Add( e.Id );
        }

        // We do not commit the transaction group,
        // because no modifications should be saved.
        // The transaction group is only created and
        // started to encapsulate the transactions
        // required by the IsolateElementTemporary
        // method. Since the transaction group is not
        // committed, the changes are automatically
        // discarded.

        //txg.Commit();
      }
    }
    finally
    {
      uiapp.DialogBoxShowing
        -= new EventHandler<DialogBoxShowingEventArgs>(
          OnDialogBoxShowing );
    }

    using( Transaction tx = new Transaction( doc ) )
    {
      tx.Start( "Update CNC Fabrication Export "
        + "History Shared Parameters" );

      exportParameters.UpdateExportHistory();

      tx.Commit();
    }
    return Result.Succeeded;
  }
```

#### CNC fabrication Export History Tracking and Creation of Shared Parameters

You probably noted above that the export history and shared parameter management is handled by a separate ExportParameters class.

The following CNC fabrication export history data is tracked and used to populate the corresponding shared parameters on each exported part:

- CncFabIsExported – a Boolean value showing whether an individual part has ever been exported
- CncFabExportedFirst – timestamp of first export
- CncFabExportedLast – timestamp of most recent export

The ExportParameters class implements the following functionality and public interface methods to fulfil this task:

- Define the user visible export history shared parameter names.
- Store the export history shared parameter definitions.
- GetDefinition – Retrieve the parameter Definition from the given element and parameter name.
- Constructor – Initialise the shared parameter definitions from a given sample element.
- IsValid – Check whether all CNC fabrication export parameter definitions were successfully initialised.
- Add – Add a part element id to the list of successfully exported parts.
- UpdateExportHistory – Update the CNC fabrication export history for the given element.
- UpdateExportHistory – Update the CNC fabrication export history for all stored element ids.
- Create – Create the shared parameters to keep track of the CNC fabrication export history:

- Retrieve shared parameter file name
- Retrieve shared parameter file object
- Create the category set for binding
- Retrieve or create shared parameter group
- Retrieve or create the three parameters

Here is the entire class implementation:

```python
/// <summary>
/// Shared parameters to keep track of
/// the CNC fabrication export history.
/// </summary>
class ExportParameters
{
  /// <summary>
  /// Define the user visible export
  /// history shared parameter names.
  /// </summary>
  const string \_is\_exported = "CncFabIsExported";
  const string \_exported\_first = "CncFabExportedFirst";
  const string \_exported\_last = "CncFabExportedLast";

  /// <summary>
  /// Store the export history
  /// shared parameter definitions.
  /// </summary>
  Definition \_definition\_is\_exported = null;
  Definition \_definition\_exported\_first = null;
  Definition \_definition\_exported\_last = null;

  Document \_doc = null;
  List<ElementId> \_ids = null;

  /// <summary>
  /// Return the parameter definition from
  /// the given element and parameter name.
  /// </summary>
  static Definition GetDefinition(
    Element e,
    string parameter\_name )
  {
    Parameter p = e.get\_Parameter(
      parameter\_name );

    Definition d = ( null == p )
      ? null
      : p.Definition;

    return d;
  }

  /// <summary>
  /// Initialise the shared parameter definitions
  /// from a given sample element.
  /// </summary>
  public ExportParameters( Element e )
  {
    \_definition\_is\_exported = GetDefinition(
      e, \_is\_exported );

    \_definition\_exported\_first = GetDefinition(
      e, \_exported\_first );

    \_definition\_exported\_last = GetDefinition(
      e, \_exported\_last );
    if( IsValid )
    {
      \_doc = e.Document;
      \_ids = new List<ElementId>();
    }
  }

  /// <summary>
  /// Check whether all CNC fabrication export
  /// parameter definitions were successfully
  /// initialised.
  /// </summary>
  public bool IsValid
  {
    get
    {
      return null != \_definition\_is\_exported
        && null != \_definition\_exported\_first
        && null != \_definition\_exported\_last;
    }
  }

  /// <summary>
  /// Add a part element id to the list of
  /// successfully exported parts.
  /// </summary>
  public void Add( ElementId id )
  {
    \_ids.Add( id );
  }

  /// <summary>
  /// Update the CNC fabrication export
  /// history for the given element.
  /// </summary>
  void UpdateExportHistory(
    Element e )
  {
    DateTime now = DateTime.Now;

    string s = string.Format(
      "{0:4}-{1:02}-{2:02}T{3:02}.{4:02}.{5:02}.{6:03}",
      now.Year, now.Month, now.Day,
      now.Hour, now.Minute, now.Second, now.Millisecond );

    s = now.ToString( "yyyy-MM-ddTHH:mm:ss.fff" );

    e.get\_Parameter( \_definition\_is\_exported )
      .Set( 1 );

    Parameter p = e.get\_Parameter(
      \_definition\_exported\_first );

    string s2 = p.AsString();

    if( null == s2 || 0 == s2.Length )
    {
      p.Set( s );
    }

    e.get\_Parameter( \_definition\_exported\_last )
      .Set( s );
  }

  /// <summary>
  /// Update the CNC fabrication export
  /// history for all stored element ids.
  /// </summary>
  public void UpdateExportHistory()
  {
    foreach( ElementId id in \_ids )
    {
      UpdateExportHistory(
        \_doc.GetElement( id ) );
    }
  }

  /// <summary>
  /// Create the shared parameters to keep track
  /// of the CNC fabrication export history.
  /// </summary>
  public static void Create( Document doc )
  {
    /// <summary>
    /// Shared parameters filename; used only in case
    /// none is set and we need to create the export
    /// history shared parameters.
    /// </summary>
    const string \_shared\_parameters\_filename
      = "export\_cnc\_fab\_shared\_parameters.txt";

    const string \_definition\_group\_name = "CncFab";

    Application app = doc.Application;

    // Retrieve shared parameter file name

    string sharedParamsFileName
      = app.SharedParametersFilename;

    if( null == sharedParamsFileName
      || 0 == sharedParamsFileName.Length )
    {
      string path = Path.GetTempPath();

      path = Path.Combine( path,
        \_shared\_parameters\_filename );

      StreamWriter stream;
      stream = new StreamWriter( path );
      stream.Close();

      app.SharedParametersFilename = path;

      sharedParamsFileName
        = app.SharedParametersFilename;
    }

    // Retrieve shared parameter file object

    DefinitionFile f
      = app.OpenSharedParameterFile();

    using( Transaction t = new Transaction( doc ) )
    {
      t.Start( "Create CNC Export Tracking "
        + "Shared Parameters" );

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

      t.Commit();
    }
  }
}
```

#### External Application Implementation

There is nothing very special about the external application implementation.

All it does is present the three buttons to trigger the external commands to create the shared parameters and perform the export to the two supported formats:

![Export to CNC fabrication add-in](img/export_cnc_fab_app.png)

In fact, it does do one thing more, namely read the bitmap images for the command button icons and tooltip images from embedded resources.

Here is the complete implementation:

```python
class App : IExternalApplication
{
  public const string Caption
    = "Export to CNC Fabrication";

  static string \_namespace\_prefix
    = typeof( App ).Namespace + ".";

  const string \_name = "DXF";
  const string \_name2 = "SAT";
  const string \_name3 = "Create\r\nShared\r\nParameters";

  const string \_class\_name = "CmdDxf";
  const string \_class\_name2 = "CmdSat";
  const string \_class\_name3 = "CmdCreateSharedParameters";

  const string \_tooltip\_format
    = "Export to CNC Fabrication in {0} format";

  const string \_tooltip\_long\_description\_format
    = "Export Revit parts to CNC Fabrication in {0} format.";

  /// <summary>
  /// Load a new icon bitmap from embedded resources.
  /// For the BitmapImage, make sure you reference
  /// WindowsBase and PresentationCore, and import
  /// the System.Windows.Media.Imaging namespace.
  /// </summary>
  BitmapImage NewBitmapImage(
    Assembly a,
    string imageName )
  {
    // to read from an external file:
    //return new BitmapImage( new Uri(
    //  Path.Combine( \_imageFolder, imageName ) ) );

    Stream s = a.GetManifestResourceStream(
      \_namespace\_prefix + imageName );

    BitmapImage img = new BitmapImage();

    img.BeginInit();
    img.StreamSource = s;
    img.EndInit();

    return img;
  }

  public Result OnStartup(
    UIControlledApplication a )
  {
    Assembly exe = Assembly.GetExecutingAssembly();
    string path = exe.Location;

    // Create ribbon panel

    RibbonPanel p = a.CreateRibbonPanel( Caption );

    // Create DXF button

    PushButtonData d = new PushButtonData(
      \_name, \_name, path,
      \_namespace\_prefix + \_class\_name );

    d.ToolTip = string.Format( \_tooltip\_format, \_name );
    d.Image = NewBitmapImage( exe, "cnc\_icon\_16x16\_size.png" );
    d.LargeImage = NewBitmapImage( exe, "cnc\_icon\_32x32\_size.png" );
    d.LongDescription = string.Format( \_tooltip\_long\_description\_format, \_name );
    d.ToolTipImage = NewBitmapImage( exe, "cnc\_icon\_full\_size.png" );

    p.AddItem( d );

    // Create SAT button

    d = new PushButtonData(
      \_name2, \_name2, path,
      \_namespace\_prefix + \_class\_name2 );

    d.ToolTip = string.Format( \_tooltip\_format, \_name2 );
    d.Image = NewBitmapImage( exe, "cnc\_icon\_16x16\_size.png" );
    d.LargeImage = NewBitmapImage( exe, "cnc\_icon\_32x32\_size.png" );
    d.LongDescription = string.Format( \_tooltip\_long\_description\_format, \_name2 );
    d.ToolTipImage = NewBitmapImage( exe, "cnc\_icon\_full\_size.png" );

    p.AddItem( d );

    // Create shared parameters button

    d = new PushButtonData(
      \_name3, \_name3, path,
      \_namespace\_prefix + \_class\_name3 );

    d.ToolTip
      = "Create shared parameters for tracking export history";

    d.LongDescription
      = "Create and bind shared parameters to the "
      + "Parts category for tracking export history:\r\n\r\n"
      + " \* CncFabIsExported - Boolean\r\n"
      + " \* CncFabExportedFirst - Text timestamp ISO 8601\r\n"
      + " \* CncFabExportedLast - Text timestamp ISO 8601";

    d.ToolTipImage = NewBitmapImage( exe,
      "cnc\_icon\_full\_size.png" );

    p.AddItem( d );

    return Result.Succeeded;
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    return Result.Succeeded;
  }
}
```

#### Download

To enable you explore this for yourself, the current version is available from the
[ExportCncFab GitHub repository](https://github.com/jeremytammik/ExportCncFab),
and the version discussed here is
[release 2014.0.0.12](https://github.com/jeremytammik/ExportCncFab/releases/tag/2014.0.0.12).
Enjoy!

#### Thinking Outside the Box

For some fabulous and hilarious examples of thinking outside the box, take a look at these
[test answers that are 100% wrong and totally genius at the same time](http://distractify.com/fun/fails/test-answers-that-are-totally-wrong-but-still-genius).