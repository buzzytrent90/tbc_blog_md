---
post_number: "0165"
title: "Prompt the User for Pinning and IFC Export"
slug: "prompt_user"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'revit-api']
source_file: "0165_prompt_user.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0165_prompt_user.html"
---

### Prompt the User for Pinning and IFC Export

We recently touched on the topic of
[IFC export](http://thebuildingcoder.typepad.com/blog/2009/05/vb-samples-and-other-questions.html#1)
and the lack of API access to this functionality.
Another property that the Revit API also does not provide write access to is the Element.Pinned property.
In both of these cases, however, write access is provided through the user interface, and it is possible to programmatically read the current state.
Therefore, an application can determine the current state and prompt the user to execute the required action if necessary.
While it would be nicer to be able to implement a fully automated process, this approach is fool-proof and safe.

### Prompt the User to Pin a Link

Jim Bish of
[Inlet Technology](http://www.inlettechnology.com)
recently suggested an effective, simple and reliable work-around for the lack of write access to the Element.Pinned property.
His requirement is especially to pin unpinned Revit link instances.
One solution is to simply
[prompt the user to pin them](http://thebuildingcoder.typepad.com/blog/2009/06/convex-hull-and-volume-computation.html?cid=6a00e553e1689788330115718ed107970b#comment-6a00e553e1689788330115718ed107970b):
```vbnet
Public Function Execute( \_
  ByVal commandData As ExternalCommandData, \_
  ByRef message As String, \_
  ByVal elements As ElementSet) \_
As IExternalCommand.Result \_
Implements IExternalCommand.Execute
  Dim app As Application = commandData.Application
  Dim doc As Document = app.ActiveDocument
  Dim element As Element = Nothing
  Dim f As Filter
  f = app.Create.Filter.NewCategoryFilter( \_
    BuiltInCategory.OST\_RvtLinks)
  Dim result As New List(Of Element)
  Dim n As Integer = doc.Elements(f, result)
  For Each element In result
    If TypeOf (element) Is Instance Then
      If Not element.Pinned Then
        MsgBox("Hello! " \_
          & element.ObjectType.Name \_
          & " is not pinned! Please pin it.")
      End If
    End If
  Next
End Function
```

Some features that might be worth adding to Jim's quick solution:
Some kind of logging functionality, so that you are not forced to acknowledge the message box individually for each non-pinned link.
It might also be helpful to add some more information to the message, such as the link's element id, in a widget allowing you to copy it to the clipboard.
This will enable you to select it in the user interface, for instance in order to zoom in to it.

### Prompt the User to Export to IFC

Another solution prompting the user to execute a command due to the lack of programmatic access was proposed by Richard Brand of
[Itannex](http://www.itannex.com),
to handle the lack of an API to drive the IFC export functionality:

My tool needs to generate a ZIP file containing the Revit file and the IFC export.
This ZIP file should be created when the user selects a menu item in my application.
That is why I cannot produce the IFC files through a nightly batch process driving Revit through a journal file.
I 'solved' it by checking if there is an IFC file present. If not, or if it is not recent, I ask the user to export the model to IFC by hand into a particular directory.
As long as the Revit API does not provide access to the IFC export I guess we have to solve this issue this way.
It should be much more user friendly if the IFC export would be generated automatically.