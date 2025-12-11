---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: code_example
optimization_date: '2025-12-11T11:44:13.870836'
original_url: https://thebuildingcoder.typepad.com/blog/0392_export_xml.html
post_number: 0392
reading_time_minutes: 3
series: general
slug: export_xml
source_file: 0392_export_xml.htm
tags:
- csharp
- elements
- filtering
- python
- revit-api
- sheets
- transactions
- views
title: Export Data to XML
word_count: 627
---

### Export Data to XML

Here is another issue of general interest and that arose during the
[DevLab in Waltham](http://thebuildingcoder.typepad.com/blog/2010/06/devlab-and-birthday.html) last week:

**Question:** How can I export some Revit model data to an XML file, for instance certain sheet properties?

**Answer:** That is actually very easy.
All you need to do is:

- Collect the Revit elements of interest, for instance all sheets.- Extract and save the data of interest from them, for instance the sheet number.- Iterate over the data collection and export it to XML.

You could of course also skip saving the data in an intermediate container and export it to an external file directly.
Adding the intermediate step might be useful, for instance to sort the data items or add other processing.

Exporting to XML is easy, because the .NET framework includes lots of XML formatting functionality.
It would also be easy to generate your own XML output file by hand, though.

Here is a super simple little ViewSheet data container to use for the intermediate storage:
```csharp
class SheetData
{
  public bool IsPlaceholder { get; set; }
  public string Name { get; set; }
  public string SheetNumber { get; set; }

  public SheetData( ViewSheet v )
  {
    IsPlaceholder = v.IsPlaceholder;
    Name = v.Name;
    SheetNumber = v.SheetNumber;
  }
}
```

Obviously this data holder is rather overly simplistic, but you can easily add other items of interest to you to it.

I implemented a new Building Coder sample command which performs the steps outlined above making use of this data container class:
```python
[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
class CmdSheetData : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    // retrieve all sheets

    FilteredElementCollector a
      = new FilteredElementCollector( doc );

    a.OfCategory( BuiltInCategory.OST\_Sheets );
    a.OfClass( typeof( ViewSheet ) );

    // create a collection of all relevant data

    List<SheetData> data = new List<SheetData>();

    foreach( ViewSheet v in a )
    {
      // create some data for each sheet and add
      // to some serializable collection called Data
      SheetData item = new SheetData( v );
      data.Add( item );
    }

    // write out data collection to xml

    XmlTextWriter w = new XmlTextWriter(
      "C:/SheetData.xml", null );

    w.Formatting = Formatting.Indented;
    w.WriteStartDocument();
    w.WriteComment( string.Format(
      " SheetData from {0} on {1} by Jeremy ",
      doc.PathName, DateTime.Now ) );

    w.WriteStartElement( "ViewSheets" );

    foreach( SheetData item in data )
    {
      w.WriteStartElement( "ViewSheet" );

      w.WriteElementString( "IsPlaceholder",
        item.IsPlaceholder.ToString() );

      w.WriteElementString( "Name", item.Name );

      w.WriteElementString( "SheetNumber",
        item.SheetNumber );

      w.WriteEndElement();
    }
    w.WriteEndElement();
    w.WriteEndDocument();
    w.Close();

    return Result.Succeeded;
  }
}
```

Here is the XML file resulting from running this command in a simple Revit model:
```csharp
<?xml version="1.0"?>
<!-- SheetData from C:\tmp\sheets\_and\_views.rvt
     on 2010-06-12 18:16:48 by Jeremy -->
<ViewSheets>
  <ViewSheet>
    <IsPlaceholder>False</IsPlaceholder>
    <Name>Unnamed</Name>
    <SheetNumber>A101</SheetNumber>
  </ViewSheet>
  <ViewSheet>
    <IsPlaceholder>False</IsPlaceholder>
    <Name>Unnamed</Name>
    <SheetNumber>A102</SheetNumber>
  </ViewSheet>
  <ViewSheet>
    <IsPlaceholder>False</IsPlaceholder>
    <Name>Unnamed</Name>
    <SheetNumber>A103</SheetNumber>
  </ViewSheet>
  <ViewSheet>
    <IsPlaceholder>False</IsPlaceholder>
    <Name>Unnamed</Name>
    <SheetNumber>A104</SheetNumber>
  </ViewSheet>
</ViewSheets>
```

I initially created this as a stand-alone application, and here is the add-in manifest file, source code, and Visual Studio solution for that, compressed in the archive file
[SheetData.zip](zip/SheetData.zip).

As said, since it seems useful to keep track of this simple XML exporting functionality for future use as well, I also added the command to The Building Coder samples.
Here is
[version 2011.0.72.0](zip/bc_11_72.zip)
of the complete source code and Visual Studio solution including the new command.
Since I use a
[RvtSamples include file](http://thebuildingcoder.typepad.com/blog/2008/11/loading-the-building-coder-samples.html)
BcSamples.txt to load The Building Coder samples, included in the archive file, I provide no separate add-in manifest files for these commands.