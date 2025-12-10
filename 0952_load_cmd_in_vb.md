---
post_number: "0952"
title: "Load Your Own External Command on the Fly"
slug: "load_cmd_in_vb"
author: "Jeremy Tammik"
tags: ['elements', 'python', 'revit-api']
source_file: "0952_load_cmd_in_vb.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0952_load_cmd_in_vb.html"
---

﻿

### Load Your Own External Command on the Fly

How to reload an external Revit command without having to restart the Revit session has been a subject of discussion many times in the past, e.g. in
[reloading an add-in for debug without restart](http://thebuildingcoder.typepad.com/blog/2012/12/reload-add-in-for-debug-without-restart.html),
which lists four possibilities:

- Set up Visual Studio to efficiently load the desired project and the debugee, use the standard cycle hitting F5 to start debugging, and stop debugging when you need to make code changes.
- Use the
  [RevitRubyShell](https://github.com/hakonhc/RevitRubyShell) or
  [RevitPythonShell](http://code.google.com/p/revitpythonshell) to
  work interactively with the Revit API on the command line.
- Use the SharpDevelop IDE, temporarily converting the debugee source code to a macro.
- Use the AddInManager.

Revit 2014 adds some new twists to the scenario by the introduction of the MacroManager API, which enables programmatic listing, creating, removing, editing, debugging, and running of macros, and the option to load an add-in mid-session without restarting Revit.
Unfortunately, there is no option to unload an add-in, once loaded.

The difficulty is that the AppDomain into which the add-in is loaded locks the add-in DLL, preventing any further changes to it.

One could create a new individual AppDomain for each add-in to work around that.

Another option is to not load the add-in from a DLL at all, but stream in the source code using reflection instead.

Here is an example of the latter approach in VB by
David Rock and Yamin Tengono of
[BSE – Building Services Engineers](http://www.bse.com.au).
David says:

I am running my commands from the ribbon with the following function:

```vbnet
Public Shared Sub InvokeRevitCommand( \_
  ByVal strCommandName As String,
  ByVal commandData As Object,
  ByRef message As String,
  ByVal elements As Object,
  ByVal fullPathDllName As String)

  ' Load the assembly into a byte array.
  ' This way it WON'T lock the dll to disk.

  Dim assemblyBytes As Byte() = File.ReadAllBytes(
    fullPathDllName)

  Dim objAssembly As Assembly = Assembly.Load(
    assemblyBytes)

  ' Walk through each type in the assembly.

  For Each objType As Type In objAssembly.GetTypes()
    ' Pick up a class.
    If objType.IsClass Then
      If objType.Name.ToLower = strCommandName.ToLower Then

        Dim ibaseObject As Object = Activator.CreateInstance(objType)

        Dim arguments As Object() = New Object() {
          commandData, message, elements}

        Dim result As Object

        result = objType.InvokeMember(
          "Execute",
          BindingFlags.[Default] Or BindingFlags.InvokeMethod,
          Nothing, ibaseObject, arguments)

        Exit For
      End If
    End If
  Next
End Sub
```

This way I can re-build the DLL without exiting Revit and my sometimes quite large Revit projects.

I have over 100 Revit commands now and most are loaded via this dynamic method.
This makes it super easy to update them on the fly.

Very many thanks to David and Yamin for developing and sharing this effective solution!