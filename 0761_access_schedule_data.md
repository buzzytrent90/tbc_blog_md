---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.0
content_type: code_example
optimization_date: '2025-12-11T11:44:14.538246'
original_url: https://thebuildingcoder.typepad.com/blog/0761_access_schedule_data.html
post_number: '0761'
reading_time_minutes: 7
series: general
slug: access_schedule_data
source_file: 0761_access_schedule_data.htm
tags:
- csharp
- elements
- filtering
- parameters
- python
- revit-api
- schedules
- selection
- sheets
- views
- windows
title: Schedule API and Access to Schedule Data
word_count: 1316
---

### Schedule API and Access to Schedule Data

Somehow I seem to be spending the vast majority of my time just shovelling information back and forth, preparing for conferences and trainings and answering questions.
It is always a great pleasure for me to actually get in a teeny weenie bit of programming in between, so I am happy that I managed to find time to create the new little sample application described below.

One of the many exciting enhancements in the Revit 2013 API is the new Schedule API.
Here is the description from the Revit API help file RevitAPI.chm What's New section:

#### Schedule API

Several new classes have been added to allow schedule views to be created, modified, and added to drawing sheets. Major new classes include:

- The ViewSchedule class that represents the schedule view. Its create methods are used to create new schedules.- The ScheduleField class for the individual fields in a schedule.- The ScheduleSheetInstance class represents schedules placed on sheets. The create method creates an instance of a schedule on a sheet.- The ScheduleDefinition class defines the contents of a schedule view, including:
        1. Basic properties that determine the kind of schedule, such as the schedule's category.- A set of fields that become the columns of the schedule.- Filters that restrict the set of elements visible in the schedule.- Sorting and grouping criteria.

#### Running API commands in Schedule Views

API commands may now be active when a Schedule is active in the current document.

#### Schedule export

The ViewSchedule class has been added to the API and the method ViewSchedule.Export method exports an existing schedule to a text file.

