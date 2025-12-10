---
post_number: "0280"
title: "Import LandXML Surface"
slug: "landxml"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'python', 'revit-api', 'views']
source_file: "0280_landxml.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0280_landxml.html"
---

### Import LandXML Surface

Someone recently asked for an API within Revit for handling LandXML data.
There is no API for LandXML, but there is nothing to stop us from using standard XML parsing to read LandXML files and use the standard Revit API to generate terrain and other objects.

Here is a solution provided by Emmanuel Weyermann of Autodesk demonstrating how to parse a LandXML file and generate a topography surface in the Revit database from its data.
I added it as a new external command CmdLandXml to The Building Coder sample application.
It prompts the user to select a LandXML input file, parses it to obtain a list of XYZ points, and passes this list to the dedicated API method NewTopographySurface to create the surface.
Here is the entire code of the external command's Execute method:
```python
Application app = commandData.Application;
Document doc = app.ActiveDocument;

W.OpenFileDialog dlg = new W.OpenFileDialog();

dlg.Filter = "LandXML files (\*.xml)|\*.xml";

dlg.Title = "Import LandXML and "
  + "Create TopographySurface";

if( dlg.ShowDialog() != W.DialogResult.OK )
{
  return CmdResult.Cancelled;
}

XmlDocument xmlDoc = new XmlDocument();
xmlDoc.Load( dlg.FileName );

XmlNodeList pnts
  = xmlDoc.GetElementsByTagName( "Pnts" );

char[] separator = new char[] { ' ' };
double x = 0, y = 0, z = 0;
XYZ xyz;

XYZArray pts = app.Create.NewXYZArray();

for( int k = 0; k < pnts.Count; ++k )
{
  for( int i = 0;
    i < pnts[k].ChildNodes.Count; ++i )
  {
    int j = 1;

    string text = pnts[k].ChildNodes[i].InnerText;
    string[] coords = text.Split( separator );

    foreach( string coord in coords )
    {
      switch( j )
      {
        case 1:
          x = Double.Parse( coord );
          break;
        case 2:
          y = Double.Parse( coord );
          break;
        case 3:
          z = Double.Parse( coord );
          break;
        default:
          break;
      }
      j++;
    }
    xyz = new XYZ( x, y, z );
    pts.Append( xyz );
  }
}

TopographySurface surface
  = doc.Create.NewTopographySurface( pts );

return CmdResult.Succeeded;
```

You need to change the view property to display topography to see the resulting surface after running this command.

One limitation of this sample is that it does not check what units are defined on the LandXML file.

Here is a zip file
[LandXMLfiles.zip](zip/LandXMLfiles.zip)
containing Emmanuel's original solution as well as a sample LandXML input file.
The resulting topography surface displayed by Revit after reading in and processing the file looks like this:
![LandXML topography surface](img/landxml_topography_surface.png)

Here is
[version 1.1.0.57](zip/bc11057.zip)
of the complete Building Coder sample source code and Visual Studio solution including the new command.

Many thanks to Emmanuel for providing this interesting sample!