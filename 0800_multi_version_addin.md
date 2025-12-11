---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.618765'
original_url: https://thebuildingcoder.typepad.com/blog/0800_multi_version_addin.html
post_number: 0800
reading_time_minutes: 7
series: general
slug: multi_version_addin
source_file: 0800_multi_version_addin.htm
tags:
- csharp
- elements
- python
- references
- revit-api
- views
- windows
title: Multi-Version Add-in
word_count: 1316
---

### Multi-Version Add-in

This is blog post number 800, in case you are interested.
I'll celebrate that fact by going out for a run in the woods after posting.

As we all saw when migrating add-ins from Revit 2012 to Revit 2013, the new version requires an add-in to be compiled using the .NET framework version 4.0.
In spite of this, a Revit 2012 add-in can still be loaded into Revit 2013.
I created a sample to prove this.

Not only can a Revit 2012 add-in be loaded into Revit 2013, it can even use .NET reflection to access new Revit 2013 API functionality, provided you know what you are doing.

Here is the question that motivated me to look into this topic again:

**Question:** While migrating to the Revit 2013 API, I ran into problems similar to the ones encountered while
[migrating The Building Coder samples](http://thebuildingcoder.typepad.com/blog/2012/04/migrate-building-coder-samples-to-revit-2013.html).

I would like to develop an add-in which is backwards compatible with the previous version of Revit, supporting for example both Revit 2012 and Revit 2013 in one .NET assembly DLL.

Is that possible at all?

**Answer:** Yes, this is absolutely possible.

This is the complete short answer: in general, the Revit API assemblies are upwards compatible.
If you create an add-in using the Revit 2012 API assemblies and make no use of Revit 2013 API features, and all the Revit 2012 API features that you do make use of remained unchanged in Revit 2013, then you can load and execute your 2012 add-in in Revit 2013.

I created a sample to prove this, as part of my
[eStorage for Revit](http://labs.autodesk.com/utilities/revit_estorage) EstoreFile
sample application, available from the Autodesk Labs
[ADN Plugin of the Month Catalog](http://labs.autodesk.com/utilities/ADN_plugins/catalog).

EstoreFile stores and manages the contents of external files in Revit extensible storage on selected elements in the BIM model.

It was originally created and described as part of the samples for the Autodesk University 2011 extensible storage
[class](http://au.autodesk.com/?nd=event_class&session_id=9263&jid=1725932) and
[lab](http://au.autodesk.com/?nd=event_class&session_id=9726&jid=1725932),
as explained in the overview of the AU 2011
[Revit and AEC API sessions](http://thebuildingcoder.typepad.com/blog/2011/09/revit-and-aec-api-classes-at-autodesk-university.html),
and updated for the
[AEC DevCamp](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-one.html#2) with
an additional external command added to support the new Revit 2013
[DataStorage element](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html#5).

Two important requirements were added when packaging the EstoreFile command as a separate plug-in of the month for the labs:

- Support for both Revit 2012 and Revit 2013.- Support for a help file.

One of the new
[Revit 2013 add-in integration features](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2) is
support for contextual help: the user can simply hit F1 on an add-in command button to bring up an Internet URL or a local help file entry.

The contextual help functionality is provided by the ContextualHelp class in the Autodesk.Revit.UI namespace.
I can therefore use the presence of this class to determine whether my add-in is running in Revit 2013 (or later), or an earlier release.
At the same time, I can store the System.Type of this class to use reflection to access its functionality:
```csharp
  /// <summary>
  /// Retrieve the Revit 2013 ContextualHelp class
  /// type if it exists.
  /// </summary>
  static Type \_contextual\_help\_type = null;

  /// <summary>
  /// Call this method once only to set the
  /// ContextualHelp class type if it exists.
  /// This can be used for a runtime detection
  /// whether we are running in Revit 2012 or 2013.
  /// </summary>
  static void SetContextualHelpType()
  {
    // Create an instance of the RevitAPIUI.DLL
    // assembly to query its type information:

    Assembly rvtApiUiAssembly
      = Assembly.GetAssembly( typeof( UIDocument ) );

    \_contextual\_help\_type = rvtApiUiAssembly.GetType(
      "Autodesk.Revit.UI.ContextualHelp" );
  }
```

Now, seeing as I compile this application using the Revit 2012 API, I have no direct access to the new Revit 2013 functionality.

If I wish to make use of it anyway, I can do so using .NET reflection to determine the methods and properties on the system type and making calls to those.

In this case, given a PushButtonData instance 'd', I want to instantiate a new ContextualHelp instance 'ch' and set it on 'd'.
The code to achieve this in Revit 2013 looks like this:
```csharp
  ContextualHelp ch = new ContextualHelp(
    ContextualHelpType.ChmFile, helppath );

  d.SetContextualHelp( ch );
```

The same thing can be achieved using reflection like this:
```csharp
  /// <summary>
  /// Assign the add-in help file to the
  /// given ribbon push button contextual help.
  /// This method checks for Revit 2013 at runtime
  /// and uses .NET Reflection to access the
  /// ContextualHelp class and SetContextualHelp
  /// method which are not available in the
  /// Revit 2012 API.
  /// </summary>
  void AddContextualHelp( PushButtonData d, int i )
  {
    // Invoke constructor:

    ConstructorInfo[] ctors
      = \_contextual\_help\_type.GetConstructors();

    const int contextualHelpType\_ChmFile = 3;

    object instance = ctors[0].Invoke(
      new object[] {
      contextualHelpType\_ChmFile,
      HelpPath } ); // + "#" + i.ToString()

    // Set the help topic URL

    PropertyInfo property = \_contextual\_help\_type
      .GetProperty( "HelpTopicUrl" );

    property.SetValue( instance, i.ToString(), null );

    // Invoke SetContextualHelp method:

    Type pbdType = d.GetType();

    MethodInfo method = pbdType.GetMethod(
      "SetContextualHelp" );

    method.Invoke( d, new object[] { instance } );
  }
```

The external application OnStartup method makes use of this by performing the following steps:

- Determine the contextual help System.Type and infer from its presence whether we are running in Revit 2012 or 2013.- If we are running in 2013, we can use the contextual help functionality to display the help file, so the separate help command implemented for Revit 2012 is removed.
    Therefore, I also remove the reference to that command in the help text displayed by the expanded tooltip text.- Determine the help file path.- Create the ribbon panel and populate it with buttons for all the commands.- If we are in 2013, skip adding the help command and add contextual help to each of the other commands instead.

Here is the entire OnStartup method source code:
```python
public Result OnStartup( UIControlledApplication a )
{
  int i;

  SetContextualHelpType();

  if( HasContextualHelp )
  {
    // Remove the description of the help command
    // from the long tooltip add-in overview

    i = \_long\_description\_tooltip\_overview
      .IndexOf( "\r\n\r\nHelp: " );

    Debug.Assert( 0 < i, "expected to find "
      + "description of help command in long "
      + "description tootip overview" );

    \_long\_description\_tooltip\_overview
      = \_long\_description\_tooltip\_overview
        .Substring( 0, i );
  }

  ControlledApplication c = a.ControlledApplication;

  Debug.Print( string.Format(
    "EstoreFile: Running in {0} {1}.{2}, "
    + "so contecxtual help is {3}available.",
    c.VersionName, c.VersionNumber, c.VersionBuild,
    (HasContextualHelp ? "" : "not ") ) );

  Assembly exe = Assembly.GetExecutingAssembly();
  string dllpath = exe.Location;
  string dir = Path.GetDirectoryName( dllpath );
  \_help\_path = Path.Combine( dir, \_help\_filename );

  if( !File.Exists( \_help\_path ) )
  {
    string s = "Please ensure that the EstoreFile "
      + "help file {0} is located in the same "
      + " folder as the add-in assembly DLL:\r\n"
      + "'{1}'.";

    System.Windows.MessageBox.Show(
      string.Format( s, \_help\_filename, dir ),
      \_name, System.Windows.MessageBoxButton.OK,
      System.Windows.MessageBoxImage.Error );

    return Result.Failed;
  }

  string className = GetType().FullName.Replace(
    "App", "Cmd" );

  RibbonPanel rp = a.CreateRibbonPanel( \_text );

  i = 0;

  foreach( string cmd in \_commands )
  {
    if( HasContextualHelp && cmd.Equals( "Help" ) )
    {
      break;
    }

    PushButtonData d = new PushButtonData(
      cmd, cmd, dllpath, className + cmd );

    d.ToolTip = \_tips[i];

    d.Image = NewBitmapImage( exe,
      cmd + "16.png" );

    d.LargeImage = NewBitmapImage( exe,
      cmd + "32.png" );

    d.LongDescription = \_long\_tips[i]
      + \_long\_description\_tooltip\_overview;

    if( HasContextualHelp )
    {
      AddContextualHelp( d, ++i );
    }

    rp.AddItem( d );
  }
  return Result.Succeeded;
}
```

For the complete source code, Visual Studio solution, and add-in manifest, please refer to the Autodesk Labs
[eStorage for Revit](http://labs.autodesk.com/utilities/revit_estorage) download.

I trust this answers your question and provides a neat sample to boot.