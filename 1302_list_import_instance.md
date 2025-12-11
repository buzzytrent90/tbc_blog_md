---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.720277'
original_url: https://thebuildingcoder.typepad.com/blog/1302_list_import_instance.html
post_number: '1302'
reading_time_minutes: 7
series: elements
slug: list_import_instance
source_file: 1302_list_import_instance.htm
tags:
- csharp
- elements
- filtering
- levels
- revit-api
- selection
- transactions
- views
title: List All Import Instances
word_count: 1407
---

### List All Import Instances

Have you ever wondered whether you have any duplicate imported CAD instances in your model?

My colleague Nikolay Shulga from the Revit development team implemented a nice little end user utility to answer this question.
By the way, Nikolay and I go back a long time, way back in the beginning of the
[IAI and IFC project](https://en.wikipedia.org/wiki/BuildingSMART),
decades ago, in previous lives...

In Nikolay's own words:

A while ago I wrote a prototype app to list ImportInstance objects in a Revit project. The idea is to list duplicate instances – people importing the same data multiple times – and perhaps do other useful things.

I don’t think I can maintain that app – not enough bandwidth and demand to make it a part of Revit. I’m wondering whether you could make a blog entry out of the idea – hopefully someone picks it up and makes something out of it. If you can figure out a way to make it an open source app, even better.

- **Motivation** – People often import drawings – usually DWG, of course, but could be DGN – multiple times in current-view-only mode. That grows the size of the project, creates performance issues. We found that people don’t often know what data is imported or linked. That was an analysis tool that was developed for a specific project, look at performance issues related to imported data.
- **Spec** – The formal spec is short: 'Let’s list the imported data and report it in a minimally useful way'.
- **Implementation** – Well, you have it already; see below   :-)
- **Cool API aspects** – None, apart from the fact that simple stuff, if applied correctly, can add a disproportional amount of value.
- **Cool ways to use it** – Well, data exchange is not cool, but necessary. This helps users to keep their data in better shape – police against multiple imports, know what is in the file.
- **How it can be enhanced** – One idea we had was to have an imported data browser, similar to standard Revit view browser – or, perhaps, enhance the view browser.
- **A suitable sample model** – Simply create a Revit project, import a DWG multiple times and run the tool.

Here is the entire implementation:

