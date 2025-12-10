---
post_number: "0035"
title: "Loading The Building Coder Samples"
slug: "loading_bc_samples"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'references', 'revit-api', 'walls']
source_file: "0035_loading_bc_samples.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0035_loading_bc_samples.html"
---

### Loading The Building Coder Samples

As I write these blog entries, I continuously add new commands to The Building Coder sample project.
While doing so, I wish keep my list of external commands up to date with a minimum of effort.
I have several important requirements:

- Maintain one single file listing the command information.
- Automatically create an entry point in Revit for each command.
- Reuse the same file in all flavours of Revit, with no additional effort.
- Integrate the command data with other ADN samples that I maintain.

The easiest way to achieve these goals is to make use of the Revit SDK sample external application RvtSamples, which I described in more detail in the post on
[Loading SDK Samples](http://thebuildingcoder.typepad.com/blog/2008/09/loading-sdk-sam.html).

So far, however, I have actually been duplicating the information, because on one hand, I wanted to keep it in a separate file. On the other hand, I had to copy it into the RvtSamples input file, RvtSamples.txt.

For quite a while, I have been thinking that a simple solution to this problem would be for RvtSamples to support an #include mechanism, like the C and C++ pre-processors do. The C #include pragma allows one file to reference another one, and the reference behaves just as if the content of the referenced file were inserted into the referencing one at the #include pragma location.

Once I started looking at the implementation, it turned out to be very simple. The original RvtSamples module Application.cs reads in the contents of RvtSamples.txt in the line:

```csharp
  string[] lines = File.ReadAllLines(filename);
```

I defined a new helper method ReadAllLinesWithInclude and replaced that call by:

```csharp
  string[] lines = ReadAllLinesWithInclude( filename );
```

Here is the implementation of ReadAllLinesWithInclude:

```csharp
char[] \_trimChars = new char[] {
  ' ', '"', '\'', '<', '>' };

string[] ReadAllLinesWithInclude(
  string filename )
{
  if( !File.Exists( filename ) )
  {
    ErrorMsg( filename + " not found." );
    return new string[] { };
  }
  string[] lines = File.ReadAllLines(
    filename );

  int n = lines.GetLength(0);
  ArrayList all\_lines = new ArrayList( n );
  foreach( string line in lines )
  {
    string s = line.TrimStart();
    if( s.ToLower().StartsWith( "#include" ) )
    {
      string filename2 = s.Substring( 8 );
      filename2 = filename2.Trim( \_trimChars );
      all\_lines.AddRange( ReadAllLinesWithInclude(
        filename2 ) );
    }
    else
    {
      all\_lines.Add( line );
    }
  }
  return all\_lines.ToArray( typeof( string ) )
    as string[];
}
```

Now, I have added only two lines to the end of RvtSamples.txt, one each for the ADN and The Building Coder samples, and these two lines will never need to be edited again:

```
  #include C:/a/j/adn/train/revit/2009/src/AdnSamples.txt
  #include C:/a/j/adn/train/revit/2009/src/bc/BcSamples.txt
```

Here are the entire contents of BcSamples.txt, with the header informational comments removed, and the binary path somewhat shortened for the sake of clarity:

```
  /&ADN/&Bc/List Walls
  List wall lengths and areas
  C:/bin/BuildingCoder.dll
  BuildingCoder.CmdListWalls

  /&ADN/&Bc/Wall Dimensions
  Extract wall solid and list all its dimensions
  C:/bin/BuildingCoder.dll
  BuildingCoder.CmdWallDimensions

  /&ADN/&Bc/Relationship Inverter
  Determine opening > wall host relationships and invert them to wall > opening
  C:/bin/BuildingCoder.dll
  BuildingCoder.CmdRelationshipInverter

  /&ADN/&Bc/Filter Performance
  Compare Type Filter versus using an anonymous method to filter elements
  C:/bin/BuildingCoder.dll
  BuildingCoder.CmdFilterPerformance

  /&ADN/&Bc/Element Materials
  Retrieve building element materials
  C:/bin/BuildingCoder.dll
  BuildingCoder.CmdGetMaterials

  /&ADN/&Bc/Azimuth
  Calculate azimuth
  C:/bin/BuildingCoder.dll
  BuildingCoder.CmdAzimuth

  /&ADN/&Bc/Bounding Box
  Retrieve Element Bounding Box
  C:/bin/BuildingCoder.dll
  BuildingCoder.CmdBoundingBox

  /&ADN/&Bc/Slab Boundaries
  Determine polygonal floor slab boundary loops
  C:/bin/BuildingCoder.dll
  BuildingCoder.CmdSlabBoundary

  /&ADN/&Bc/Slab Sides
  Determine floor slab side faces
  C:/bin/BuildingCoder.dll
  BuildingCoder.CmdSlabSides
```

By the way, note that you can use an ampersand sign '&' to define keyboard mnemonics to provide shortcut access to the menu items when defining them through an external application. I find this especially handy when I need to start a command repeatedly to debug an application.

Here is my updated version of
[RvtSamples 1.0.1.0 supporting #include](http://thebuildingcoder.typepad.com/blog/files/RvtSamples1010.zip).
I incremented the version number from 1.0.0.0 to 1.0.1.0.