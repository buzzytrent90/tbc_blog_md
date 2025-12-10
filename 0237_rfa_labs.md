---
post_number: "0237"
title: "Revit Family Creation API Labs"
slug: "rfa_labs"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'levels', 'parameters', 'references', 'revit-api', 'schedules', 'views']
source_file: "0237_rfa_labs.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0237_rfa_labs.html"
---

### Revit Family Creation API Labs

The handouts and presentation for my
[Autodesk University](http://au.autodesk.com)
[family API session](https://au.autodesk.com/?nd=class&session_id=5265&jid=323682)
are finally done, as well they should be, since they are due to be submitted by today.
The last part I worked on was to add more documentation on the family API labs to the handout document.
I think this overview will be of interest here as well.
The family API labs were originally created and documented by Mikako Harada, the ADN AEC workgroup lead, so all of the material presented here is due to her.
Talking about that prompts me to mention the webcasts where this material was first used:

#### ADN webcast registration statistics

Most of the material that I am presenting at AU was originally prepared for webcasts on the Revit
[Family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html) and
[MEP API](http://thebuildingcoder.typepad.com/blog/2009/09/the-revit-mep-api.html) and then posted to this blog as well.
Mikako just sent around some statistics on the numbers of registrations for the various webcasts held by the Autodesk Developer Network this year.
The overview of all webcasts and other training classes is available online from the Autodesk Developer Network
[API training schedule](http://www.adskconsulting.com/adn/cs/api_course_sched.php).
All in all, we presented on 25 different topics and had a total of 1576 attendees registered.
The absolute leader in the number of registrations was the Revit API webcast with 250, followed by the Inventor API with 100.
The Revit family and MEP API webcasts were also pretty popular with 75 and 64 respectively, so the three Revit webcasts together attracted 389 out of 1576 registrations or about 25%.
This just goes to show that the Revit API is an important and popular topic.

#### Family API Labs: Creating an Example Family

Returning to the Revit Family API Labs, they are a collection of exercises which introduce you step by step to the creation of a column family.
The objective is to learn the basics of the
[family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html).
The labs start at zero and proceed through the absolute beginning steps up to slightly more advanced aspects.
Here are the four steps of increasing complexity covered:

1. [Define a rectangular profile column family](#1).
2. [Define an L-shaped profile column family](#2).
3. [Add formulae and materials](#3).
4. [Add visibility control](#4).

The full Visual Studio solution and projects, source code, detailed documentation and instructions for each step are included separately for both C# and VB.
Here is an overview of the main files and directories:

- rfa\_labs.sln – common C# and VB Visual Studio solution file- cs – C# source code directory
    - LabsCs.csproj- 1\_ColumnRectangle.cs- 2\_ColumnLshape.cs- 3\_ColumnFormulaMaterial.cs- 4\_ColumnVisibility.cs- Util.cs- csdoc – C# documentation and detailed step-by-step instructions directory
      - Family Lab1 - Create Rectangular Column\_CS.rtf- Family Lab2 - Create L-Shape Column\_CS.rtf- Family Lab3 - Add Formula and Material\_CS.rtf- Family Lab4 - Add Visibility Control\_CS.rtf- vb – VB source code directory
        - LabsVb.vbproj- 1\_ColumnRectangle.vb- 2\_ColumnLshape.vb- 3\_ColumnFormulaMaterial.vb- 4\_ColumnVisibility.vb- vbdoc – VB documentation and detailed step-by-step instructions directory
          - Family Lab1 - Create Rectangular Column.rtf- Family Lab2 - Create L-Shape Column.rtf- Family Lab3 - Add Formula and Material.rtf- Family Lab4 - Add Visibility Control.rtf

The source code and instruction documents are part of the
[Family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html) webcast materials.
For your convenience, I am including them
[here](zip/rfa_labs_20091028.zip) as well.

The following summary provides a high-level overview of the implementation.
Detailed information and step-by-step instructions are provided in the source code comments and the dedicated csdoc and vbdoc documents.

#### Lab 1 – Create Rectangular Column

The first lab demonstrates how to implement the following four essential basic steps for creating a new family:

1. Check the family context and category.- Create a simple solid using extrusion.- Set alignments.- Add types.

It makes use of the following classes and methods:

```
doc.IsFamilyDocument
doc.OwnerFamily.FamilyCategory.Name
doc.FamilyCreate.NewExtrusion()
doc.FamilyCreate.NewAlignment()
familyMgr = doc.FamilyManager
familyMgr.NewType()
familyMgr.Parameter(); familyMgr.Set()
```

Here is an overview of the four steps listed above in slightly greater detail:

1. In the first lab, we create a simple rectangular column family defining three column types with different dimensions. Before proceeding with the definition of the geometry and types, we need to ensure that we are working in a family template file and the correct family category.
   The document provides a property IsFamilyDocument to check the former, and the owner family category gives us the category of the template file.- Creating the geometry is easily achieved by defining a rectangular profile in the XY plane and extruding it using the NewExtrusion method. When creating the geometry, we need to account for the Revit database units and convert all our length measurements to feet.- One method to drive the geometry parametrically is to specify alignments between certain parts of the geometry and reference planes. In this very simple case, the template file already provides six reference planes for the upper and lower levels and the right, left, front and back faces, and we have six corresponding faces on the column. We need to specify alignments between these reference planes and the corresponding faces of our solid extrusion. This is done by identifying the matching pairs of solid face and reference plane and calling NewAlignment for each. In this simple case, the solid faces can be identified by their normal vectors and the reference planes by their names. In a more complex case, such as the next example in lab 2, we may have to create the reference planes ourselves, and use more complex algorithms to identify the reference planes and the solid geometry faces to associate with each other.- Finally, we create a few sample types for the family. In this simple case again, the template file already defines the parameters that we require for this, namely Width and Depth. Lab 2 will demonstrate how to define our own new parameters. Since the parameters are given, all we need to do is access the FamilyManager object and use its AddType and get\_Parameter methods to specify the types to create and their dimensions.

Further in-depth details on the process and source code snippets to achieve each of these four steps is given in the separate documents 'Family Lab1 - Create Rectangular Column.rtf' in the csdoc and vbdoc subdirectories for C# and VB, respectively.

#### Lab 2 – Create L-Shaped Column

The second lab builds on the first to define a column with a slightly more complex L-shaped cross section profile.
Due to this, the predefined reference planes and parameters provided by the template file are not sufficient to drive all aspects of the geometry, so we have to create our own additional ones.
To be precise, we need reference planes, parameters and dimensions to determine and measure the thickness of the two 'legs' of the L shape.

Also, in the L-shaped column solid, the normal vector alone is not enough to identify all of the individual solid faces, so that algorithm needs to be refined as well.
In our case, we use the reference plane associated with each solid face to identify it.

Finally, the lab also demonstrates adding new dimensioning to the family definition.
The following steps are added to the existing ones:

- Add reference planes.- Add parameters.- Add dimensions.

The classes and methods used include:

```
doc.FamilyCreate.NewReferencePlane()
familyMgr.AddParameter()
doc.FamilyCreate.NewDimension()
```

Separate C# and VB versions of the detailed instructions, explanations, and source code snippets for this lab are provided by the document 'Family Lab2 - Create L-Shape Column\_CS.rtf' in the csdoc and vbdoc subdirectories of the sample material.

#### Lab 3 – Add Formulae and Materials

In the third step, we further enhance the column family defined in the previous step by defining formulae for the L shape leg width parameters and by assigning a material to one of the family types.

- Add formulae.- Add materials.

The classes and methods used include:

```
familyMgr.SetFormula()
pSolid.Parameter("Material")
familyMgr.AddParameter()
familyMgr.AssociateElementParameterToFamilyParameter()
```

The two parameters are defined as formulae so that the L shape leg width equals a quarter of the column width, and similarly for the depth.
The key here is the family manager SetFormula method.

For the material, an instance parameter for the material group is added to the family and associated with the material parameter of the solid.
Here, the key is the AssociateElementParameterToFamilyParameter method.

Again, separate C# and VB versions of the detailed instructions, explanations, and source code snippets for this lab are provided by the document 'Family Lab3 - Add Formula and Material\_CS.rtf' in the csdoc and vbdoc subdirectories of the sample material.

#### Lab 4 – Add Visibility Control

Performance is a critical aspect of building family content, and significant performance improvements can be achieved by making use of the visibility settings accessible for each element in a family through the FamilyElementVisibility class, which can define which levels of detail and which types of views it appears in.
That is the focus of this final lab, which implements the following:

- Implement a single line representation for the column in coarse view:
  - Single symbolic line representation in plan view.- Single model line representation in model view.- Set the visibility control so that the single line representation is used in coarse view.

The classes and methods used include:

```
doc.FamilyCreate.NewSymbolicCurve()
doc.FamilyCreate.NewModelCurve()
FamilyElementVisibility(FamilyElementVisibilityType.ViewSpecific/Model)
FamilyElementVisibility.IsShownInFine, etc.
pLine.SetVisibility(pFamilyElementVisibility)
```

The detailed documentation and C# and VB code snippets for this lab are provided by the document 'Family Lab4 - Add Visibility Control\_CS.rtf' in the csdoc and vbdoc subdirectories of the sample material.