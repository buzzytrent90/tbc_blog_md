---
post_number: "0153"
title: "Revit Library Shape Type Catalogue Parameters"
slug: "catalogue_param"
author: "Jeremy Tammik"
tags: ['family', 'geometry', 'parameters', 'revit-api', 'views']
source_file: "0153_catalogue_param.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0153_catalogue_param.html"
---

### Revit Library Shape Type Catalogue Parameters

A query on the property sets used by Revit on IFC export of steel beams led to the unearthing of some useful documentation of the parameters and dimensions used to define some commonly used shapes in the Revit type library.

**Question:**
I am analysing an IFC export file created by Revit and trying to retrieve the exact dimensions of steel beams in the model.
IFC has a standard specification for these which is publicly available.
Revit apparently uses some own property sets and assigns them to standard IFC beam objects.
These custom property sets are obviously not part of the IFC specification.
What are the exact specification of these parameters and their meanings?

**Answer:**
The Revit beam parameters are indeed exported in a Revit specific property set.
No standard parameters have been defined for the different structural profiles.
Here is some documentation including pictures and parameter values for some structural profiles and other shapes.
These documents were created back in early 2006 illustrating the parameters of the most commonly used shapes in the Revit library.
Please note that a lot of new content files have been created since then:

- [Concrete and Precast Concrete Shapes.zip](zip/Concrete_and_Precast_Concrete_Shapes.zip)- [Steel and Wood Shapes.doc](zip/Steel_and_Wood_Shapes.zip)

The former is a set of images, the latter a document describing the parameter format used in the part catalogue text files.

The images for the concrete and precast concrete shapes describes the dimensions and parameters used in each of the following families:

- Concrete-Rectangular Beam.rfa- Precast-Double Tee.rfa- Precast-Inverted Tee.rfa- Precast-L Shaped Beam.rfa- Precast-Rectangular Beam.rfa- Precast-Single Tee.rfa

The images included are simply screen snapshots of two different views of the geometry and dimensioning and one of the Family Types dialogue listing the corresponding dimensions and parameters of a specific type.

The steel and wood shapes document is more comprehensive and explains the format of the text files used to specify type parameters:

![Type catalogue text file](img/type_catalogue_text_file.png)

Here is a list of the families and corresponding text files described in this document:

- I-Shape(W Shape): M\_W-Wide Flange.txt- M-Shape (Miscellaneous Wide Flange): M\_M-Miscellaneous Wide Flange.txt- HP-Shape (HP-Bearing Pile): M\_HP-Bearing Pile.txt- C-Shape (Channel): M\_HP-Bearing Pile.txt- MC-Shape (Miscellaneous Channel): M\_MC-Miscellaneous Channel.txt- ST- Structural Tee: M\_ST-Structural Tee.txt- MT- Structural Tee: M\_MT-Structural Tee.txt- WT- Structural Tee: M\_WT-Structural Tee.txt- L-Angle: M\_L-Angle.txt- LL-Double Angle: M\_L-Angle.txt- Hollow Tube: M\_HSS-Hollow Structural Section.txt- Circular Hollow Tube: M\_HSS-Round Structural Tubing.txt

- Pipe Column: M\_Pipe-Column.rfa- Open Web Steel Joist: M\_K-Series Bar Joist-Angle Web.rfa, M\_K-Series Bar Joist-Rod Web.rfa, M\_DLH-Series Bar Joist.rfa, M\_LH-Series Bar Joist.rfa- Columns- Lumber: M\_Dimension Lumber-Column.rfa- Columns- Timber: M\_Timber-Column.rfa- Beam- Lumber: M\_Dimension Lumber.rfa- Plywood Web Joist: M\_Plywood Web Joist.rfa- Beam- Timber: M\_Timber.rfa- Open Web Wood Joist: M\_Wood Open Web Joist.rfa

Most of the information included describes the parameters by showing their association with dimensioning on the model geometry, for instance like this for the miscellaneous wide flange:

![Miscellaneous wide flange dimensions](img/miscellaneous_wide_flange.gif)