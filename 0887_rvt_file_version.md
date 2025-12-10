---
post_number: "0887"
title: "Basic File Info and RVT File Version"
slug: "rvt_file_version"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'python', 'references', 'revit-api', 'schedules', 'views', 'windows']
source_file: "0887_rvt_file_version.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0887_rvt_file_version.html"
---

### Basic File Info and RVT File Version

I presented an initial solution in Python for
[determining the version of an RVT file on disk](http://thebuildingcoder.typepad.com/blog/2008/10/rvt-file-version.html) from
outside Revit making use of its OLE document Structured Storage format back in 2008.
David Echols later shared a more full-fledged
[C# Revit OLE Storage viewer](http://thebuildingcoder.typepad.com/blog/2010/06/open-revit-ole-storage.html) listing
more information and displaying the preview image.

Since then, the Revit API added support for reading basic RVT file info through the BasicFileInfo class.

Victor Chekalin now presents another C# console application showing how some of this data can be accessed independently of the Revit API.

Before looking at that, here are some other topics that also came up in the past day or two:

- [Building Setout Points](#2) used commercially
- [Screen recording](#3) with Camtasia on the Mac
- [Sustainable Autodesk Offices](#4)
- Reading the [RVT file version](#5) without using the Revit API

#### Building Setout Points Used in a Commercial Application

I am glad to say that one of the sample add-ins I am particularly happy with now made its way into a commercial application.

Here is an excerpt from the current Australian
[ARSoftwareSolutions](http://www.arsoftwaresolutions.com.au)
newsletter:

A recent meander through the TheBuildingCoder website led me to a routine for
[marking the corners of structural concrete corners](http://thebuildingcoder.typepad.com/blog/2012/08/structural-concrete-setout-point-add-in.html).
Once marked the points could be scheduled and provided to the builder as a file of set out point coordinates.
These can then be established on site using various surveying tools.
This seemed like something a lot of companies could benefit from so I set about including it in my software.
(Please refer to <http://thebuildingcoder.typepad.com/blog/rst> for more complete details.
Thank you to both Paul Hellawell of GHD and Jeremy Tammik for the idea and some of the coding.)

Of course I couldn't just leave it the way it was so I expanded on the concept, added extra functionality, and this now forms a part of ARUtils - Leader.

The basic concept is to mark all structural concrete corners with a "SetoutPoint" family.
Critical setout points can then use a "Major" type of that family to indicate they are a critical point.
Alternatively other points can be automatically designated as minor, centre points of curves, control points of splines, or just intermediate points along curves.
Points can be promoted to Major or demoted to Minor as required for construction or changing needs.

When automatically placed at a point, each marker has the X,Y, and Z coordinates assigned to the marker.
Once this has been done, the families can be scheduled and the points exported for use by the builder.
The routine also keeps track of the item that generated the points so points can be grouped on a generating object basis.
![Point markers](img/setout_points_ar1.jpeg)

*Points marked. Diagram shows Major, Minor, Centre, Control and Curve following points.*

![Tagged points](img/setout_points_ar2.jpeg)

*Tagged points*

![Key setout points](img/setout_points_ar3.png)

*Scheduled Keypoints*

![Schedule of all points](img/setout_points_ar4.png)

*Schedule - All points*

![Interface](img/setout_points_ar5.jpeg)

*The interface*

Note: You can automatically mark all structural concrete items, or pick specific items of any type to have markers placed.

Note: Should the project location change the coordinates can be automatically synchronised using the "Renumber / Update" option.

I am happy to see my efforts helping people and making their way out into the world :-)

#### Screen Recording on the Mac

On a completely different topic, I have been struggling a bit the past few days to set up a proper screen recording environment for myself to create a canned presentation.

I want to show some Powerpoint slides, record my narration, and also make use of Revit and Visual Studio for add-in demonstrations.

I planned to use Camtasia for this, as I have past experience with it and it does a good job.

The Revit and Visual Studio requirements led me to believe that I had to install Camtasia in my Windows virtual machine.

When I did that, I discovered that the Windows version that I had installed was European and therefore lacked the Windows Media Player, which caused Camtasia to fail. It took a while to find out, though.

I went off and downloaded a US version of Windows, created a new virtual machine, installed Revit, Visual Studio, Camtasia, and some other bits and pieces on it.

After doing all that, I discovered that the Camtasia output quality from this setup is suboptimal, and there are often problems with the audio getting lost completely.

After struggling a bit more with that, a colleague told me that the easiest thing to do is just to use Camtasia in the native Mac system and record the full screen.
I can display the Powerpoint slides in full screen, record my narration, and switch back and forth to a full-screen virtual Windows machine at any time.
So I wasted a lot of time there.
Now I can get going with my recording just as soon as I finish this post.

#### Sustainable Autodesk Offices

Back to the architectural realm, two new case studies on the green office design of the Autodesk offices in Milan and Farnborough are now live on the Autodesk Sustainable Design Center:

- [Farnborough](http://images.autodesk.com/adsk/files/customer_stories_us_4p_farnborough_r3.pdf)
- [Milano](http://images.autodesk.com/adsk/files/customer_stories_us_2p_milan_r3.pdf)

These projects were implemented by Morgan Lovell plc and Goring & Straja Architects, respectively.
The case studies describe the projects, challenges, solutions, results and learning experiences made.

I hope you find this interesting, useful and feel inspired to pursue the same kind of greening efforts in your own company :-)

#### Determining the RVT File Version Without Revit

Back to the main topic that I started discussing above.

This came up again prompted by a
[comment](http://thebuildingcoder.typepad.com/blog/2008/10/rvt-file-version.html?cid=6a00e553e168978833017d4061ff1d970c#comment-6a00e553e168978833017d4061ff1d970c) by
Victor Chekalin, or Виктор Чекалин, on the
[original post](http://thebuildingcoder.typepad.com/blog/2008/10/rvt-file-version.html).

He points to a stackflow discussion thread:
[How can I get the Revit file version using the Revit API?](http://stackoverflow.com/questions/14481652/how-can-i-get-the-revit-file-version-using-the-revit-api/14493458)

**Question:** In the Revit API I know that I can get the version of the Revit instance that is currently running:

- `ControlledApplication.VersionBuild`
- `ControlledApplication.VersionName`
- `ControlledApplication.VersionNumber`

However, I would like to get the version of a Revit file itself before I open it.
This way I could stop the automatic upgrade dialog that shows when a user opens an older Revit file in a newer version of Revit.
I'm using Revit 2013 and expecting files from 2011, 2012, and 2013.

**Answer from
[skeletank](http://stackoverflow.com/users/180529/skeletank):**
I took a closer look at the
[discussion group posting](http://www.revitforum.org/architecture-general-revit-questions/6853-there-tool-determine-what-version-revit.html) from
my question and found that the version information is actually human readable among the other gibberish of the file.
The ".rvt" file is stored in OLE format so you can see the content if you use a tool like
[Structured Storage Viewer](http://www.mitec.cz/ssv.html).
It will be located under *BasicFileInfo*.

If you wanted to then you could probably use an
[OLE library for .NET](http://stackoverflow.com/questions/2897328/is-there-any-library-to-access-ole-structured-storage-from-c) to
read the data but I used a `StreamReader` and a `Regex` instead.

Here is the regular expression:

`Revit\sArchitecture\s(?\d{4})\s(Build:\s(?\w*)\((?\w{3})\)\)`

Here is the code:
```csharp
private void ControlledApplication\_DocumentOpening(
  object sender,
  DocumentOpeningEventArgs e )
{
  FileInfo revitFileToUpgrade
    = new FileInfo( e.PathName );

  Regex buildInfoRegex = new Regex(
    @"Revit\sArchitecture\s(?<Year>\d{4})\s"
    +@"\(Build:\s(?<Build>\w\*)\((<Processor>\w{3})\)\)" );

  using( StreamReader streamReader =
    new StreamReader( e.PathName, Encoding.Unicode ) )
  {
    string fileContents = streamReader.ReadToEnd();

    Match buildInfo = buildInfoRegex.Match( fileContents );
    string year = buildInfo.Groups["Year"].Value;
    string build = buildInfo.Groups["Build"].Value;
    string processor = buildInfo.Groups["Processor"].Value;
  }
}
```

This will allow you to get the year, build number, and the type of processor.
Notice, however, that my version checks specifically for Architecture so you will need to modify it for MEP or Structural.

**Answer from
[Victor Chekalin](http://stackoverflow.com/users/989259/victor-chekalin):**

As you said, Revit file format is a Structured Storage document and information you need is stored in the BasicFileInfo stream.

Within the Revit 2013 API you can use the SavedInVersion method of the BasicFileInfo class.

Here is the
[full console application](http://pastebin.com/FUBcmSRx) demonstrating
how to extract BasicFileInfo data without the Revit API.

Unfortunately I don't now the format of the BasicFileInfoStream. But if you read it as string you can get version in which file was created.

Reading only the BasicFileInfo is much better than reading the whole file.
Imagine if the Revit project is over 500 MB.
You will read the whole file into memory when you call
```csharp
  string fileContents = streamReader.ReadToEnd();
```

Also, Regular expression on the huge file works slow.

I think you should use
```csharp
  var rawString = System.Text.Encoding.Unicode
    .GetString( rawData );
```

from my sample and use Regex in the rawString instead the whole file.

For your convenience, here is the full source code of Victor's solution:
```csharp
using System;
using System.IO;
using System.IO.Packaging;
using System.Reflection;
using System.Runtime.InteropServices;

namespace ConsoleApplication1
{
  class Program
  {
    private const string StreamName = "BasicFileInfo";

    static void Main( string[] args )
    {
      string pathToRevitFile = ( 1 == args.Length )
        ? args[0]
        : @"pathToRevitFile";

      if( !StructuredStorageUtils.IsFileStucturedStorage(
        pathToRevitFile ) )
      {
        throw new NotSupportedException(
          "File is not a structured storage file" );
      }

      var rawData = GetRawBasicFileInfo( pathToRevitFile );

      var rawString = System.Text.Encoding.Unicode.GetString(
        rawData );

      var fileInfoData = rawString.Split(
        new string[] { "\0", "\r\n" },
        StringSplitOptions.RemoveEmptyEntries );

      foreach( var info in fileInfoData )
      {
        Console.WriteLine( info );
      }
    }

    private static byte[] GetRawBasicFileInfo(
      string revitFileName )
    {
      if( !StructuredStorageUtils.IsFileStucturedStorage(
        revitFileName ) )
      {
        throw new NotSupportedException(
          "File is not a structured storage file" );
      }

      using( StructuredStorageRoot ssRoot =
          new StructuredStorageRoot( revitFileName ) )
      {
        if( !ssRoot.BaseRoot.StreamExists( StreamName ) )
          throw new NotSupportedException( string.Format(
            "File doesn't contain {0} stream", StreamName ) );

        StreamInfo imageStreamInfo =
            ssRoot.BaseRoot.GetStreamInfo( StreamName );

        using( Stream stream = imageStreamInfo.GetStream(
          FileMode.Open, FileAccess.Read ) )
        {
          byte[] buffer = new byte[stream.Length];
          stream.Read( buffer, 0, buffer.Length );
          return buffer;
        }
      }
    }
  }

  public static class StructuredStorageUtils
  {
    [DllImport( "ole32.dll" )]
    static extern int StgIsStorageFile(
      [MarshalAs( UnmanagedType.LPWStr )]
      string pwcsName );

    public static bool IsFileStucturedStorage(
      string fileName )
    {
      int res = StgIsStorageFile( fileName );

      if( res == 0 )
        return true;

      if( res == 1 )
        return false;

      throw new FileNotFoundException(
        "File not found", fileName );
    }
  }

  public class StructuredStorageException : Exception
  {
    public StructuredStorageException()
    {
    }

    public StructuredStorageException( string message )
      : base( message )
    {
    }

    public StructuredStorageException(
      string message,
      Exception innerException )
      : base( message, innerException )
    {
    }
  }

  public class StructuredStorageRoot : IDisposable
  {
    StorageInfo \_storageRoot;

    public StructuredStorageRoot( Stream stream )
    {
      try
      {
        \_storageRoot
          = (StorageInfo)InvokeStorageRootMethod(
            null, "CreateOnStream", stream );
      }
      catch( Exception ex )
      {
        throw new StructuredStorageException(
          "Cannot get StructuredStorageRoot", ex );
      }
    }

    public StructuredStorageRoot( string fileName )
    {
      try
      {
        \_storageRoot
          = (StorageInfo)InvokeStorageRootMethod(
            null, "Open", fileName, FileMode.Open,
            FileAccess.Read, FileShare.Read );
      }
      catch( Exception ex )
      {
        throw new StructuredStorageException(
          "Cannot get StructuredStorageRoot", ex );
      }
    }

    private static object InvokeStorageRootMethod(
      StorageInfo storageRoot,
      string methodName,
      params object[] methodArgs )
    {
      Type storageRootType
        = typeof( StorageInfo ).Assembly.GetType(
          "System.IO.Packaging.StorageRoot",
          true, false );

      object result = storageRootType.InvokeMember(
        methodName,
        BindingFlags.Static | BindingFlags.Instance
        | BindingFlags.Public | BindingFlags.NonPublic
        | BindingFlags.InvokeMethod,
        null, storageRoot, methodArgs );

      return result;
    }

    private void CloseStorageRoot()
    {
      InvokeStorageRootMethod( \_storageRoot, "Close" );
    }

    #region Implementation of IDisposable

    public void Dispose()
    {
      CloseStorageRoot();
    }

    #endregion

    public StorageInfo BaseRoot
    {
      get { return \_storageRoot; }
    }
  }
}
```

For your even greater convenience, I am also providing
[RevitBasicFileInfo.zip](zip/RevitBasicFileInfo.zip) containing
the full Visual Studio solution that I created to test it.

Here is the result of running this on a sample model on my system:

![Basic File Info](img/basic_file_info.png)

Many thanks to Victor for his clear explanation and solution!

**Addendum:** To get access to the StreamInfo class you need reference the WindowsBase.dll assembly.

The question marks displayed in the result console above are due to the fact that one part of BasicFileInfo is encoded in Unicode and another in BigEndianUnicode.

The second part is rendered legible by accessing it like this instead:

```
  var rawString = System.Text.Encoding
    .BigEndianUnicode.GetString( rawData );
```

Here is the same file information presented twice, once in standard little endian and once in big endian decoding:

![Basic File Info in little and big endian decoding](img/basic_file_info2.png)

This is a summary of the resulting readable text fields:

- Little-endian:

- Autodesk Revit 2013 (Build: 20120716\_1115)
- C:\j\tmp\rvt\floor\_slab.rvt

- Big-endian:

- 00000000-0000-0000-0000-000000000000
- Worksharing: Not enabled
- Username:
- Central Model Path:
- Revit Build: Autodesk Revit 2013 (Build: 20120716\_1115)
- Last Save Path: C:\j\tmp\rvt\floor\_slab.rvt
- Open Workset Default: 3
- Project Spark File: 0
- Central Model Identity: 00000000-0000-0000-0000-000000000000
- Locale when saved: ENU