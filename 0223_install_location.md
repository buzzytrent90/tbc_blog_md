---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.578225'
original_url: https://thebuildingcoder.typepad.com/blog/0223_install_location.html
post_number: '0223'
reading_time_minutes: 5
series: general
slug: install_location
source_file: 0223_install_location.htm
tags:
- csharp
- revit-api
- windows
title: Revit Install Location
word_count: 901
---

### Revit Install Location

Quite a while back, we discussed the
[Revit install path and product GUIDs](http://thebuildingcoder.typepad.com/blog/2009/02/revit-install-path-and-product-guids.html).
Last week, Jim Bish of
[Inlet Technology](http://www.inlettechnology.com) wrote me a nice message making use of that and including VB code demonstrating how to use the install path and also how to determine the product GUID from the registry without having to manually type it in or copy and paste it from a table.

**Jim says**: It has always been my policy to give back to others as they have given to me.
Here are two functions that will retrieve the ProductCode and subsequently, the InstallLocation of Revit Structure 2010. It can be modified for any install of Revit, and is primarily used to get a path to the revit.ini file for Appends, and Repairs, and Uninstalls of my External Commands and External Applications.
```vbnet
Public Function GetRevitProductCode()
  Dim key As String \_
    = "SOFTWARE\Autodesk\Revit\Autodesk Revit Structure 2010"
  Using rgk1 As RegistryKey \_
    = Registry.LocalMachine.OpenSubKey(key)
    For Each sSubkeyName As String In \_
      rgk1.GetSubKeyNames()
      Using rgk2 As RegistryKey \_
        = rgk1.OpenSubKey(sSubkeyName)
        Try
          Return rgk2.GetValue("ProductCode")
        Catch generatedExceptionName As Exception
          Console.WriteLine("Error Occured")
        End Try
      End Using
    Next
  End Using
End Function
Public Function GetRevitInstallLocation( \_
  ByVal sProdCode As String)
  Dim key As String \_
    = "SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall"
  Using rgk1 As RegistryKey \_
    = Registry.LocalMachine.OpenSubKey(key)
    Using rgk2 As RegistryKey \_
      = rgk1.OpenSubKey(sProdCode)
      Try
        Return rgk2.GetValue("InstallLocation")
      Catch generatedExceptionName As Exception
        Console.WriteLine("Error Occured")
      End Try
    End Using
  End Using
End Function
```

I used these two functions as a basis for implementing a new Building Coder sample command CmdInstallLocation to ensure that this useful functionality is easily found and available.
It determines and lists the product code and the install location for the currently executing Revit application and also tries to do so for all three flavours, Architecture, MEP and Structure.
On my system, the result of running it looks like this:

![Revit install locations](img/install_location.png)

I factored out various useful bits of functionality from Jim's code as follows:

Define the Revit product and Windows uninstall registry paths:

```csharp
const string \_reg\_path\_uninstall
  = @"SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall";

const string \_reg\_path\_for\_flavour
  = @"SOFTWARE\Autodesk\Revit\Autodesk Revit {0} 2010";
```

Note that the registry path for the Revit product is actually a format string which still needs to be populated with the Revit product flavour.
The correct string to specify the flavour can be obtained directly from the ProductType enumeration.
We use it in the following function which returns the final Revit product registry path:

```csharp
string RegPathForFlavour( ProductType flavour )
{
  return string.Format( \_reg\_path\_for\_flavour, flavour );
}
```

We also need methods to extract data from the registry.
In one case, we use the Revit product registry path to determine the product key, which is located in the value named ProductCode in the subkey named Components.
In the other case, we determine the Revit install location from the value named InstallLocation in the subkey specified by the product code in the Windows uninstall key.
Both of these values' data type is string.
Due to these two similar access paths, I can implement one single function to read the data:

```csharp
string GetSubkeyValue(
  string reg\_path\_key,
  string subkey\_name,
  string value\_name )
{
  using( RegistryKey key
    = Registry.LocalMachine.OpenSubKey( reg\_path\_key ) )
  {
    using( RegistryKey subkey
      = key.OpenSubKey( subkey\_name ) )
    {
      return subkey.GetValue( value\_name ) as string;
    }
  }
}
```

Defined like this, making use of the method for the two cases described above is extremely simple:

```csharp
string GetRevitProductCode( string reg\_path\_product )
{
  return GetSubkeyValue( reg\_path\_product,
    "Components", "ProductCode" );
}

string GetRevitInstallLocation( string product\_code )
{
  return GetSubkeyValue( \_reg\_path\_uninstall,
    product\_code, "InstallLocation" );
}
```

We define a little helper method to format the data obtained for presentation in the message box:

```csharp
string FormatData(
  string description,
  string version\_name,
  string product\_code,
  string install\_location )
{
  return string.Format
    ( "{0}: {1}"
    + "\nProduct code: {2}"
    + "\nInstall location: {3}",
    description,
    version\_name,
    product\_code,
    install\_location );
}
```

Finally we can put it all together into the mainline code of our external command's Execute function.
First, we look at the currently executing Revit application.
The application Product property provides the ProductType, which can be used to obtain the registry path for this flavour of Revit, determine its product code and from that the install location.
Next we loop through all three ProductType enumeration values and check for other installed flavours of Revit as well.
The final result is displayed in the message box shown above:

```csharp
Application app = commandData.Application;

string reg\_path\_product
  = RegPathForFlavour( app.Product );

string product\_code
  = GetRevitProductCode( reg\_path\_product );

string install\_location
  = GetRevitInstallLocation( product\_code );

string msg = FormatData(
  "Running application",
  app.VersionName,
  product\_code,
  install\_location );

foreach( ProductType p in
  Enum.GetValues( typeof( ProductType ) ) )
{
  try
  {
    reg\_path\_product = RegPathForFlavour( p );

    product\_code = GetRevitProductCode(
      reg\_path\_product );

    install\_location = GetRevitInstallLocation(
      product\_code );

    msg += FormatData(
      "\n\nInstalled product",
      p.ToString(),
      product\_code,
      install\_location );
  }
  catch( Exception ex )
  {
  }
}

Util.InfoMsg( msg );

return CmdResult.Failed;
```

Here is
[version 1.1.0.48](zip/bc11048.zip)
of the complete Building Coder sample source code and Visual Studio solution including the new command.

Many thanks to Jim for providing the original VB code snippets and the idea for this post!