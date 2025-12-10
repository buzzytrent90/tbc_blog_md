---
post_number: "0354"
title: "Add-In Manifest and Guidize"
slug: "guidizer"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'revit-api', 'views', 'windows']
source_file: "0354_guidizer.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0354_guidizer.html"
---

### Add-In Manifest and Guidize

Developers have always had a lot of complaints about the Revit.ini add-in registration facility.
It was hard to access and modify the file in the Revit installation directory and the same file is shared by all users.
Revit 2011 offers a completely new and vastly improved add-in registration facility using add-in manifest files.
The new mechanism supports a number of new options and there is no reason to continue using the obsolete Revit.ini mechanism.
Let's have a look at this new mechanism and also present a little tool that I developed to simplify one aspect in the creation of these files:

- [Add-In Manifest Files](#1)
- [Add-In Manifest Tags](#2)
- [Add-In Manifest Encoding](#3)
- [Add-In Client Id](#4)
- [Guidize](#5)

#### Add-In Manifest Files

An add-in manifest is an XML file located in specific location on the hard disk.
It can apply either to a specific user or all.
There are some required tags which must be specified for both an external application and an external command, such as Assembly, FullClassName, ClientId.
An external command also uses the Text and Description tags.
These required tags provide the same information as the entries used in the obsolete Revit.ini add-in registration mechanism.
However, the add-in manifest also offers a large number of exciting new functionality, and therefore many new optional tags can be used to define tooltip, image, visibility, availability, localisation and other features.

Here is an example of a minimal add-in manifest file for an external command:

```
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<RevitAddIn>
  <AddIn Type="Command">
    <Text>Convert Pipes to Conduits</Text>
    <Description>Convert Pipes to Conduits</Description>
    <Assembly>C:\src\p2c\p2c\bin\Debug\p2c.dll</Assembly>
    <FullClassName>p2c.Command</FullClassName>
    <ClientId>835d6ad1-1a99-4039-95dc-e752ff635928</ClientId>
  </AddIn>
</RevitAddIn>
```

Manifest files are read automatically by Revit when they are placed in one of the following two locations in the Application Data folder on a user's system.
There are separate locations for individual users or all users, and they also depend on whether you are running Windows XP or Vista/Windows 7:

- For a specific user:
  - For Windows XP - C:\Documents and Settings\\Application Data\Autodesk\Revit\Addins\2011- For Vista/Windows 7 - C:\Users\\AppData\Roaming\Autodesk\Revit\Addins\2011- For all users:
    - For Windows XP - C:\Documents and Settings\All Users\Application Data\Autodesk\Revit\Addins\2011- For Vista/Windows 7 - C:\ProgramData\Autodesk\Revit\Addins\2011

All files with an '.addin' filename extension found in these locations will be read and processed by Revit during start-up. Multiple add-ins may be loaded by a single manifest file. The manifest file syntax defines tags allowing you to specify all the information previously defined in the Revit.ini registration data, i.e. assembly path, full class name, text and description:

This significantly simplifies installation, since all you have to do is copy the add-in assembly to your hard disk, update the assembly path specified in the manifest file, and copy the manifest file to the desired location where Revit can pick it up without affecting any other existing functionality or touching any Revit files or other parts of the Revit installation.

So to install an add-in, you simply copy the manifest file to the appropriate application data location on the hard disk.
An installer may obviously need to update the assembly path.
The Revit SDK includes the
[add-in utility DLL](http://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html) which
can be used to edit or automatically generate the manifest file on the fly.
It supports reading, writing and manipulation of manifest files and also querying the Revit product installation locations.

One little thing to note is that for a serious application, due to .NET framework restrictions and issues with Load versus LoadFrom, it is recommended to place the add-in into the Revit Program folder in the same location as Revit.exe.

The Revit.ini registration mechanism remains in place for the 2011 release but will be removed in the future.
It does not offer any of the new capabilities listed below.

#### Add-In Manifest Tags

The basic and obligatory add-in manifest tags include:

- **Assembly:**
  The full path to the add-in assembly file. Required for all ExternalCommands and ExternalApplications.- **FullClassName:**
    The full name of the class in the assembly file which implements IExternalCommand or IExternalApplication. Required for all ExternalCommands and ExternalApplications.- **ClientId:**
      A GUID which represents the id of this particular application. ClientIds must be unique for a given session of Revit. Autodesk recommends you generate a unique GUID for each registered application or command. Required for all ExternalCommands and ExternalApplications. The property UIApplication.ActiveAddInId provides programmatic access to this value, if required.- **Name:**
        The name of application. Required; for ExternalApplications only.- **Text:**
          The name of the button. Optional; use this tag for ExternalCommands only. The default is "External Tool".- **Description:**
            Short description of the command, will be used as the button tooltip. Optional; use this tag for ExternalCommands only. The default is a tooltip with just the command text.

The add-in manifest mechanism also supports a long list of additional new functionality.
Here is a complete list of the remaining XML tags that can be used in an add-in manifest file:

- **VisibilityMode:**
  Provides the ability to specify if the command is visible in project documents, family documents, or no document at all. Also provides the ability to specify the discipline(s) where the command should be visible. Multiple values may be set for this option. Optional; use this tag for ExternalCommands only. The default is to display the command in all modes and disciplines, including when there is no active document. Previously written external commands which need to run against the active document should either be modified to ensure that the code deals with invocation of the command when there is no active document, or apply the NotVisibleWhenNoActiveDocument mode.- **AvailabilityClassName:**
    The full name of the class in the assembly file which implemented IExternalCommandAvailability. This class allows the command button to be selectively greyed out depending on context. Optional; use this tag for ExternalCommands only. The default is a command that is available whenever it is visible.- **LargeImage:**
      The path to the icon to use for the button in the External Tools pull-down menu. The icon should be 32 x 32 pixels for best results. Optional; use this tag for ExternalCommands only. The default is to show a button without an icon.- **LongDescription:**
        Long description of the command, will be used as part of the button's extended tooltip.
        This tooltip is shown when the mouse hovers over the command for a long amount of time.
        You can split the text of this option into multiple paragraphs by placing <p> tags around each paragraph.
        Optional; use this tag for ExternalCommands only.
        If neither of this property and TooltipImage are supplied, the button will not have an extended tooltip.- **TooltipImage:**
          The path to an image file to show as a part of the button extended tooltip, shown when the mouse hovers over the command for a longer amount of time. Optional; use this tag for ExternalCommands only. If neither of this property and TooltipImage are supplied, the button will not have an extended tooltip.- **LanguageType:**
            Localization setting for Text, Description, LargeImage, LongDescription, and TooltipImage of external tools buttons. Revit will load the resource values from the specified language resource DLL. The value can be one of the eleven languages supported by Revit. If no LanguageType is specified, the language resource which the current session of Revit is using will be automatically loaded. For more details see the section on Localization.

As you can see, there is a large amount of very useful new functionality buried in here!
For full details, especially on localisation, external command accessibility and the IExternalCommandAvailability interface, please refer to the What's New section in the Revit API documentation.

#### Add-In Manifest Encoding

One detail that may require some attention is the encoding of the add-in manifest file.
Note that the xml tag also specifies the encoding, for example utf-8, utf-16, and others.
You need to ensure that the file really is stored in a format matching that attribute.

A common problem during beta testing was to define the encoding="utf-16" in the xml tag, but actually save the manifest file as ASCII. As said, the file encoding needs to correspond to the defined encoding,
so one would either have to change the encoding to "ASCII" or save the add-in file as UTF-16.

The simplest possible editor to use is probably Notepad, where you can use File > Save As... > Encoding > ANSI or Unicode.

In Visual Studio 2008, you can use File > Save As... > Drop down list button > Save with Encoding... > Yes > Encoding: > Choose the encoding > Line endings: Current Setting > OK.

Here is an overview of some of the possible encoding settings:

- "us-ascii"
  - Notepad: ANSI- Notepad++: Encode in ANSI- VS Save As: US-ASCII - Codepage 20127- VS Properties > Encoding: US-ASCII- "utf-8"
    - Notepad: UTF-8- Notepad++: Encode in UTF-8- VS Save As: Unicode (UTF-8 with signature) - Codepage 65001- VS Properties > Encoding: Unicode (UTF-8)- "utf-16"
      - Notepad: Unicode- Notepad++: Encode in UCS-2 Little Endian- VS Save As: Unicode - Codepage 1200- VS Properties > Encoding: Unicode- "unicodeFFFE"
        - Notepad: Unicode big endian- Notepad++: Encode in UCS-2 Big Endian- VS Save As: Unicode (Big-Endian) - Codepage 1201- VS Properties > Encoding: Unicode (Big-Endian)

#### Add-In Client Id

As you saw in the example add-in manifest presented above, the basic required information is completely analogous to the entries used in Revit.ini, except for the new client id.

As described in the overview of the manifest tags, it is a
[GUID](http://en.wikipedia.org/wiki/Guid) which
represents the id of this particular application.
We discussed GUIDs in the context of the element identifiers generated for the
[DWF and IFC export](http://thebuildingcoder.typepad.com/blog/2009/02/uniqueid-dwf-and-ifc-guid.html).

You can query an application's client id programmatically through the API property UIApplication.ActiveAddInId.

The client id must be unique, and every single external command requires its own id.
You cannot reuse the same client id for several commands, even if they are defined and implemented by the same application.

There are several different ways to generate a unique GUID for any purpose.
One is provided with Visual Studio, guidgen.exe:

![Guidgen](img/guidgen.png)

On my system, this utility is located in the folder C:\Program Files\Microsoft Visual Studio 9.0\Common7\Tools.

A new GUID can also be generated by the .NET framework API call System.Guid.NewGuid(), which I make use of in the utility presented below.

#### Guidize

I want to present a little tool that I developed to automatically populate the client id tag in the new add-in manifest file.

As we discussed above, the new command registration mechanism requires every Revit plug-in to define its own unique client id in the add-in manifest file.
While porting the Revit Introduction Labs and implementing add-in manifest files for all its commands, I got tired of manually generating GUIDs with gudgen.exe and editing them into the manifest files.
I implemented a little tool that enabled me to write the manifest files without client id entries, and then let the tool populate them automatically afterwards.
That saved me from manually running and copy and pasting dozens of client ids.

So, in case anybody is tired of manually generating GUIDs for Revit commands, I proudly present guidize, a Revit Add-In Manifest File Client Id GUID Generator.

It reads a Revit add-in manifest file, searches for all the ClientId tags, and populates them with newly generated GUIDs.
By default it only populates empty ones, so that existing GUIDs are not overwritten.
However, with the -o switch, you can also overwrite existing ones.
The option -b creates a backup file, and the -v verbose mode prints out some info on what the tool is doing:

```
C:\a\j\adn\case\bsd\1256939\ >guidize -?
guidize 1.0 -- populate Revit add-in manifest file ClientId tags
Copyright (C) 2010 by Jeremy Tammik, Autodesk Inc. All rights reserved.

usage: guidize [options] manifest_file
  -? or -h  help
  -b        backup previous version of the manifest file
  -o        overwrite existing ClientId entries
  -v        verbose
```

After manually creating and editing dozens of GUIDs and with the prospect of requiring many more, I found this little tool to be an extremely comfortable addition to my kit.

Here is the complete
[source code and Visual Studio solution file](guidize_src.zip),
and also the ready-built
[executable](guidize_exe.zip) for
non-programmers.