---
post_number: "0828"
title: "Exporting Parameter Data to Excel"
slug: "export_param_excel"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'parameters', 'python', 'revit-api', 'selection', 'sheets', 'transactions']
source_file: "0828_export_param_excel.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0828_export_param_excel.html"
---

### Exporting Parameter Data to Excel

Did you observe the international
[talk like a pirate](http://en.wikipedia.org/wiki/International_Talk_Like_a_Pirate_Day) holiday
yesterday?

I did not, I'm sorry to say.
I only became aware of it this morning trying to find some reason why it was so quiet yesterday.
Aaarrr!

Anyway, here is an update of an age-old sample application, originally created by Miroslav Schonauer for the very first Revit API training classes after the first introduction of the Revit API back in the Revit 2008 timeframe.

Miro put together a whole collection of useful samples, which survived time and changes extraordinarily well and made it through to become the
[Xtra Revit API training labs](http://thebuildingcoder.typepad.com/blog/2012/04/xtra-adn-revit-2013-api-training-labs.html).

I received several queries lately on how to export data to Excel, with various follow-up options such as later including data from linked files as well, and enabling a re-import of modified data.

#### The FireRating SDK Sample

Actually, talking about re-importing the exported data, a simple example of this has been around all along as well.
The FireRating SDK sample demonstrates:

- Creating and populating a new shared parameter in the model.- Exporting all its values to Excel.- Importing modified data back in from Excel to update the Revit model.

I mentioned this important sample a number of times, ever since the early beginnings of the blog, often to demonstrate how to add a shared parameter to various element types in the model:

- [Adding a shared parameter to a DWG file](http://thebuildingcoder.typepad.com/blog/2008/11/adding-a-shared-parameter-to-a-dwg-file.html).- [Defining a new parameter](http://thebuildingcoder.typepad.com/blog/2008/11/defining-a-new-parameter.html).- [Adding a shared parameter to an RFA file](http://thebuildingcoder.typepad.com/blog/2009/06/adding-a-shared-parameter-to-an-rfa-file.html).- [Model group shared parameter](http://thebuildingcoder.typepad.com/blog/2009/06/model-group-shared-parameter.html).- [Parameter access and scheduling](http://thebuildingcoder.typepad.com/blog/2010/05/parameter-access-and-scheduling.html).

One very special use of this sample was in the special 100th anniversary description of
[utilizing Revit API resources](http://thebuildingcoder.typepad.com/blog/2009/02/utilizing-revit-api-resources.html) to
get a beginner up to speed and running.

Because of its importance, this sample also made its way into the Revit API Xtra training labs in the shape of three commands corresponding to the steps listed above:

- Lab4\_3\_1\_CreateAndBindSharedParam- Lab4\_3\_2\_ExportSharedParamToExcel- Lab4\_3\_3\_ImportSharedParamFromExcel

Of course, the importance of shared parameters for add-in was greatly diminished by the introduction of
[extensible storage](http://thebuildingcoder.typepad.com/blog/2011/04/extensible-storage.html).

#### Parameter Export to Excel Considerations

Anyway, back to the subject at hand.

The external command Lab4\_2\_ExportParametersToExcel in the ADN Xtra labs implements exporting all parameter data of all Revit elements to Excel.
'All' is relative, though...

It bases the selection of parameters on the standard Revit API Element.Parameters property.
An attempt is made to export the values of the parameters returned in this collection, and others are ignored.
Many elements do have other parameters associated with them as well, as demonstrated by
[BipChecker, the built-in parameter explorer](http://thebuildingcoder.typepad.com/blog/2011/09/unofficial-parameters-and-bipchecker.html).
They could easily be added to the export as well, of course.

Furthermore, there are of course a multitude of other important data items not stored in parameters that might be interesting to export and potentially modify and re-import as well.
It might be worthwhile checking whether
[RDBLink](http://thebuildingcoder.typepad.com/blog/2009/11/adding-a-column-to-rdblink-export.html) does
anything like that...
RDBLink was originally part of the Revit 2008, 2009 and 2010 SDKs, then matured into a
[subscription pack product](http://thebuildingcoder.typepad.com/blog/2011/06/subscription-packs.html).

The choice of Excel as an export target is not mine, nor would it normally be so.
Due to popular demand, though, this command makes use of the Excel COM interface and .NET Interop to access that.
It launches or attaches to a running instance of Excel and makes it visible, so you can see the work sheets and parameters being added one by one.
It might be faster to make Excel invisible, and faster still to use some other library to generate the XLS file without direct access to Excel, and faster still choosing some completely different file format such as SLK, CSV, or, heaven forbid, TXT.

The command selects all elements in the entire model, both types (e.g. family symbols) and non-type elements.
Each element is identified in the export by its element id, and a flag is added to tell whether it is a type or not.

The original implementation exported only model elements.
Support for
[all elements](http://thebuildingcoder.typepad.com/blog/2010/06/filter-for-all-elements.html) was
implemented by creating a union of two complementary filtered element collectors, which is easily possible and normally
[not recommended](http://thebuildingcoder.typepad.com/blog/2010/06/filter-for-all-elements.html).

The elements are sorted by category, and elements with no valid category are ignored.
The elements are sorted into a dictionary of separate containers for each category.

The category names are used to create individual work sheets in a new Excel work book.

The category names need some massaging to confirm with the Excel work sheet naming conventions; the name must:

- Not be blank.- Not exceed 31 characters.- Not contain any of the following characters:   :   \   /   ?   \*   [   or   ].

In the original implementation, the entire parameter access was encapsulated in a try-catch exception handler.
Many elements returned null, though, triggering and exception and slowing down the process enormously.
Every exception handler is resource intensive and will significantly slow down execution and consume resources.
An exception handler should be designed to handle unexpected, exceptional cases only;
[exceptions should be exceptional](http://www.jacopretorius.net/2009/10/exceptions-should-be-exceptional.html).
Adding a preceding check for a null parameter before actually trying to access it speeded things up significantly.
Maybe the exception handler can be removed completely?

As said, the elements are identified in the resulting Excel data by their element id.
This is not a very safe method of identifying elements, because the element id may change, e.g. by work sharing operations.
It would be safer to use the UniqueId instead.

When the new work book is set up, Excel automatically adds a couple of work sheets to it.
The number of default work sheets added can be defined in the Excel application settings.
I initially implemented code to remove the unneeded work sheets, but later commented it out to simply let them be.
They don't really hurt.

For each category, all the elements are examined to determine what parameters they contain.
A column is added to the work sheet for each parameter, and a header is set up listing the parameter name.
We iterate over the elements in that category and add a row listing their element id, type flag, and parameter values for each.

#### Parameter Export to Excel Implementation

Let's summarise the steps:

- Collect all elements and sort them by category.- Attach to or launch Excel and create a new work book.- Loop through all the categories and set up a work sheet for each.- Determine all parameters for the given category and create the work sheet header listing them.- Iterate over each elements of the category and export its element id, type flag and parameter values.- Report the results.

Here is the code implementing this as a read-only external command:
```python
/// <summary>
/// Export all parameters for each model
/// element to Excel, one sheet per category.
/// </summary>
[Transaction( TransactionMode.ReadOnly )]
public class Lab4\_2\_ExportParametersToExcel
  : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    // Extract and group the data from Revit in a
    // dictionary, where the key is the category
    // name and the value is a list of elements.

    Stopwatch sw = Stopwatch.StartNew();

    Dictionary<string, List<Element>> sortedElements
      = new Dictionary<string, List<Element>>();

    // Iterate over all elements, both symbols and
    // model elements, and them in the dictionary.

    ElementFilter f = new LogicalOrFilter(
      new ElementIsElementTypeFilter( false ),
      new ElementIsElementTypeFilter( true ) );

    FilteredElementCollector collector
      = new FilteredElementCollector( doc )
        .WherePasses( f );

    string name;

    foreach( Element e in collector )
    {
      Category category = e.Category;

      if( null != category )
      {
        name = category.Name;

        // If this category was not yet encountered,
        // add it and create a new container for its
        // elements.

        if( !sortedElements.ContainsKey( name ) )
        {
          sortedElements.Add( name,
            new List<Element>() );
        }
        sortedElements[name].Add( e );
      }
    }

    // Launch or access Excel via COM Interop:

    X.Application excel = new X.Application();

    if( null == excel )
    {
      LabUtils.ErrorMsg(
        "Failed to get or start Excel." );

      return Result.Failed;
    }
    excel.Visible = true;

    X.Workbook workbook = excel.Workbooks.Add(
      Missing.Value );

    X.Worksheet worksheet;

    // We cannot delete all work sheets,
    // Excel requires at least one.
    //
    //while( 1 < workbook.Sheets.Count )
    //{
    //  worksheet = workbook.Sheets.get\_Item(1) as X.Worksheet;
    //  worksheet.Delete();
    //}

    // Loop through all collected categories and
    // create a worksheet for each except the first.
    // We sort the categories and work trough them
    // from the end, since the worksheet added last
    // shows up first in the Excel tab.

    List<string> keys = new List<string>(
      sortedElements.Keys );

    keys.Sort();
    keys.Reverse();

    bool first = true;

    int nElements = 0;
    int nCategories = keys.Count;

    foreach( string categoryName in keys )
    {
      List<Element> elementSet
        = sortedElements[categoryName];

      // Create and name the worksheet

      if( first )
      {
        worksheet = workbook.Sheets.get\_Item( 1 )
          as X.Worksheet;

        first = false;
      }
      else
      {
        worksheet = excel.Worksheets.Add(
          Missing.Value, Missing.Value,
          Missing.Value, Missing.Value )
          as X.Worksheet;
      }

      name = ( 31 < categoryName.Length )
        ? categoryName.Substring( 0, 31 )
        : categoryName;

      name = name
        .Replace( ':', '\_' )
        .Replace( '/', '\_' );

      worksheet.Name = name;

      // Determine the names of all parameters
      // defined for the elements in this set.

      List<string> paramNames = new List<string>();

      foreach( Element e in elementSet )
      {
        ParameterSet parameters = e.Parameters;

        foreach( Parameter parameter in parameters )
        {
          name = parameter.Definition.Name;

          if( !paramNames.Contains( name ) )
          {
            paramNames.Add( name );
          }
        }
      }
      paramNames.Sort();

      // Add the header row in bold.

      worksheet.Cells[1, 1] = "ID";
      worksheet.Cells[1, 2] = "IsType";

      int column = 3;

      foreach( string paramName in paramNames )
      {
        worksheet.Cells[1, column] = paramName;
        ++column;
      }
      var range = worksheet.get\_Range( "A1", "Z1" );

      range.Font.Bold = true;
      range.EntireColumn.AutoFit();

      int row = 2;

      foreach( Element e in elementSet )
      {
        // First column is the element id,
        // second a flag indicating type (symbol)
        // or not, both displayed as an integer.

        worksheet.Cells[row, 1] = e.Id.IntegerValue;

        worksheet.Cells[row, 2] = (e is ElementType)
          ? 1
          : 0;

        column = 3;

        string paramValue;

        foreach( string paramName in paramNames )
        {
          paramValue = "\*NA\*";

          Parameter p = e.get\_Parameter( paramName );

          if( null != p )
          {
            //try
            //{
              paramValue
                = LabUtils.GetParameterValue( p );
            //}
            //catch( Exception ex )
            //{
            //  Debug.Print( ex.Message );
            //}
          }

          worksheet.Cells[row, column++]
            = paramValue;
        } // column

        ++nElements;
        ++row;

      } // row

    } // category == worksheet

    sw.Stop();

    TaskDialog.Show( "Parameter Export",
      string.Format(
        "{0} categories and a total "
        + "of {1} elements exported "
        + "in {2:F2} seconds.",
        nCategories, nElements,
        sw.Elapsed.TotalSeconds ) );

    return Result.Succeeded;
  }
}
```

I ran the command on the basic architectural sample model rac\_basic\_sample\_project.rvt.
It takes about two minutes to complete, produces this
[XLS file output](zip/rac_basic_sample_project.xlsx) containing
124 work sheets, and displays the following message on terminating:

![Export parameter values to Excel](img/export_param_excel.png)

The timing is not very relevant, since I was doing other things at the same time on the machine.

Here is
[adn\_labs\_2013\_2012-09-19.zip](zip/adn_labs_2013_2012-09-19.zip) containing
the complete source code, Visual Studio solution and RvtSamples include file of the ADN training labs with the updated Lab4\_2\_ExportParametersToExcel external command.