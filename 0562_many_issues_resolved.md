---
post_number: "0562"
title: "Many Issues Resolved"
slug: "many_issues_resolved"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'references', 'revit-api', 'views', 'walls']
source_file: "0562_many_issues_resolved.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0562_many_issues_resolved.html"
---

### Many Issues Resolved

I mentioned the main new features of the upcoming
[Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/03/revit-2012-api-features.html) last week.
Many of the improvements were directly driven by requests from developers, and many of the most common wish list items have been addressed.
Here are a couple of examples of this:

- Full read and write access to wall and floor layer [compound structure](#1264438) and [wall reveals](#1267865).- Read and write access to [wall join](#1267843) functionality.- Access to the [paint material for walls](#1267866).- Access to [linked file](#1267916) information.

**Question:** Is it possible in Revit to create a wall style and add or delete layers within the wall style?

If I look at the API functions I cannot find an option to add or delete a material from or to a wall style.
Nor do I see any possibility to create a new CompoundStructureLayer (constructor) to add it to a CompoundStructureLayerArray.
The CompoundStructureLayerArray function has only functions to add or insert a CompoundStructureLayer to the CompoundStructureLayerArray, but no function to delete or move an array member.
Is there another solution to solve this problem?

**Answer:** In the Revit 2011 API, you cannot create new layers in a wall type's compound layer structure.
That, however, has been enhanced in Revit 2012, which will allow you to freely create and manipulate the compound layer structure as you see fit.

I presented a work-around for Revit 2011 in the discussion on
[system family creation](http://thebuildingcoder.typepad.com/blog/2011/02/system-family-creation.html):
create the required number of layers manually and save the wall types in the template file.
Then, when your application needs a new wall type with a specific number of layers, it can use a suitable one of the manually created predefined wall types by duplicating and modifying that.

As said, the Revit 2012 now does offer this capability.
This is one of the very first new features mentioned in the pre-release API documentation:

#### CompoundStructure and WallSweeps

The CompoundStructure class has been replaced. The old CompoundStructure class in the API was read-only and supported only the nominal layers list without support for the settings related to vertical regions, sweeps, and reveals. The new CompoundStructure class supports read and modification of all of the structure information include vertical regions, sweeps and reveals.

The CompoundStructure.Layers property has been replaced by CompoundStructure.GetLayers() and CompoundStructure.SetLayers().

The CompoundStructureLayer class has also been replaced for the same reasons as CompoundStructure.

The following table maps properties from the old CompoundStructureLayer to the new version of the class:

| Old property | New property | Notes |
| --- | --- | --- |
| DeckProfile | DeckProfileId | Is now an ElementId |
| DeckUsage | DeckEmbeddingType | Uses a different enum |
| Function | Function | Uses a different enum |
| Material | MaterialId | Is now an ElementId |
| Thickness | Width |
| Variable | N/A | Not part of the layer class, use CompoundStructure.VariableLayerIndex instead. |

The property HostObjAttributes.CompoundStructure has been replaced by two methods:

- HostObjAttributes.GetCompoundStructure()- HostObjAttributes.SetCompoundStructure()

Remember that you must set the CompoundStructure back to the HostObjAttributes instance in order for any change to be stored in the element.

In addition to the information on wall sweeps found in the CompoundStructure class, there is a new API representing a wall sweep or reveal. The WallSweep class is an element that represents either a standalone wall sweep/reveal, or one added by the settings in a given wall's compound structure. Standalone wall sweeps and reveals may be constructed using the static method Create().

**Question:** How can I retrieve information such as object id, geometry, parameters, etc. for wall reveals?
I know how to obtain this data for wall sweeps, but not reveals.
Please advise.

**Answer:** As indicated by the last paragraph quoted above, the Revit 2012 API provides full access to wall sweeps and reveals.
It provides the WallSweep and WallSweepInfo classes which cover the properties of these elements.

There are two kinds of sweeps and reveals: those defined in the wall structure itself, and those placed externally and attached to the wall.
In the former case, the WallSweepInfo can be obtained directly from the CompoundStructure.
In the latter, the WallSweep element subtype can be found, and related to the wall by querying its GetHostIds property.

**Question:** How can I programmatically access the "Disallow Join" option for Wall objects?

**Answer:** The ability to disallow wall joins has been requested in the past by other partners, and the Revit 2012 API now provides it:

#### Allow and disallow wall end joins

The new methods

- WallUtils.DisallowWallJoinAtEnd()- WallUtils.AllowWallJoinAtEnd()- WallUtils.IsWallJoinAllowedAtEnd()

provide access to the setting for whether or not joining is allowed at a particular end of the wall. If joins exist at the end of a wall and joins are disallowed, the walls will become disjoint. If joins are disallowed for the end of the wall, and then the setting is toggled to allow the joins, the wall will automatically join to its neighbours if there are any.

**Question:** I can use the Revit Paint tool to define the material of part of a wall surface.
How would I retrieve the paint material information including its geometry?

**Answer:** The Revit 2011 API does not provide access to the split faces on a wall or their paint information.
The Revit 2012 API, however, provides the ability to get the regions of each face of solids, e.g. walls, and each region will provide material information too. The Revit 2012 API help file RevitAPI.chm states the following:

#### Face.HasRegions & Face.GetRegions()

This property and method provide information about the faces created by the Split Face command. HasRegions returns a Boolean value indicating if the face has any Split Face regions. GetRegions returns a list of faces. As the material of these faces can be independently modified through the UI with the Paint tool, the material of each face can be found from its MaterialElementId property.

With this new API in Revit 2012, you should be able to retrieve the different paint materials on split surfaces of a wall.

**Question:** I would like to use the Link Revit file functionality from the API, but I cannot find any solution on how to execute such an operation from the API.

I would also be interested in moving such a linked document.
For this I need to know exactly where it is, but the Location property value is not of LocationPoint type.

**Answer:** One focus of the Revit 2012 API is the ability to obtain the information about linked files.
It provides the following new features related to Linked Models:

#### External File References (Linked Files)

The API can now tell what elements in Revit are references to external files, and can make some modifications to where Revit loads external files from.

An Element which contains an ExternalFileReference is an element which refers to some external file (ie. a file other than the main .rvt file of the project.) Two new Element methods, IsExternalFileReference() and GetExternalFileReference(), let you get the ExternalFileReference for a given Element.

ExternalFileReference contains methods for getting the path of the external file, the type of the external file, and whether the file was loaded, unloaded, not found, etc. the last time the main .rvt file was opened.

The classes RevitLinkType and CADLinkType can have IsExternalFileReference() return true. RevitLinkTypes refer to Revit files linked into another Revit project. CADLinkTypes refer to DWG files. Note that CADLinkTypes can also refer to DWG imports, which are not external file references, as imports are brought completely into Revit. A property IsImport exists to let users distinguish between these two types.

Additionally, the element which contains the location of the keynote file is an external file reference, although it has not been exposed as a separate class.

There is also a class ExternalFileUtils, which provides a method for getting all Elements in a document which are references to external files.

Additionally, the classes ModelPath and ModelPathUtils have been exposed. ModelPaths can store paths to a location on disk, a network drive, or a Revit Server location. ModelPathUtils provides methods for converting between modelPath and String.

Finally, the class TransmissionData allows users to examine the external file data in a closed Revit document, and to modify path and load-state information for that data. Two methods, ReadTransmissionData, and WriteTransmissionData, are provided. With WriteTransmissionData, users can change the path from which to load a given link, and can change links from loaded to unloaded and vice versa. (WriteTransmissionData cannot be used to add links to or remove links from a document, however.)

Newly exposed classes:

- ExternalFileReference - A non-Element class which contains path and type information for a single external file which a Revit project references. ExternalFileReference also contains information about whether the external file was loaded or unloaded the last time the associated Revit project was opened.- ExternalFileUtils - A utility class which allows the user to find all external file references, get the external file reference from an element, or tell whether an element is an external file reference.- RevitLinkType - An element representing a Revit file linked into a Revit project.- CADLinkType - An element representing a DWG drawing. CADLinkTypes may be links, which maintain a relationship with the file they originally came from, or imports, which do not maintain a relationship. The property IsImport will distinguish between the two kinds.- LinkType - The base class of RevitLinkType and CADLinkType.- ModelPath - A non-Element class which contains path information for a file (not necessarily a .rvt file.) Paths can be to a location on a local or network drive, or to a Revit Server location.- ModelPathUtils - A utility class which provides methods for converting between strings and ModelPaths.- TransmissionData - A class which stores information about all of the external file references in a document. The TransmissionData for a Revit project can be read without opening the document.

As you saw in the
[Revit 2012 API overview](http://thebuildingcoder.typepad.com/blog/2011/03/revit-2012-api-features.html),
these items form just a small subset of all the issues addressed, so we can continue looking forward to ever greater possibilities and more powerful applications using the Revit API.