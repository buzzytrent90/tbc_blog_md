---
post_number: "1723"
title: "Add Template Ini File"
slug: "add_template_ini_file"
author: "Jeremy Tammik"
tags: ['revit-api', 'sheets', 'views', 'windows']
source_file: "1723_add_template_ini_file.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1723_add_template_ini_file.html"
---

### Accessing and Modifying Settings in the Ini File
Some interesting settings are stored in and can be modified by editing the Revit ini file `Revit.ini`.
Peter [@pgerz](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/859112) pointed
out yet another possibility in his answer to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [adding a project template to 'New Project' via API](https://forums.autodesk.com/t5/revit-api-forum/adding-project-template-to-new-project-via-api/m-p/8585348):
\*\*Question:\*\* I noticed that the UI method for adding a project template to the 'New Project' dialog on the Start Window is by going to Options > File Locations and clicking the little plus symbol.
Is there any way to achieve this same effect using the API?
I would like to add templates to the dropdown.
\*\*Answer:\*\* You can do it by editing the ini file with standard .NET functions; it is located at:
- C:\Users\%username%\AppData\Roaming\Autodesk\Revit\Autodesk Revit 2019\Revit.ini
In the section `[DirectoriesENU]`, modify the setting `DefaultTemplate`.
Example:
```csharp
string oriFile = @""
+ Environment.GetEnvironmentVariable( "appdata" )
+ @"\Autodesk\Revit\Autodesk Revit 2019\Revit.ini";
string tmpFile = @"c:\temp\11.ini";
if( System.IO.File.Exists( oriFile ) )
{
using( StreamReader sr = new StreamReader(
oriFile, Encoding.Unicode ) )
{
StreamWriter sw = new StreamWriter( tmpFile,
false, Encoding.Unicode );
string inputLine = "";
while( ( inputLine = sr.ReadLine() ) != null )
{
if( inputLine.StartsWith( "DefaultTemplate=" ) )
{
if( inputLine.Contains( "Example_SCHEMA.rte" ) )
{
// do nothing
}
else
{
inputLine = inputLine
+ @", Example_SCHEMA=C:\temp\Example_SCHEMA.rte";
}
}
sw.WriteLine( inputLine );
}
sw.Close();
}
System.IO.File.Replace( tmpFile, oriFile, null );
}
```
Many thanks to Peter for this solution!
![Stencil](img/stencil.png)