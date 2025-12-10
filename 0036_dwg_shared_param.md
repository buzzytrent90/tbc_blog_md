---
post_number: "0036"
title: "Adding a Shared Parameter to a DWG File"
slug: "dwg_shared_param"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'parameters', 'revit-api', 'walls']
source_file: "0036_dwg_shared_param.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0036_dwg_shared_param.html"
---

### Adding a Shared Parameter to a DWG File

I was recently asked whether it is possible to attach a shared parameter to a DWG drawing file imported into a Revit model. The exploration of this issue was interesting enough for me to like to share it.

First of all, if you are interested in attaching a shared parameter to Revit elements, there are some pretty clear examples available demonstrating how to achieve this. One is the FireRating sample in the Revit SDK, the other is the Revit API introduction labs suite 4-3-1, 4-3-2, and 4-3-3, which is based on the FireRating sample and includes some extensions as well. The former is in VB, the latter both C# and VB.

When defining a new shared parameter, you need to specify which categories it applies to.
The FireRating sample attaches a shared parameter 'FireRating' to all doors in the Revit model, and it identifies them using the BuiltInCategory OST\_Doors. The labs extend this slightly in order to show that you can attach a shared parameter to walls as well. Doors are standard family instances, whereas walls use a system family.

The interesting thing that cropped up exploring this for imported drawing files is that the process of importing the drawing into Revit causes Revit to create a new category specifically for the inserted file on the fly, named the same as the original drawing file. For instance, after inserting Drawing1.dwg, a new category named Drawing1.dwg appears in Document.Categories. Therefore, for imported drawing files, we obviously cannot use a built-in category to identify which instances to attach the parameter to.

So, to create a new shared parameter for an inserted drawing file, you will need to keep track of the drawing name and use that to identify the category to define the shared parameter for. I have tested this procedure in the attached C# version of the labs and verified that it works.

The set of categories defined in a document is managed using the Revit API Categories container class.
One nice thing about this class is that it provides keyed access to the categories it contains by either category name or by built-in category enumeration value. In C#, we see the following two overloads:

```
  public virtual Category get_Item(
    string key );

  public virtual Category get_Item(
    BuiltInCategory categoryId );
```

In our case, this means that we can use both BuiltInCategory.OST\_Doors and "Drawing1.dwg" as identifiers to define the category we are interested in:

```csharp
  static public BuiltInCategory Target
    = BuiltInCategory.OST\_Doors;
  static public BuiltInCategory Target
    = BuiltInCategory.OST\_Walls;
  static public string Target
    = "Drawing1.dwg";
```

In this simple example, we are using a hard-wired drawing file name. In real life, this would have to be managed more flexibly.

For the full code of the external command class Lab4\_3\_1\_CreateAndBindSharedParam that creates the shared category for an imported drawing file, please look at the corresponding C# file in the Revit API introduction labs provided
[here](http://thebuildingcoder.typepad.com/blog/files/rac_labs_20081107.zip).