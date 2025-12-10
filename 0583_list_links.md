---
post_number: "0583"
title: "List Linked Files and TransmissionData"
slug: "list_links"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'references', 'revit-api', 'rooms', 'views', 'walls', 'windows']
source_file: "0583_list_links.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0583_list_links.html"
---

### List Linked Files and TransmissionData

Today is the day of our
[Revit 2012 API webcast](http://thebuildingcoder.typepad.com/blog/2011/05/revit-2012-api-webcast.html), in just six hours' time!
In case you missed it so far, I assume that there is still time for a
[last minute registration](http://usa.autodesk.com/adsk/servlet/item?id=10417086&siteID=123112&cname=Revit%20API,%20Webcast,%20May%2019%202011,%20201123).

I mentioned that one of the new features in the Revit 2012 API is the
[access to linked file information](http://thebuildingcoder.typepad.com/blog/2011/03/many-issues-resolved.html#1267916) and
the TransmissionData class which stores information about all of the external file references in a document.
The TransmissionData for a Revit project can be read without fully opening the document in the user interface.

Now the following simple question on this cropped up which prompted me to explore this in a little further depth:

**Question:** Is it possible to list all linked files with their paths Revit API?

**Answer:** Yes, it definitely is, and the Revit 2012 API makes it very easy to do so.

The overview of the
[access to linked file information](http://thebuildingcoder.typepad.com/blog/2011/03/many-issues-resolved.html#1267916) lists
the TransmissionData class mentioned above.
Its description in the Revit API help file RevitAPI.chm states:

A class representing information on all external file references in a document.

TransmissionData stores information on both the previous state and requested state of an external file reference. This means that it stores the load state and path of the reference from the most recent time this TransmissionData's document was opened. It also stores load state and path information for what Revit should do the next time the document is opened.

As such, TransmissionData can be used to perform operations on external file references without having to open the entire associated Revit document. The methods ReadTransmissionData and WriteTransmissionData can be used to obtain information about external references, or to change that information. For example, calling WriteTransmissionData with a TransmissionData object which has had all references set to LinkedFileStatus.Unloaded would cause no references to be loaded upon next opening the document.

This enables the implementation of code to list all the links of a given Revit document.
Here is a method ListLinks which exercises the new TransmissionData class to achieve exactly that:
```csharp
/// <summary>
/// List all DWG, RVT and other links of a given document.
/// </summary>
void ListLinks( ModelPath location )
{
  string path = ModelPathUtils
    .ConvertModelPathToUserVisiblePath( location );

  string content = string.Format(
    "The document at '{0}' ",
    path );

  List<string> links = null;

  // access transmission data in the given Revit file

  TransmissionData transData = TransmissionData
    .ReadTransmissionData( location );

  if( transData == null )
  {
    content += "does not have any transmission data";
  }
  else
  {
    // collect all (immediate) external references in the model

    ICollection<ElementId> externalReferences
      = transData.GetAllExternalFileReferenceIds();

    int n = externalReferences.Count;

    content += string.Format(
      "has {0} external reference{1}{2}",
      n, PluralSuffix( n ), DotOrColon( n ) );

    links = new List<string>( n );

    // find every reference that is a link

    foreach( ElementId refId in externalReferences )
    {
      ExternalFileReference extRef
        = transData.GetLastSavedReferenceData( refId );

      links.Add( string.Format( "{0} {1}",
        extRef.ExternalFileReferenceType,
        ModelPathUtils.ConvertModelPathToUserVisiblePath(
          extRef.GetPath() ) ) );
    }
  }
  Debug.Print( content );

  TaskDialog dlg = new TaskDialog( "List Links" );

  dlg.MainInstruction = content;

  if( null != links && 0 < links.Count )
  {
    string s = string.Join( "  \r\n",
      links.ToArray() );

    Debug.Print( s );

    dlg.MainContent = s;
  }
  dlg.Show();
}
```

It takes a ModelPath argument to determine the document to analyse.
The ModelPath can be instantiated from a normal file pathname using the ModelPathUtils class methods, for instance like this using the current active document in the Execute method of an external command:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  View view = commandData.View;

  if( null == view )
  {
    message = "Please run this command in an active document.";
    return Result.Failed;
  }
  else
  {
    Document doc = view.Document;

    ModelPath modelPath = ModelPathUtils
      .ConvertUserVisiblePathToModelPath(
        doc.PathName );

    ListLinks( modelPath );

    return Result.Succeeded;
  }
}
```

Here is the output it generates in a simple sample model:
![List links output](img/list_links.png)

The output is also listed as text in the Visual Studio debug output window:

```
The document at 'C:\tmp\linked_file2.rvt' has 3 external references:
  KeynoteTable RevitKeynotes_Metric.txt
  RevitLink walls.rvt
  CADLink TestHouse.dwg
```

Here is
[ListLinks.zip](zip/ListLinks.zip)
containing the entire Visual Studio solution implementing this command.

There is obviously still room for improvement to this.
Currently it is hardwired to analyse the active document.
It would probably be useful to be able to select one or more other documents.

Here are some simple ideas for a useful end user utility to list all links in user selected Revit files which could be easily implemented:

- Select one or more Revit documents.- Run ListLinks on them.- Display the results in a modeless dialogue and terminate the command.- Add a 'save to text file' button to the form.

Please let me know if you are interested in an improved version implementing some of these, and we'll see whether I ever get around to doing it.