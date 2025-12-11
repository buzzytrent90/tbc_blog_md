---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.9
content_type: code_example
optimization_date: '2025-12-11T11:44:14.980899'
original_url: https://thebuildingcoder.typepad.com/blog/0953_reload_debug_cmd.html
post_number: 0953
reading_time_minutes: 7
series: structural
slug: reload_debug_cmd
source_file: 0953_reload_debug_cmd.htm
tags:
- elements
- filtering
- python
- revit-api
- transactions
- views
- windows
- structural
title: Reloading and Debugging External Commands on the Fly
word_count: 1377
---

﻿

### Reloading and Debugging External Commands on the Fly

After publishing the little sample code snippet yesterday showing how to
[load and reload an external Revit command](http://thebuildingcoder.typepad.com/blog/2013/05/load-your-own-external-command-on-the-fly.html) from
a byte stream to avoid locking the .NET assembly DLL, I realised that this can be combined rather nicely with my own
[external command lister](http://thebuildingcoder.typepad.com/blog/2013/05/external-command-lister-and-adding-ribbon-commands.html).

The combination of these two enables loading, debugging, reloading and re-debugging any simple external command on the fly, which has been a subject of
[regular discussion in the past](http://thebuildingcoder.typepad.com/blog/2012/12/reload-add-in-for-debug-without-restart.html).

By 'simple', I mean commands that do not have external dependencies or special loading requirements that make it difficult to load them lacking additional information beyond the .NET assembly path name.

The external command lister initially used the Assembly.LoadFrom method to load the .NET assembly, which locks the DLL.

Replacing this by reading the byte stream from the DLL and passing the result into Assembly.Load instead avoids that lock.

Adding a method to instantiate an external command implementation object and invoke its Execute method enables execution and debugging of an arbitrary external command.

Here is a live five-minute demonstration to show exactly what I mean:

To achieve this, I basically just added two methods to the existing code:

- [ExternalCommandLister.Launch](#2)
- [Command.OnDoubleClick](#3)

If you are impatient, uninterested or already know it all yourself, please go ahead and jump straight down to the
[summary and download](#4) section.

#### ExternalCommandLister Enhancements

I extended the ExternalCommandLister class by adding a Launch method taking the fully specified class name of the external command implementation.

I also modified the constructor to load the add-in assembly from a byte stream instead of directly from the .NET assembly file to avoid the DLL locking issue preventing subsequent reloading.

The Launch method requires the current instance of the ExternalCommandData to pass in to the external command it invokes, and the assembly we create needs to be kept available as well, so I added two new member variables for these.

It also checks the Result code returned by the invocation and reports that together with an optional error message in case of failure.

Here is the complete updated class implementation:

```python
class ExternalCommandLister
{
  string \_assembly\_filename;
  string[] \_external\_commmand\_class\_names;
  ExternalCommandData \_commandData;
  Assembly \_asm;

  public ExternalCommandLister(
    string assembly\_filename,
    ExternalCommandData commandData )
  {
    \_assembly\_filename = assembly\_filename;
    \_external\_commmand\_class\_names = null;
    \_commandData = commandData;

    if( !File.Exists( assembly\_filename ) )
    {
      throw new ArgumentOutOfRangeException(
        "assembly\_filename", "file not found" );
    }
    try
    {
      // No need to load the Revit API assemblies,
      // because we are ourselves a Revit API add-in
      // inside of Revit, so they are guaranteed to
      // be present.

      //Assembly revit = Assembly.LoadFrom( "C:/Program Files/Autodesk/Revit Architecture 2014/RevitAPI.dll" );
      //string root = "C:/Program Files/Autodesk Revit Architecture 2014/";
      //Assembly adWindows = Assembly.LoadFrom( root + "AdWindows.dll" );
      //Assembly uiFramework = Assembly.LoadFrom( root + "UIFramework.dll" );
      //Assembly revit = Assembly.LoadFrom( root + "RevitAPI.dll" );

      // Load the selected assembly into
      // the current application domain:

      //Assembly asm = Assembly.LoadFrom(
      //  assembly\_filename );

      // Load the selected assembly into the current
      // application domain via byte array to avoid
      // locking the DLL:

      byte [] assemblyBytes = File.ReadAllBytes(
        \_assembly\_filename );

      \_asm = Assembly.Load( assemblyBytes );

      if( null == \_asm )
      {
        Util.ErrorMsg( string.Format(
          "Unable to load assembly '{0}'",
          assembly\_filename ) );
      }
      else
      {
        IEnumerable<Type> types = \_asm.GetTypes()
          .Where<Type>( t =>
            null != t.GetInterface(
              "IExternalCommand" ) );

        \_external\_commmand\_class\_names = types
          .Select<Type, string>( t => t.FullName )
          .ToArray();
      }
    }
    catch( Exception ex )
    {
      Util.ErrorMsg( string.Format(
        "Exception '{0}' processing assembly '{1}'",
        ex.Message, assembly\_filename ) );
    }
  }

  public string AssemblyFilename
  {
    get
    {
      return Path.GetFileName( \_assembly\_filename );
    }
  }

  public string[] CommandClassnames
  {
    get
    {
      return \_external\_commmand\_class\_names;
    }
  }

  public Result Launch( string command\_name )
  {
    Debug.Assert(
      \_external\_commmand\_class\_names.Contains(
        command\_name ),
      "expected valid command name" );

    Type typ = \_asm.GetType( command\_name );

    object cmd = \_asm.CreateInstance( typ.FullName );

    string message = null;
    Autodesk.Revit.DB.ElementSet elements = null;

    object [] args = new object[] {
      \_commandData, message, elements };

    BindingFlags flags = (BindingFlags)
      ( (int) BindingFlags.Default
      | (int) BindingFlags.InvokeMethod );

    Result r = (Result) typ.InvokeMember( "Execute",
      flags, null, cmd, args );

    message = args[1] as string;

    int n = (null == message) ? 0 : message.Length;

    Util.InfoMsg( string.Format(
      "{0} returned {1}{2}{3}",
      command\_name, r,
      ( 0 == n ? "." : ": " ),
      ( 0 == n ? "" : message ) ) );

    return r;
  }
}
```

#### Command Class Enhancements

The external command implementation class obviously also needs to be extended in order to enable call the Launch method.

The initial version prompts the user to select an assembly to parse and lists the external command implementation classes it detects in a multi-line text box.

I initially thought of adding a context menu to the text box to enable launching a selected command.

However, it turned out to be easier to implement handling of a double click event instead.

The double click event handler detects the current line and extracts the full external command implementation class name from that to pass to the Launch method:

```python
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  /// <summary>
  /// Define the initial .NET assembly folder.
  /// </summary>
  const string \_assembly\_folder\_name
    = "C:\\ProgramData\\Autodesk\\Revit\\Addins\\2014";

  /// <summary>
  /// Select a .NET assembly file in the given folder.
  /// </summary>
  /// <param name="folder">Initial folder.</param>
  /// <param name="filename">Selected filename on success.</param>
  /// <returns>Return true if a file was successfully selected.</returns>
  static bool FileSelect(
    string folder,
    out string filename )
  {
    OpenFileDialog dlg = new OpenFileDialog();
    dlg.Title = "Select .NET Assembly or Cancel to Exit";
    dlg.CheckFileExists = true;
    dlg.CheckPathExists = true;
    dlg.InitialDirectory = folder;
    dlg.Filter = ".NET Assembly DLL Files (\*.dll)|\*.dll";
    bool rc = ( DialogResult.OK == dlg.ShowDialog() );
    filename = dlg.FileName;
    return rc;
  }

  void OnDoubleClick( object sender, EventArgs e )
  {
    Debug.Print( "{0}: {1}", sender, e );

    TextBox tb = sender as TextBox;

    if( null != tb )
    {
      string text = tb.Text;
      int i = tb.GetFirstCharIndexOfCurrentLine();
      text = text.Substring( i );
      int n = text.IndexOf( '\n' );
      if( 0 <= n )
      {
        text = text.Substring( 0, n );
      }
      text.Trim();
      Debug.Print( text );
      if( 0 < text.Length )
      {
        ExternalCommandLister lister = tb.Tag
          as ExternalCommandLister;

        lister.Launch( text );
      }
    }
  }

  void DisplayExternalCommands(
    string filename,
    IWin32Window owner,
    ExternalCommandData commandData )
  {
    ExternalCommandLister lister
      = new ExternalCommandLister(
        filename, commandData );

    string[] a = lister.CommandClassnames;
    int n = a.Length;

    System.Windows.Forms.Form form
      = new System.Windows.Forms.Form();

    form.Size = new Size( 400, 150 );

    form.Text = string.Format(
      "{0} defines {1} external command{2} - double click to launch",
      lister.AssemblyFilename, n,
      ( 1 == n ? "" : "s" ) );

    form.FormBorderStyle
      = FormBorderStyle.SizableToolWindow;

    System.Windows.Forms.TextBox tb
      = new System.Windows.Forms.TextBox();

    tb.Dock = System.Windows.Forms.DockStyle.Fill;
    tb.Location = new System.Drawing.Point( 0, 0 );
    tb.Multiline = true;
    tb.TabIndex = 0;
    tb.WordWrap = false;
    tb.ReadOnly = true;

    tb.Text = string.Join( "\r\n",
      lister.CommandClassnames );

    tb.Tag = lister;

    tb.DoubleClick += new EventHandler(
      OnDoubleClick );

    form.Controls.Add( tb );

    form.ShowDialog( owner );
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    IWin32Window revit\_window
      = new JtWindowHandle(
        ComponentManager.ApplicationWindow );

    string filename;

    while( FileSelect(
      \_assembly\_folder\_name,
      out filename ) )
    {
      DisplayExternalCommands( filename,
        revit\_window, commandData );
    }
    return Result.Succeeded;
  }
}
```

#### Summary and Download

One very neat benefit of this approach is that it almost completely replaces the functionality provided by RvtSamples, in that any simple external command can be immediately launched and debugged.
It is obviously also much more flexible, 100% totally flexible, in fact.
I cannot think of any way to make it more so.

The only remaining advantage of RvtSamples is basically its very inflexibility, the fact that it provides a neat user interface with a static overview of all available samples, plus optional additional add-ins that you specify yourself using include files or by editing and adding new entries to RvtSamples.txt itself.
You thus see what you have at your disposal, whereas the flexible new loader requires to to know it in advance yourself.

Come to think of it, this is probably exactly what the AddInManager does as well to enable reloading and re-debugging of already loaded add-ins without restarting Revit and regenerating the RVT project file.

Here is [JtExternalCommandLister2.zip](zip/JtExternalCommandLister2.zip) containing
the complete source code, Visual Studio solution and add-in manifest of the updated version 2014.0.0.2 of this external command.

I hope you find this useful, instructive and inspiring.