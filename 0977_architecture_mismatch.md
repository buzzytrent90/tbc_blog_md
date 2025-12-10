---
post_number: "0977"
title: "Recursively Disable Architecture Mismatch Warning"
slug: "architecture_mismatch"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'python', 'references', 'revit-api', 'views', 'walls', 'windows']
source_file: "0977_architecture_mismatch.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0977_architecture_mismatch.html"
---

### Recursively Disable Architecture Mismatch Warning

I recently explained the cause and resolution of the
[processor architecture mismatch warning MSB3270](http://thebuildingcoder.typepad.com/blog/2013/06/processor-architecture-mismatch-warning.html).

On closer consideration, I decided to fix it for all the
[ADN training material labs and samples](http://thebuildingcoder.typepad.com/blog/2013/06/migrating-the-adn-training-labs-to-revit-2014.html),
and actually for the entire collection of Revit SDK samples as well.

This is obviously not something you want to do manually, considering that it involves a modification of hundreds of project files.

Here are the steps involved in automating and testing this process:

- [Disable mismatch warning utility](#2)
- [Project file header and property group](#3)
- [Property group string constants](#4)
- [File processing and prerequisites check](#5)
- [Recursive directory traversal](#6)
- [Mainline](#7)
- [Potential enhancements](#8)
- [Results on SDK and ADN sample code](#9)
- [Other warnings, hitherto hidden](#10)
- [Download and updated ADN training material](#11)
- [Update for Gopi](#12)

#### Disable Mismatch Warning Utility

Since this involves hundreds of project files, I implemented a script to fulfil the task.
Normally, I would use either Python or a Unix shell script driving sed.
In order to share it more easily with you guys and the development team, I decided to implement a C# .NET command line executable instead, DisableMismatchWarning.exe.

It recursively searches for all C# and VB project files in and below the current working directory and adds the relevant property group to them.

#### Project File Header and Property Group

What property group? And how, exactly, please?

Well, for instance, looking at the AddSpaceAndZone SDK sample, one of the first, alphabetically speaking, here are the first five lines in the project file with the filename extension CSPROJ (view source or copy to an editor to see the truncated lines in full):

```
<Project ToolsVersion="4.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProductVersion>9.0.30729</ProductVersion>
```

As explained in the above-mentioned discussion, I can define the required new property group to suppress the undesired compilation warning message by adding the following lines (or single long line, if desired) before the existing property groups:

```
<PropertyGroup>
  <ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch>
    None
  </ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch>
</PropertyGroup>
```

#### Property Group String Constants

Here are the two string constants that I define to check for an existing tag in order to tell whether a given project file has already been processed, and the entire property group to add if that is not the case:

```csharp
const string \_arch\_mismatch\_tag
  = "<ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch>";

const string \_property\_group
  = "  <PropertyGroup>\r\n"
  + "    <ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch>\r\n"
  + "      None\r\n"
  + "    </ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch>\r\n"
  + "  </PropertyGroup>\r\n";
```

#### File Processing and Prerequisites Check

Before I make this modification to a file, I obviously want to be absolutely sure that:

- I have write access to it.
- It really is a valid Visual Studio project file.
- It has not already been processed, i.e. it does not already contain a ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch tag.
- A backup file is created before adding the property group.

Here is a C# .NET method that performs these steps on a given filename, adds the new property group lines to it, and reports the results on the command line:

```csharp
/// <summary>
/// Examine the given file.
/// Check that it is not read-only.
/// Check that it appears to be a valid
/// Visual Studio project file.
/// Check that it does not already contain a
/// ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch
/// tag.
/// If so, create a backup file and
/// add the property group.
/// </summary>
static bool AddPropertyGroup( string filename )
{
  bool rc = false;

  Console.WriteLine( filename + " -- examine..." );

  FileAttributes atts = File.GetAttributes(
    filename );

  bool isReadOnly = (FileAttributes.ReadOnly
    == (atts & FileAttributes.ReadOnly));

  if( isReadOnly )
  {
    Console.WriteLine( filename + " -- read-only." );
    return rc;
  }

  List<string> lines = new List<string>(
    File.ReadLines( filename ) );

  bool isProjectFile
    = lines[0].StartsWith( "<?xml version=" )
    && lines[1].StartsWith( "<Project " )
    && lines[2].Equals( "  <PropertyGroup>" );

  int n = 2;

  if( !isProjectFile )
  {
    isProjectFile
      = lines[0].StartsWith( "<Project " )
      && lines[1].Equals( "  <PropertyGroup>" );

    n = 1;
  }

  if( !isProjectFile )
  {
    Console.WriteLine( filename + " -- not a Visual Studio project." );
    return rc;
  }

  bool hasArchMismatchTag = 0 < lines.Where(
    a => a.Contains( \_arch\_mismatch\_tag ) ).Count();

  if( hasArchMismatchTag )
  {
    Console.WriteLine( filename + " -- already has an architecture mismatch tag." );
    return rc;
  }

  Console.WriteLine( filename + " -- process..." );

  File.Copy( filename, filename + ".bak", true );

  lines.Insert( n, \_property\_group );

  File.WriteAllLines( filename, lines );

  rc = true;

  return rc;
}
```

#### Recursive Directory Traversal

I would like to apply this method recursively to all files with a CSPROJ or VBPROJ filename extension.

I looked and quickly found the following suggestion for a
[recursive file search in .NET](http://stackoverflow.com/questions/437728/recursive-file-search-in-net) as
a starting point:

```csharp
/// <summary>
/// Return full path of all files mathing pattern
/// found recursively below the given root folder.
/// </summary>
/// <param name="root">Root starting folder</param>
/// <param name="searchPattern">Filename pattern</param>
/// <returns>Enumeration of full file paths</returns>
static IEnumerable<string> Search(
  string root,
  string searchPattern )
{
  Queue<string> dirs = new Queue<string>();

  dirs.Enqueue( root );

  while( 0 < dirs.Count )
  {
    string dir = dirs.Dequeue();

    // files

    string[] paths = null;

    try
    {
      paths = Directory.GetFiles(
        dir, searchPattern );
    }
    catch
    {
      // swallow
    }

    if( paths != null && 0 < paths.Length )
    {
      foreach( string file in paths )
      {
        yield return file;
      }
    }

    // sub-directories

    paths = null;
    try
    {
      paths = Directory.GetDirectories( dir );
    }
    catch
    {
      // swallow
    }

    if( paths != null && paths.Length > 0 )
    {
      foreach( string subDir in paths )
      {
        dirs.Enqueue( subDir );
      }
    }
  }
}
```

As explained by its author, it avoids the access denied exception thrown by the built-in recursive search and is lazily evaluated, returning results as soon as it finds them.

#### Mainline

With these two methods in place, there is nothing left for the console application mainline to do, beyond determining the current working directory and launching the process:

```csharp
/// <summary>
/// Add a new property group to suppress the
/// processor architecture mismatch warning MSB3270
/// to all Visual Studio C# and VB .NET project
/// files with the filename extension CSPROJ or
/// VBPROJ found recursively in the current working
/// directory or any of its subfolders.
/// </summary>
static int Main( string[] args )
{
  string root = Directory.GetCurrentDirectory();

  foreach( string match in Search( root, "\*proj" ) )
  {
    string path = match.ToLower();
    if( path.EndsWith( ".csproj" )
      || path.EndsWith( ".vbproj" ) )
    {
      AddPropertyGroup( match );
    }
  }
  return 0;
}
```

#### Potential Enhancements

It might be cleaner to handle this by reading and writing the project files as XML files, not just simple line-based text.

This utility will not overwrite a read-only file.
The standard Revit SDK installation sets all project files to read-only, so you may have to change that before you can run the utility.

#### Results on SDK and ADN Sample Code

Initially, it processed only 56 project files out of the total 169 provided in the Revit 2014 SDK.
Others were not recognised as valid Visual Studio projects.

For instance, some (or all?) VB project files apparently lack the initial `?xml` tag line, and the first attribute of the `Project` tag may be either `DefaultTargets` or `ToolsVersion`.

The criteria I initially applied to check for a valid Visual Studio project file were stricter than the ones listed above.
Relaxing them slightly enabled the utility to successfully recognise and process all 169 Revit SDK projects.

All of those pesky architecture mismatch warnings are now eliminated.

#### Other Warnings, Hitherto Hidden

Once the hundreds of architecture mismatch warnings were gone, I noticed five completely unexpected and unrelated new warning messages that previously were impossible to find in the multitude:

1. XML comment is not placed on a valid language element C:\a\lib\revit\2014\SDK\Samples\FamilyCreation\WindowWizard\CS\Command.cs 41 9 WindowWizard
2. ExternalCommandRegistration: Could not locate the assembly "RevitAddInUtility". Check to make sure the assembly exists on disk. If this reference is required by your code, you may get compilation errors.
3. The referenced component 'RevitAddInUtility' could not be found.
4. RoutingPreferenceTools: Could not locate the assembly "RevitAPIIFC". Check to make sure the assembly exists on disk. If this reference is required by your code, you may get compilation errors.
5. The referenced component 'RevitAPIIFC' could not be found.

All five were trivial to fix manually by removing the offending elements, i.e. an erroneous comment in the WindowWizard Command.cs and the two completely unnecessary RevitAddInUtility and RevitAPIIFC assembly references in the ExternalCommandRegistration and RoutingPreferenceTools projects.

#### Download and Updated ADN Training Material

You can download the complete source code and Visual Studio solution from my
[DisableMismatchWarning GitHub repository](https://github.com/jeremytammik/DisableMismatchWarning),
my second foray into this realm.

I also updated all the ADN training material labs and sample applications.

The last version I posted was set up to refer to the Revit Architecture 2014 API assemblies.

I now switched to Revit 2014 Onebox, to I changed the path to these from `C:\Program Files\Autodesk\Revit Architecture 2014` to `C:\Program Files\Autodesk\Revit 2014`.

The compilation of the last version of the ADN Revit API training labs that I posted still generated
[35 warnings](zip/adn_migr_2014_b.txt).
The new version reduces these to
[19 warnings](zip/adn_migr_2014_c.txt).

Here is
[adn\_src\_2014\_1.zip](file:////a/lib/revit/2014/adn/zip/adn_src_2014_1.zip) including
the updated source code, lab instructions, Visual Studio solutions and add-in manifests of the ADN Revit API training labs, Revit MEP sample, and Revit Structure labs, link and other samples, which was last published in three separate chunks:

- [ADN Training Labs for Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/06/migrating-the-adn-training-labs-to-revit-2014.html)
- [ADN AdnRme sample for Revit MEP 2014](http://thebuildingcoder.typepad.com/blog/2013/06/the-adn-sample-adnrme-for-revit-mep-2014.html)
- [ADN Training Material for Revit Structure 2014](http://thebuildingcoder.typepad.com/blog/2013/06/adn-training-material-for-revit-structure-2014.html)

#### Update for Gopi

After comparing my ADN sample code with my colleague Gopi's version, I discovered that he had corrected one deprecated API usage warning that I had not.

In order to get us better synchronised, I went ahead and fixed that, only to discover that once I started, I might as well go ahead and fix a few more.

Here is therefore an updated
[version 2014.0.0.2](file:////a/lib/revit/2014/adn/zip/adn_labs_2014_2.zip) of
the ADN Training Labs for Revit 2014, whose compilation now only produces
[7 warnings](zip/adn_migr_2014_d.txt).

It also includes Gopi's updated version of the lab instruction documents.

I would have said that 'we'll get there eventually', meaning 'down to zero warnings', only a few of these are actually intentional, for demonstration purposes...