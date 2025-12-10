---
post_number: "0137"
title: "VB Samples and Other Questions"
slug: "vb_samples"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'vbnet', 'views', 'windows']
source_file: "0137_vb_samples.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0137_vb_samples.html"
---

### VB Samples and Other Questions

I have been very quiet now for a few days.
The reason is that I went on a climbing weekend to Mallorca with my brother and left my laptop in the car while we went for a climb.
The car was broken into, and the laptop and all my valuables were stolen, right down to the bicycle keys.
Since I had actually used the bicycle to get to the airport, the first thing I had to do on arriving back was to borrow a saw to break my own bicycle lock.
Now I am in the painful process of restoring data to a temporary computer.
A valuable lesson in letting go, I suppose. More on this below.

Anyway, here are some queries on different topics, some of which have been touched upon previously, and an updated reiteration may be of general interest:

1. [Programmatically triggering IFC export.](#1)
2. [ImportExport SDK sample in VB.](#2)
3. [OnStartup sample code in VB.](#3)
4. [64 bit OLE DB access.](#4)

**Question:**
How can I trigger a daily export of a Revit model to an IFC file?

**Answer:**
I am sorry to say that there is currently no API access to this functionality.
One option that you might explore is to simulate the user input required for this repetitive task programmatically.
You might do this directly using the Win32 API, or make use of some utilities that provide easier access to this functionality.
Here are some of the previous posts dealing with related topics:

- [Driving Revit from outside.](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html)
- [The Revit Window handle and modeless dialogues.](http://thebuildingcoder.typepad.com/blog/2009/02/revit-window-handle-and-modeless-dialogues.html)
- [AutoHotKey.](http://thebuildingcoder.typepad.com/blog/2009/01/autohotkey.html)

Another option that may fit your requirements perfectly but is not officially supported is the use of the Revit journal file.
Every action performed through the Revit user interface is recorded in a journal file, located in a subfolder of the Revit installation folder.
We once discussed
[how to obtain the path of the current journal file](http://thebuildingcoder.typepad.com/blog/2009/02/getting-the-journal-file-path.html).
Journal files can be rerun to reproduce the actions performed in a previous session, for instance by specifying them on the Revit command line during start-up.
Custom applications can define what data their commands register in the journal file, and how to process that data when a journal file is replayed, as demonstrated by the Revit SDK Journaling sample.
In your case, however, you would possibly not need to do any programming or customisation at all, just start up Revit, perform the IFC export, close down Revit again, and save the resulting journal file.
This file might be reusable to automate the IFC export on a daily basis, just as you require.

**Question:**
The Revit SDK 2010 provides a sample application named ImportExport which is of interest to me.
Unfortunately, it is in C#, I am only familiar with VB, and the example is quite comprehensive.
How can I convert this to VB, please?

**Answer:**
We already discussed the topics of conversion, disassembly, decompilation, Lutz Roeder's
[Reflector](http://thebuildingcoder.typepad.com/blog/2008/10/converting-between-vb-and-c-and-net-decompilation.html)
tool and the related topic of
[obfuscation](http://thebuildingcoder.typepad.com/blog/2008/10/converting-between-vb-and-c-and-net-decompilation.html)
in previous posts.
This tool can be used perfectly well to decompile the Revit SDK Samples, as well.
The decompiled code can be displayed in a variety of different languages, including Delphi, managed C++, Chrome and Visual Basic.
As said, the ImportExport sample is written in C#.
I opened the compiled assembly ImportExport.dll in Reflector and asked to see the source code of the Command class disassembled into Visual Basic.
Here is the result:

```vbnet
Public Class Command
Implements IExternalCommand
' Methods
Public Function Execute( \_
ByVal commandData As ExternalCommandData, \_
ByRef message As String, \_
ByVal elements As ElementSet \_
) As Result
Try
If (Nothing Is commandData.Application.ActiveDocument) Then
message = "Active view is null."
Return Result.Failed
End If
Dim mainData As New MainData(commandData)
Using mainForm As MainForm = New MainForm(mainData)
If (mainForm.ShowDialog = DialogResult.Cancel) Then
Return Result.Cancelled
End If
End Using
Catch ex As Exception
message = ex.ToString
Return Result.Failed
End Try
Return Result.Succeeded
End Function
End Class
```

I will leave it up to you to decompile the rest of the classes, if you wish.
Alternatively, there are other tools available which decompile an entire application in one go.

Nowadays, Reflector is available from [Red Gate](http://www.red-gate.com).

**Question:**
I would like my application to run automatically every time a file is opened.
I assume this can be achieved using the OnStartup method.
How is OnStartup used in VB.NET?

**Answer:**
We already looked at this topic once previously, and provided a short description on registering to
[application events in VB](http://thebuildingcoder.typepad.com/blog/2008/10/application-events-in-vb.html).
I repeated the steps described there in the Visual Studio 2008 environment for Revit Architecture 2010,
and implemented a simple sample external application named OnStartupVb and implementing OnStartup in Visual Basic for you.
The description provided in the previous post for Revit 2009 remains valid for 2010 as well.
Here is the resulting source code of the external application class:

```vbnet
Imports Autodesk.Revit
Public Class App
Implements IExternalApplication
Public Function OnShutdown(ByVal a As ControlledApplication) \_
As IExternalApplication.Result \_
Implements IExternalApplication.OnShutdown
Return IExternalApplication.Result.Succeeded
End Function
Public Function OnStartup(ByVal a As ControlledApplication) \_
As IExternalApplication.Result \_
Implements IExternalApplication.OnStartup
System.Windows.Forms.MessageBox.Show( \_
"OnStartup says Hello from Jeremy", \_
"OnStartup VB")
End Function
End Class
```

Here is a copy of the full Visual Studio solution [OnStartupVb](zip/OnStartupVb.zip).

**Question:**
No OLEDB provider registered ...
How can I retrieve a record set from an Access 2007 database using VB.NET in 64-bit Vista?

**Answer:**
If you install 32-bit Revit 2009 in 64-bit Vista, then this is possible.
This requires compiling your Revit plug-in in a 32-bit OS, then loading it into 32-bit Revit in the 64-bit OS.
On the other hand, if the Revit version is 64-bit, one possible approach is to use the SQL server database accessing driver.
Note that the 32-bit version of Revit 2010 cannot be installed on a 64-bit OS, so in that case using SQL Server or SQL Server Express would be the recommended option.

Thanks to Joe Ye for this answer!

Back to the topic of Mallorca.
Apart from criminality, it is a surprisingly beautiful place with large areas of untouched nature, mountains, and almost inaccessible beaches, in spite of the strong touristic focus in the cities and villages.
Also a huge amount of beautiful climbing opportunities.
Here I am walking along a path heading for both a lonesome beach and a challenging cliff:

![Lonesome path on Mallorca](img/1_jeremy_on_a_path.jpg)

The weather was hot enough to require some protection against the sun:

![Jeremy as a colourful Tuareg](img/2_jeremy_tuareg.jpg)