As I already mentioned, the ability to
[create a schedule](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2) is
demonstrated by the
[ScheduleCreation SDK sample](http://thebuildingcoder.typepad.com/blog/2012/04/developer-center-and-sdk-update.html#22).

Before even dreaming of creating new schedules, however, many people are interested in simply accessing the data in existing ones.

#### Accessing Schedule Data

Fitting in nicely with this, a couple of queries recently came in on accessing schedule data.

The Revit API currently does not provide a method to access and cycle through the schedule data directly.

The new Revit 2013 schedule export functionality described above provides for a very simple workaround, however:
you can use the ViewSchedule.Export method to export the schedule data to a text file, and then parse the resulting file to access the data.

The ViewSchedule.Export method takes a ViewScheduleExportOptions argument which provides the following properties to enable specification of various text file output settings:

- ColumnHeaders: how to export column headers. Default is MultipleRows.- FieldDelimiter: how to delimit fields. Default is Tab.- HeadersFootersBlanks: whether to export group headers, footers, and blank lines. Default is true.- TextQualifier: how to qualify text fields. Default is DoubleQuote.

#### ScheduleData Sample Add-in

I created a sample application to demonstrate accessing the schedule data, and not just that. It also implements the following interesting additional features:

- Access and read back the schedule data from the exported text file into your application.- Separately determine the schedule name, the column header names, and the schedule row data.- Populate a DataTable instance with the retrieved schedule data.- Create a new Windows form with a docked DataGridView in it programmatically, avoiding the need to define a hard-coded form in your add-in.- Attach the form to the Revit main window, so that it is minimised and restored together with Revit.- Populate the DataGridView from the DataTable and display the results.

I use the undocumented AdWindows assembly to
[determine the Revit parent window](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html#5).

It returns the main window Revit handle in the static property ApplicationWindow of the ComponentManager class in the Autodesk.Windows namespace.

Here is the JtWindowHandle class that I use to convert the IntPtr returned by that property to the IWin32Window required by the Form.Show method:
```python
  /// <summary>
  /// Wrapper class for converting
  /// IntPtr to IWin32Window.
  /// </summary>
  public class JtWindowHandle : IWin32Window
  {
    IntPtr \_hwnd;

    public JtWindowHandle( IntPtr h )
    {
      Debug.Assert( IntPtr.Zero != h,
        "expected non-null window handle" );

      \_hwnd = h;
    }

    public IntPtr Handle
    {
      get
      {
        return \_hwnd;
      }
    }
  }
```

Exporting the schedule data from Revit is really simple.
Here is the implementation to export all schedule data in the entire model to separate text files with the filename extension TXT, defined by \_ext, in a hard-coded folder, defined by \_export\_folder\_name:
```csharp
  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .OfClass( typeof( ViewSchedule ) );

  ViewScheduleExportOptions opt
    = new ViewScheduleExportOptions();

  foreach( ViewSchedule vs in col )
  {
    Directory.CreateDirectory(
      \_export\_folder\_name );

    vs.Export( \_export\_folder\_name,
      vs.Name + \_ext, opt );
  }
```

The application then asks the user to select any one of the resulting text output files at a time to display its data in the data grid view. When the user cancels the file selection, the command is terminated:
```csharp
  string filename;

  while( FileSelect( \_export\_folder\_name,
    out filename ) )
  {
    DisplayScheduleData( filename,
      revit\_window );
  }
```

Here is the DisplayScheduleData method that programmatically sets up a new Windows form, populates it with a docked DataGridView instance, and populates that with the schedule data retrieved from the output text files by the ScheduleDataParser class:
```csharp
  void DisplayScheduleData(
    string filename,
    IWin32Window owner )
  {
    ScheduleDataParser parser
      = new ScheduleDataParser( filename );

    System.Windows.Forms.Form form
      = new System.Windows.Forms.Form();
    form.Size = new Size( 400, 400 );
    form.Text = \_caption\_prefix + parser.Name;

    DataGridView dg = new DataGridView();
    dg.AllowUserToAddRows = false;
    dg.AllowUserToDeleteRows = false;
    dg.AllowUserToOrderColumns = true;
    dg.Dock = System.Windows.Forms.DockStyle.Fill;
    dg.Location = new System.Drawing.Point( 0, 0 );
    dg.ReadOnly = true;
    dg.TabIndex = 0;
    dg.DataSource = parser.Table;
    dg.Parent = form;

    form.ShowDialog( owner );
  }
```

You could obviously make the form modeless instead of modal by calling Show instead of ShowDialog here. Since we are not accessing the Revit API from the form, there is no danger in leaving it open after the external command terminates. That would enable navigation and exploration of the schedule data in a modeless form in parallel with continued work in the Revit model.

Finally, here is the main point, the ScheduleDataParser class that I implemented to extract the schedule name, column headers, and schedule data from the text file and populate a DataTable instance with it:
```python
class ScheduleDataParser
{
  /// <summary>
  /// Default schedule data file field delimiter.
  /// </summary>
  static char[] \_tabs = new char[] { '\t' };

  /// <summary>
  /// Strip the quotes around text strings
  /// in the schedule data file.
  /// </summary>
  static char[] \_quotes = new char[] { '"' };

  string \_name = null;
  DataTable \_table = null;

  /// <summary>
  /// Schedule name
  /// </summary>
  public string Name
  {
    get { return \_name; }
  }

  /// <summary>
  /// Schedule columns and row data
  /// </summary>
  public DataTable Table
  {
    get { return \_table; }
  }

  public ScheduleDataParser( string filename )
  {
    StreamReader stream = File.OpenText( filename );

    string line;
    string[] a;

    while( null != ( line = stream.ReadLine() ) )
    {
      a = line
        .Split( \_tabs )
        .Select<string, string>( s => s.Trim( \_quotes ) )
        .ToArray();

      // First line of text file contains
      // schedule name

      if( null == \_name )
      {
        \_name = a[0];
        continue;
      }

      // Second line of text file contains
      // schedule column names

      if( null == \_table )
      {
        \_table = new DataTable();

        foreach( string column\_name in a )
        {
          DataColumn column = new DataColumn();
          column.DataType = typeof( string );
          column.ColumnName = column\_name;
          \_table.Columns.Add( column );
        }

        \_table.BeginLoadData();

        continue;
      }

      // Remaining lines define schedula data

      DataRow dr = \_table.LoadDataRow( a, true );
    }
    \_table.EndLoadData();
  }
}
```

Here is a sample schedule in Revit to test this on:

![Schedule data in Revit view](img/schedule_data_rvt_view.png)

This is the resulting data grid view representation:

![Schedule data in data grid view](img/schedule_data_grid_view.png)

Here is
[ScheduleData.zip](zip/ScheduleData.zip) containing
the complete source code, Visual Studio solution and add-in manifest for this add-in.

On a closely related note, here is another question that just came in:

**Question:** Can calculated values in schedules be transferred to a shared parameter with some API code?

**Answer:** Sure.
Just access the schedule data as demonstrated above, and then write the relevant value to the shared parameter.