---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.1
content_type: code_example
optimization_date: '2025-12-11T11:44:13.508394'
original_url: https://thebuildingcoder.typepad.com/blog/0185_store_structured_data.html
post_number: 0185
reading_time_minutes: 6
series: general
slug: store_structured_data
source_file: 0185_store_structured_data.htm
tags:
- csharp
- elements
- family
- parameters
- python
- revit-api
title: Store Structured Data
word_count: 1136
---

### Store Structured Data

After yesterday's exciting and successful
[Revit Family API webcast](http://thebuildingcoder.typepad.com/blog/2009/07/revit-family-api-webcast.html),
here is a complete and mature solution to a frequently asked question that I mulled over for some time.

**Question:**
I read your post on
[creating a shared parameter on the model group](http://thebuildingcoder.typepad.com/blog/2009/06/model-group-shared-parameter.html),
which also shows how I can store arbitrary application data on almost any Revit element.
However, due to the way that Revit parameters are defined, the type of the data item contained within the parameter is limited to Double, Integer, String or ElementId.
What can I do to store arbitrary structured application data in a Revit project file?

**Answer:**
Just as you say, the data types supported by Revit parameters are limited to integer, real, string or element id.
The string data type will allow you to store any kind of data in a single parameter, as long as it is represented as a string.
Obviously, any kind of data can be encoded into string format.
You can either implement code to do this yourself, or make use of existing functionality provided by the .NET framework or other sources.

In case you are worrying about the data size, you can store strings of any length in a Revit string-valued parameter.
This technique has been used to store some pretty large strings in Revit parameters.
As far as we know, the only limitation is memory size, but of course Revit has never been exhaustively tested in this context.
So please be aware, as always, that you are using this technique at your own risk!

You can peek into the parameter string value using RvtMgdDbg;
although it will display only a certain maximum number of characters, you can still copy and paste the complete data into an editor using Ctrl + C and Ctrl + V.

The final encoding of binary data to a string can be achieved using base64 encoding, for example, which is provided by the .NET Convert.ToBase64String method.
The retrieval of the string data to a binary representation can be achieved using Convert.FromBase64String.

The data being stored can also be compressed by using the
[zip file format](http://en.wikipedia.org/wiki/ZIP_(file_format)),
for instance with the
[zlib library](http://www.gzip.org/zlib)
implementing compression and decompression algorithms for the zip data specification.
A port of zlib to C# is provided by
[#ziplib](http://www.icsharpcode.net/OpenSource/SharpZipLib),
whose usage is discussed in this MSDN article including some
[background info and an example usage](http://msdn.microsoft.com/en-us/magazine/cc164129.aspx).
Other links are provided in the article describing the file format.

Once you have the structured binary data in its final form, whether compressed or not, some method to convert it to string format needs to be defined and implemented.
A simple possibility is to implement a .NET class to hold the structured data and set the .NET attribute [Serializable] on it.
Then the .NET framework can automagically serialise it into a binary stream for you, which can then be either compressed or directly encoded into a base64 string for storage in the Revit string-valued parameter.

Here is a more detailed description of the technique using the Serializable attribute:

#### Generic Compound Parameters

**Question:**
How can we store any kind of custom data in a Revit model, be it a large XML file, a binary chunk, or other custom data? Shared parameters can be used to store text, numbers, URLs, Boolean Yes/No values, but not for any complex data structures such as custom objects and binary chunks.

**Answer:**
If the complex data is converted to a string, it can be stored in any text parameter. Per-object data can be stored in the appropriate Revit element parameters, per-document data on the Project Information singleton.

Various techniques exist to package complex data structures into strings, i.e. algorithmically pack and unpack: XML serialisation, SOAP serialisation, custom XML, custom algorithms and more. The drawback of custom algorithms is obviously the additional development effort required; furthermore, the resulting string size may be non-optimal. A simple solution is provided by the .NET framework:

- Declare the custom .NET class as [Serializable].- Serialise and deserialise it using a binary formatter.- Encode and decode the binary data to and from text using base64 encoding.

Converting from binary to text using only printable characters only causes a 33% size increase. Furthermore, it can be achieved in just a few lines of code in .NET.

Here is the skeleton of an example class declaration:

```csharp
  [Serializable()]
  public class B
{
. . .
}
```

This code converts an instance 'obj' into a string which can be stored in a Revit parameter by serialising into a binary stream and then encoding the binary data in a base64 string:

```csharp
  BinaryFormatter f = new BinaryFormatter();
  MemoryStream stream = new MemoryStream();
  f.Serialize( stream, obj );
  stream.Position = 0;

  long n2 = stream.Length;
  int n = (int) n2;
  byte[] buf = new byte[n];
  stream.Read( buf, 0, n );
  return Convert.ToBase64String( buf );
```

To retrieve the data from Revit, the string is read from the parameter, decoded back to binary data, and deserialised:

```csharp
  MemoryStream s = new MemoryStream( Convert.FromBase64String( s64 ) );
  s.Position = 0;

  BinaryFormatter f = new BinaryFormatter();
  return f.Deserialize( s );
```

Additional notes:

1. Make sure to use invisible parameters to avoid any user interface issues.- If you already have other binary chunks, custom packed by yourself or otherwise, then you do not need the first part to serialise the class instance, but just the base64 encoding of the binary chunk.- If you implement the simplest form listed above, then deserialisation will work only if your DLL is located in the same folder as Revit.exe.
       If it is not, you will see an error saying something like this:

```
System.Runtime.Serialization.SerializationException
Unable to find assembly 'StoreData, Version=1.0.0.0,
  Culture=neutral, PublicKeyToken=null'.
```

One solution is to ensure that assembly resides in same directory as acad.exe or Revit.exe.
To make it work regardless of where your DLL is located, you need a
[custom deserialisation helper class](http://www.codeproject.com/soap/Serialization_Samples.asp)
such as the following:

```python
public sealed class JtLinkBinder
  : System.Runtime.Serialization.SerializationBinder
{
  public override System.Type BindToType(
    string assemblyName,
    string typeName )
  {
    return Type.GetType( string.Format( "{0}, {1}",
      typeName, assemblyName ) );
  }
}
```

Using the custom serialisation binder class, the deserialisation to convert the string back to binary data looks like this:

```csharp
  BinaryFormatter f = new BinaryFormatter();
  // add this line to avoid the "unable to find assembly" issue:
  f.Binder = new JtLinkBinder();
  return f.Deserialize( s );
```

Here is a complete Visual Studio solution
[StoreData](zip/StoreData.zip)
implementing an example class and the storage functionality in a Revit parameter.