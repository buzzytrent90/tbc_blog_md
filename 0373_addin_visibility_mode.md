---
post_number: "0373"
title: "Add-In Visibility Mode"
slug: "addin_visibility_mode"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'family', 'revit-api', 'schedules', 'views', 'windows']
source_file: "0373_addin_visibility_mode.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0373_addin_visibility_mode.html"
---

### Add-In Visibility Mode

This Monday is the
[Pentecost holiday](http://en.wikipedia.org/wiki/Pentecost)
in Neuchâtel.
I used it to make my first experiments using the add-in manifest visibility mode tag.
The developer guide has the following to say about it:

Provides the ability to specify if the command is visible in project documents, family documents, or no document at all.

Also provides the ability to specify the discipline(s) where the command should be visible.
Multiple values may be set for this option.
Optional; use this tag for ExternalCommands only.
The default is to display the command in all modes and disciplines, including when there is no active document. Previously written external commands which need to run against the active document should either be modified to ensure that the code deals with invocation of the command when there is no active document, or apply the NotVisibleWhenNoActiveDocument mode.

It does not actually specify how to define multiple values for this tag, though.
Luckily, the Code Region 3-9: 'Manifest .addin ExternalCommand' also provides an example of using it:
```csharp
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<RevitAddIns>
  <AddIn Type="Command">
    <Assembly>c:\MyProgram\MyProgram.dll</Assembly>
    <AddInId>76eb700a-2c85-4888-a78d-31429ecae9ed</AddInId>
    <FullClassName>Revit.Samples.SampleCommand</FullClassName>
    <Text>Sample command</Text>
    <VisibilityMode>NotVisibleInFamily</VisibilityMode>
    <VisibilityMode>NotVisibleInMEP</VisibilityMode>
    <AvailabilityClassName>Revit.Samples.SampleAccessibilityCheck</AvailabilityClassName>
    <LongDescription>
      <p>This is the long description for my command.</p>
      <p>This is another descriptive paragraph, with notes about how to use the command properly.</p>
    </LongDescription>
    <TooltipImage>c:\MyProgram\Autodesk.jpg</TooltipImage>
    <LargeImage>c:\MyProgram\MyProgramIcon.png</LargeImage>
  </AddIn>
</RevitAddIns>
```

If the lines are truncated, please copy them to an editor to see them in full.

#### VisibilityMode Enumeration

The list of all the possible values to use in this tag is currently not provided explicit in the API documentation, but we will be adding that information anon.
The values to use are simply the string representations of the VisibilityMode enumeration values, some of which are used in Code Region 3-11: 'Creating and editing a manifest file' in the developer guide:
```csharp
  // this command only visible in Revit MEP,
  // Structure, and only visible in Project
  // document or when no document at all:
  command1.VisibilityMode
    = VisibilityMode.NotVisibleInArchitecture
    | VisibilityMode.NotVisibleInFamily;
```

The VisibilityMode enumeration is documented in the
[RevitAddInUtility](http://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html) help file
RevitAddInUtility.chm located in the SDK folder, so the full list can be found there:

Describes the conditions under which a particular external command will be visible in the Revit UI.

##### Members

- AlwaysVisible: The command is available in all possible modes supported by the Revit API.- NotVisibleInProject: The command is invisible when there is a project document active.- NotVisibleInFamily: The command is invisible when there is a family document active.- NotVisibleWhenNoActiveDocument: The command is invisible when there is no active document.- NotVisibleInArchitecture: The command is invisible in Autodesk Revit Architecture.- NotVisibleInStructure: The command is invisible in Autodesk Revit Structure.- NotVisibleInMechanical: The command is invisible when the Mechanical discipline editing tools are available, e.g. in Autodesk Revit MEP.- NotVisibleInElectrical: The command is invisible when the Electrical discipline editing tools are available, e.g. in Autodesk Revit MEP.- NotVisibleInPlumbing: The command is invisible when the Plumbing discipline editing tools are available, e.g. in Autodesk Revit MEP.- NotVisibleInMEP: The command is invisible in Autodesk Revit MEP.

##### Remarks

Note that there are a few conditions where the Revit API framework prevents commands from being available always:

1. When the user has another Revit command active, e.g. creating Windows, Doors, or editing sketches.- When the active view is in perspective mode, or when the view is a Legend, Schedule, Walkthrough, Material Takeoffs, Drawings Lists, View Lists, Note Blocks, View Lists, etc.- When the user is editing an in-place family.

#### Loose Connector Utility

Now I implemented a little utility to check for unconnected MEP connectors.
Since my list of add-ins is continuously growing, I thought it might be about time to start hiding as many as possible of them when they are not needed.
In this case, I only want access to this utility in the Revit MEP project context, i.e. I wanted to hide it in all other modes:

- Architecture- Family editor- Structure- No active document

Using the couple of values mentioned in the developer guide and simply guessing what the missing ones might be, I ended up with the following manifest file:
```csharp
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<RevitAddIn>
  <AddIn Type="Command">
    <Text>Loose Connectors</Text>
    <Description>List all unconnected Revit MEP connectors</Description>
    <VisibilityMode>NotVisibleInArchitecture</VisibilityMode>
    <VisibilityMode>NotVisibleInFamily</VisibilityMode>
    <VisibilityMode>NotVisibleInStructure</VisibilityMode>
    <VisibilityMode>NotVisibleWhenNoActiveDocument</VisibilityMode>
    <Assembly>C:\bin\LooseConnectors.dll</Assembly>
    <FullClassName>LooseConnectors.Command</FullClassName>
    <ClientId>fc508f72-84b2-4e82-942c-ad9ec07cbe4a</ClientId>
  </AddIn>
</RevitAddIn>
```

I tested it in the contexts mentioned above, and lo and behold, my external command is not displayed, just as desired.