```csharp
#region Namespaces
using System;
using System.Collections.Generic;
using System.Collections.Specialized;
using System.Diagnostics;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using System.IO;
#endregion

namespace ListImportInstances
{
  /// <summary>
  /// A generic interface to report imported
  /// data found in a specific project.
  /// </summary>
  interface IReportImportData
  {
    bool init( string projectName );
    void startReportSection( string sectionName );
    void logItem( string item );
    void setWarning();
    void done();
    string getLogFileName();
  }

  class SimpleTextFileBasedReporter : IReportImportData
  {
    public SimpleTextFileBasedReporter()
    {
    }

    public bool init( string projectFileName )
    {
      bool outcome = false;
      m\_currentSection = null;
      m\_warnUser = false;

      if( 0 != projectFileName.Length )
      {
        m\_projectFileName = projectFileName;
      }
      else
      {
        m\_projectFileName = "Default";
      }

      m\_logFileName = System.IO.Path.Combine(
        System.IO.Path.GetDirectoryName( m\_projectFileName ),
        System.IO.Path.GetFileNameWithoutExtension(
          m\_projectFileName ) ) + "-ListOfImportedData.txt";

      // Construct log file name from projectFileName
      // and try to open file. Project file name is
      // assumed to be valid (expected to be called
      // on an open doc).

      try
      {
        m\_outputFile = new StreamWriter( m\_logFileName );
        m\_outputFile.WriteLine( "List of imported CAD data in "
          + projectFileName );
        outcome = true;
      }
      catch( System.UnauthorizedAccessException )
      {
        TaskDialog.Show( "FindImports",
          "You are not authorized to create "
            + m\_logFileName );
      }
      catch( System.ArgumentNullException ) // oh, come on.
      {
        TaskDialog.Show( "FindImports",
          "That's just not fair. Null argument for StreamWriter()" );
      }
      catch( System.ArgumentException )
      {
        TaskDialog.Show( "FindImports",
          "Failed to create " + m\_logFileName );
      }
      catch( System.IO.DirectoryNotFoundException )
      {
        TaskDialog.Show( "FindImports",
          "That's not supposed to happen: directory not found: "
          + System.IO.Path.GetDirectoryName( m\_projectFileName ) );
      }
      catch( System.IO.PathTooLongException )
      {
        TaskDialog.Show( "FindImports",
          "The OS thinks the file name " + m\_logFileName
          + " is too long" );
      }
      catch( System.IO.IOException )
      {
        TaskDialog.Show( "FindImports",
          "An IO error has occurred while writing to "
          + m\_logFileName );
      }
      catch( System.Security.SecurityException )
      {
        TaskDialog.Show( "FindImports",
          "The OS thinks your access rights to "
          + System.IO.Path.GetDirectoryName( m\_projectFileName )
          + " are insufficient" );
      }
      return outcome;
    }

    public void startReportSection( string sectionName )
    {
      endReportSection();
      m\_outputFile.WriteLine();
      m\_outputFile.WriteLine( sectionName );
      m\_outputFile.WriteLine();

      m\_currentSection = sectionName;
    }

    public void logItem( string item )
    {
      m\_outputFile.WriteLine( item );
    }

    public void setWarning()
    {
      m\_warnUser = true;
    }

    public void done()
    {
      endReportSection();
      m\_outputFile.WriteLine();
      m\_outputFile.WriteLine( "The End" );
      m\_outputFile.WriteLine();
      m\_outputFile.Close();

      // Display "done" dialog, potentially open log file

      TaskDialog doneMsg = null;

      if( m\_warnUser )
      {
        doneMsg = new TaskDialog(
          "Potential issues found. Please review the log file" );
      }
      else
      {
        doneMsg = new TaskDialog(
          "FindImports completed successfully" );
      }

      doneMsg.AddCommandLink(
        TaskDialogCommandLinkId.CommandLink1,
        "Review " + m\_logFileName );

      switch( doneMsg.Show() )
      {
        default:
          break;

        case TaskDialogResult.CommandLink1:
          // Display the log file
          Process.Start( "notepad.exe", m\_logFileName );
          break;
      }
    }

    public string getLogFileName()
    {
      return m\_logFileName;
    }

    private void endReportSection()
    {
      if( null != m\_currentSection )
      {
        m\_outputFile.WriteLine();
        m\_outputFile.WriteLine( "End of "
          + m\_currentSection );
        m\_outputFile.WriteLine();
      }
    }

    private string m\_projectFileName;
    private string m\_logFileName;
    private StreamWriter m\_outputFile;
    private string m\_currentSection;

    /// <summary>
    /// Tell the user to review the log file
    /// </summary>
    private bool m\_warnUser;
  }

  [Transaction( TransactionMode.ReadOnly )]
  public class Command : IExternalCommand
  {
    private void listImports( Document doc )
    {
      FilteredElementCollector col
        = new FilteredElementCollector( doc )
          .OfClass( typeof( ImportInstance ) );

      NameValueCollection listOfViewSpecificImports
        = new NameValueCollection();

      NameValueCollection listOfModelImports
        = new NameValueCollection();

      NameValueCollection listOfUnidentifiedImports
        = new NameValueCollection();

      foreach( Element e in col )
      {
        // Collect all view-specific names.

        if( e.ViewSpecific )
        {
          string viewName = null;

          try
          {
            Element viewElement = doc.GetElement(
              e.OwnerViewId );
            viewName = viewElement.Name;
          }
          catch( Autodesk.Revit.Exceptions
            .ArgumentNullException ) // just in case
          {
            viewName = String.Concat(
              "Invalid View ID: ",
              e.OwnerViewId.ToString() );
          }

          if( null != e.Category )
          {
            listOfViewSpecificImports.Add(
              importCategoryNameToFileName(
                e.Category.Name ), viewName );
          }
          else
          {
            listOfUnidentifiedImports.Add(
              e.Id.ToString(), viewName );
          }
        }
        else
        {
          listOfModelImports.Add(
            importCategoryNameToFileName(
              e.Category.Name ), e.Name );
        }
      }

      IReportImportData logOutput
        = new SimpleTextFileBasedReporter();

      if( !logOutput.init( doc.PathName ) )
      {
        TaskDialog.Show( "FindImports",
          "Unable to create report file" );
      }
      else
      {
        if( listOfViewSpecificImports.HasKeys() )
        {
          logOutput.startReportSection(
            "View Specific Imports" );

          listResults( listOfViewSpecificImports,
            logOutput );
        }

        if( listOfModelImports.HasKeys() )
        {
          logOutput.startReportSection( "Model Imports" );
          listResults( listOfModelImports, logOutput );
        }

        if( listOfUnidentifiedImports.HasKeys() )
        {
          logOutput.startReportSection(
            "Unknown import instances" );
          listResults( listOfUnidentifiedImports,
            logOutput );
        }

        if( !sanityCheckViewSpecific(
          listOfViewSpecificImports, logOutput ) )
        {
          logOutput.setWarning();
          //TaskDialog.Show("FindImportedData",
          //"Possible issues found. Please review the log file");
        }

        logOutput.done();
      }
    }

    /// <summary>
    /// This is an import category. It is created from
    /// a CAD file name, with appropriate (number) added.
    /// We want to use the file name as a key for our
    /// list of import instances, so strip off the
    /// brackets.
    /// </summary>
    private string importCategoryNameToFileName(
      string catName )
    {
      string fileName = catName;
      fileName = fileName.Trim();

      if( fileName.EndsWith( ")" ) )
      {
        int lastLeftBracket = fileName.LastIndexOf( "(" );

        if( -1 != lastLeftBracket )
          fileName = fileName.Remove( lastLeftBracket ); // remove left bracket
      }

      return fileName.Trim();
    }

    private void listResults(
      NameValueCollection listOfImports,
      IReportImportData logFile )
    {

      foreach( String key in listOfImports.AllKeys )
      {
        logFile.logItem( key + ": "
          + listOfImports.Get( key ) );
      }
    }

    /// <summary>
    /// Run a few basic sanity checks on the list of
    /// view-specific imports.
    /// View-specific sanity is not the same as model
    /// sanity. Neither is necessarily sane.
    /// True means possibly sane, false means probably
    /// not.
    /// </summary>
    private bool sanityCheckViewSpecific(
      NameValueCollection listOfImports,
      IReportImportData logFile )
    {
      logFile.startReportSection(
        "Sanity check report for view-specific imports" );

      bool status = true;

      // Count number of entities per key.

      foreach( String key in listOfImports.AllKeys )
      {
        string[] levels = listOfImports.GetValues( key );
        if( levels != null && levels.GetLength( 0 ) > 1 )
        {
          logFile.logItem( "CAD data " + key
            + " appears to have been imported in "
            + "Current View Only mode multiple times. "
            + "It is present in views "
            + listOfImports.Get( key ) );
          status = false;
        }
      }
      return status;
    }

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIApplication uiapp = commandData.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Document doc = uidoc.Document;

      listImports( doc );

      return Result.Succeeded;
    }
  }
}
```

Many thanks to Nikolay for sharing this!

I followed his instructions and created the following trivial minimal sample model with three imports of a 2D DWG to test it:

![Three DWG import instances](img/three_dwg_imports.png)

Launching the external command generates a report of duplicate instances, stores it in a text file and displays the following task dialogue, with a command link to view the file:

![ListImportInstances task dialogue with command link](img/ListImportInstances01.png)

Clicking the command link opens the text in the default application, in this case Notepad:

![ListImportInstances report on duplicate instances](img/ListImportInstances02.png)

As always, the most up-to-date version including the full source code, Visual Studio solution and add-in manifest is provided by the
[ListImportInstances GitHub repository](https://github.com/jeremytammik/ListImportInstances).

The version presented above is
[release 2015.0.0.1](https://github.com/jeremytammik/ListImportInstances/releases/tag/2015.0.0.1).

I am looking forward to hearing from you how you make use of and enhance this.

Please feel free to fork and submit pull requests.