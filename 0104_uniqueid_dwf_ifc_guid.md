---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.9
content_type: code_example
optimization_date: '2025-12-11T11:44:13.368845'
original_url: https://thebuildingcoder.typepad.com/blog/0104_uniqueid_dwf_ifc_guid.html
post_number: '0104'
reading_time_minutes: 8
series: general
slug: uniqueid_dwf_ifc_guid
source_file: 0104_uniqueid_dwf_ifc_guid.htm
tags:
- csharp
- doors
- elements
- python
- revit-api
- walls
- windows
title: UniqueId versus DWF and IFC GUID
word_count: 1586
---

### UniqueId versus DWF and IFC GUID

One frequent question from developers has been on how to correlate the GUIDs or globally unique identifiers used when exporting models to DWF and IFC from Revit with the element ids and unique ids used internally in the Revit model.

#### GUID and UniqueId

A
[GUID](http://en.wikipedia.org/wiki/Guid)
is a 16-byte, i.e. 128 bit number.
It is commonly split up into several fields of varying lengths and written in groups of 8-4-4-4-12 hexadecimal characters,
i.e. 32 characters to represent the 16 bytes or 128 bits.

On the other hand, we have the Revit UniqueId.
Every element has such a unique identifier, which is returned through the API as a string.
This string is formatted in groups of 8-4-4-4-12-8 hexadecimal characters.
It is thus similar to the standard GUID format, but has 8 additional characters at the end.
These 8 additional hexadecimal characters are large enough to store 4 bytes or a 32 bit number, which is exactly the size of a Revit element id.

If I create a couple of walls in a row, I note that the GUID part of the UniqueId is the same for all of them, and the last 8 bytes differ and exactly represent their individual element ids.
Here are the various ids of two walls, exported to both DWF and IFC:

```
Id=130315; Class=Wall; Category=Walls; Name=Generic - 200mm;
  UniqueId = 60f91daf-3dd7-4283-a86d-24137b73f3da-0001fd0b;
  Dwf Guid = 60f91daf-3dd7-4283-a86d-24137b720ed1;
  Ifc Guid = 1W_HslFTT2WwXj91DxSWxH

Id=130335; Class=Wall; Category=Walls; Name=Generic - 200mm;
  UniqueId = 60f91daf-3dd7-4283-a86d-24137b73f3da-0001fd1f;
  Dwf Guid = 60f91daf-3dd7-4283-a86d-24137b720ec5
  Ifc Guid = 1W_HslFTT2WwXj91DxSWx5
```

You can see that the first 16 bytes or 32 hex characters of the unique identifiers are identical.
This portion is apparently internally called EpisodeId in Revit.
The unique ids only differ in the 4 byte or 8 hex character suffix at the end.
In the case above, the two differing suffixes for the walls are indeed their element ids in hexadecimal representation: 130315 decimal equals 0x1fd0b, and 130335 equals 0x1fd1f.

There are several items of interest in the data displayed for these two walls:

- The UniqueId does not adhere to the standard GUID format.
- The DWF GUID does.
- The IFC GUID looks completely different from both.

Obviously, it would be useful to convert between the three different formats.
The following two sections discuss exactly that:

- Converting back and forth between the DWF and IFC GUID formats.
- How to convert from the Revit UniqueId to the standard GUID form used in DWF.

#### IFC GUID Encoding

First of all, it is important to understand that the IFC GUID is completely identical to the DWF GUID.
It is simply encoded in a different format to create a shorter text representation to save a few bytes in the text-based IFC files.

I was actually involved in the original definition of this approach, as well as once being the secretary of the
[IAI](http://www.iai-tech.org),
and to my surprise I see my name is still listed in the source code history of CreateGuid\_64.c, the module implementing this encoding algorithm.

To demonstrate the encoding and decoding process, I made use of the original CreateGuid\_64.c C module and compiled it into a stand-alone DLL named IfcGuid.dll. Further down, we discuss this as well as a stand-alone .NET console application written in C# to test it.

#### UniqueId to GUID Encoding

Now we know that the DWF and IFC GUIDs are in fact identical, we can use the term GUID to refer to both of them.

The next question is how the GUID used for exporting a Revit element is defined from its internal Revit database properties such as element id and UniqueId.

Four different rules are used for various groups of Revit elements to generate their GUID used for DWF and IFC export.
They may combine data from the Revit element id and the UniqueId both of the element itself and some other element:

1. If the element is being exported after an IFC import, we preserve the GUID originally contained in the IFC Entity. This is a rare case.
2. If there is a one-to-one correspondence between a Revit Element and an IFC entity, we create the GUID by taking the GUID in the EpisodeId of the element and XORing it with the element id.
3. For special IFC Entities that have no corresponding element in Revit, such as IfcBuilding and IfcProject, and for some elements that are duplicated, such as an opening for a window or door instance, we create the GUID by taking the Detach GUID of the project and XORing it with the element id.
4. For other IFC Entities, we create a GUID on the fly.

Given the string representation of a Revit UniqueId in the variable 'a', the following code implements the GUID creation:

```csharp
Print( "Revit UniqueId: " + a );

Guid episodeId = new Guid( a.Substring( 0, 36 ) );

int elementId = int.Parse( a.Substring( 37 ),
  NumberStyles.AllowHexSpecifier );

Print( "    EpisodeId: " + episodeId.ToString() );
Print( string.Format( "    ElementId: {0} = {1}",
  elementId.ToString(), elementId.ToString( "x8" ) ) );

int last\_32\_bits = int.Parse( a.Substring( 28, 8 ),
  NumberStyles.AllowHexSpecifier );

int xor = last\_32\_bits ^ elementId;

a = a.Substring( 0, 28 ) + xor.ToString( "x8" );

Print( "          Guid: " + a + "\n" );
```

For the first wall listed above, the result of executing this is:

```
Revit UniqueId: 60f91daf-3dd7-4283-a86d-24137b73f3da-0001fd0b
     EpisodeId: 60f91daf-3dd7-4283-a86d-24137b73f3da
     ElementId: 130315 = 0001fd0b
          Guid: 60f91daf-3dd7-4283-a86d-24137b720ed1
```

So we now know how to calculate the GUID of a Revit element, and the previous section explained that it is possible to convert back and forth between the GUID and the IFC format.

The time has come to make use of all this and put together a utility to exercise and test all the possible conversions.

#### IFC GUID and UniqueId Encoder and Decoder

To demonstrate the Revit element GUID generation and IFC GUID encoding and decoding process, I wrote a little stand-alone program that implements both. It provides the following functionality:

- Convert Revit UniqueId to GUID.
- Encode GUID to IFC format.
- Decode IFC format GUID to standard representation.

It makes use of the original IFC CreateGuid\_64.c C module, compiled it into a stand-alone DLL named IfcGuid.dll.
The rest of the program is written in .NET.
Therefore, we need a C# client interface for the unmanaged C code to test the encoding and decoding from the standard GUID into the IFC format and back again.
The two important functions defined in CreateGuid\_64.c for achieving this are

```csharp
char \* getString64FromGuid( const GUID \*pGuid, char \* buf, int len );
BOOL getGuidFromString64( const char \*string, GUID \*pGuid );
```

I will spare you the C code.
It is included in the complete solution available below.
The C# statements used to provide access to these two functions from .NET look like this:

```csharp
[DllImport( "IfcGuid.dll" )]
static extern void getString64FromGuid(
  ref Guid guid,
  StringBuilder s64 );

[DllImport( "IfcGuid.dll" )]
static extern bool getGuidFromString64(
  string s64,
  ref Guid guid );
```

I implemented two little wrapper methods in C# to provide more comfortable access to the two functions:

```csharp
static Guid String64ToGuid( string s64 )
{
  Guid guid = new Guid();
  getGuidFromString64( s64.ToString(), ref guid );
  return guid;
}

static string GuidToString64( Guid guid )
{
  StringBuilder s = new StringBuilder(
    "                        " );
  getString64FromGuid( ref guid, s );
  return s.ToString();
}
```

Here is the mainline of the program:

```python
static void Main( string[] args )
{
  Print( "IfcGuid by Jeremy Tammik." );
  if( 0 == args.Length )
  {
    Print( "Do you have a GUID for me?" );
  }
  else
  {
    string a = args[0];
    int n = a.Length;

    if( 45 == n )
    {
      Print( "Revit UniqueId: " + a );

      Guid episodeId = new Guid( a.Substring( 0, 36 ) );

      int elementId = int.Parse( a.Substring( 37 ),
        NumberStyles.AllowHexSpecifier );

      Print( "    EpisodeId: " + episodeId.ToString() );
      Print( string.Format( "    ElementId: {0} = {1}",
        elementId.ToString(), elementId.ToString( "x8" ) ) );

      int last\_32\_bits = int.Parse( a.Substring( 28, 8 ),
        NumberStyles.AllowHexSpecifier );

      int xor = last\_32\_bits ^ elementId;

      a = a.Substring( 0, 28 ) + xor.ToString( "x8" );

      Print( "          Guid: " + a + "\n" );
    }

    if( 22 == a.Length )
    {
      Guid guid = String64ToGuid( a );
      string s = GuidToString64( guid );
      Print( "Ifc base64 encoding: " + a );
      Print( "    decoded to GUID: " + guid.ToString() );
      Print( "  reencoded to IFC: " + s );
      Debug.Assert( s.Equals( a ),
        "expected decode + encode to return original guid" );
    }
    else
    {
      Guid guid = new Guid( a );
      string s1 = ToBase64.ToBase64.GetId( guid );
      string s2 = Convert.ToBase64String( guid.ToByteArray() );
      string s3 = new ShortGuid.ShortGuid( guid ).ToString();
      string s4 = GuidToString64( guid );
      Guid guid2 = String64ToGuid( s4 );

      Print( "Original 128-bit GUID: " + guid.ToString() );
      Print( "    ToBase64 encoding: " + s1 );
      Print( " .NET base64 encoding: " + s2 );
      Print( "  ShortGuid encoding: " + s3 );
      Print( " IFC base 64 encoding: " + s4 );
      Print( " decoded back to GUID: " + guid2.ToString() );
      Debug.Assert( guid2.Equals( guid ),
        "expected encode + decode to return original guid" );
    }
  }
}
```

It checks whether an argument is given.
If not, it asks you to provide a GUID of some kind.
If the string length of the argument provided is 45, we can assume that it is a Revit UniqueId and generate a standard GUID from it using the algorithm described above.
If its string length is 22, we can assume that it is an IFC encoded GUID and decode it.
Otherwise, it is assumed to be a standard GUID and we encode it into IFC format.
In both cases, we apply the inverse operation to the result and ensure that the original data is restored.

Here is the complete
[Visual Studio 2005 solution](zip/ifcguid.zip).