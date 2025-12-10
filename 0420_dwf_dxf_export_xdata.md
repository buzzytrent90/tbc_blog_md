---
post_number: "0420"
title: "DWG and DXF Export Xdata Specification"
slug: "dwf_dxf_export_xdata"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'parameters', 'revit-api']
source_file: "0420_dwf_dxf_export_xdata.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0420_dwf_dxf_export_xdata.html"
---

### DWG and DXF Export Xdata Specification

I am still on my diving vacation on
[Ko Tao](http://en.wikipedia.org/wiki/Ko_Tao).
Unfortunately, the visibility under water was really terrible yesterday, and diving conditions are not that good in general.
The rain that normally falls in November never came last year and now the water is warmer than usual.
The
[corals](http://en.wikipedia.org/wiki/Coral) that enter into a symbiosis with certain algae which give them their colour have shed their guests and are all white, or even worse, covered in dirt and grey or green.

Yesterday we dived at the
[Southwest Pinnacle](http://en.wikipedia.org/wiki/Diving_in_Ko_Tao) at
a depth of 22 meters and aborted the dive after 25 minutes due to almost zero underwater visibilility, the worst ever seen, said some.
We went over to Hin Ngam at Aow Leuk beach for a shallower dive away from the current, and that was a bit better.
Anyway, today I am taking a day off diving and writing this instead.

Returning to Revit programming news, here is a technical note that has been available for quite a while that Joe just pointed out in a recent case and that I was not previously aware of:

#### Revit DWG/DXF export: specification of the extended data (xdata)

**Question:** A DWG/DXF file exported from Revit contains Extended Data (or Xdata) attached to each entity.
Could you explain what this data represents?

**Answer:** Every DWG (DXF) file created by Revit during the export contains Extended Data (or Xdata).
Xdata created by Revit has an associated application name "REVIT."
Each pair of values in the list defines the type identifier code and the value of an object parameter.
Here is a list of the data stored:

1. Element id- Category id- Sub-category id- Material id- Type id- Is Material Overridden By Face

The first five data items are structured in pairs:

- First the above code 1 to 6 stored as a 16-bit integer with the DXF group code 1070.- Second the id data stored as a 32-bit integer with the DXF group code 1071.

The material override information only applies to polymesh objects.
There is no explicit value for that parameter.
If the record for it exists in the object, its value is true, otherwise false.

Each object created by Revit export can contain one record for each parameter.
The values of parameters are inherited by the hierarchy of the objects (e.g., from instance to block to polymesh).
The value defined in the object overrides the inherited value.

The following is an example of an entity containing Revit's extended data in DXF format:

```
AcDbEntity
  83
D-DOOR-SYMB
...

AcDbPolyFaceMesh
...

Extended entity definition data:
1001
REVIT
1002
{
1070
     1
1071
    52525
1070
     2
1071
    31431
1070
     3
1071
    27901
1070
     4
1071
    66553
1070
     5
1071
    12345
1070
     6
1002
}
1001
```

This means that this polymesh entity came from the host object element with element id 52525, category id 31431, sub-category id 27901, material id 66553, type id 12345, and its material was defined in the Revit face itself and not inherited from the object definitions.

#### Named Object Dictionary Entries

The mapping between the category, sub-category and type ids and their names is stored in corresponding tables in the Named Object Dictionary (i.e., an AcDbDictioinary of the name "REVIT\_DICTIONARY").
REVIT\_DICTIONARY contains the following identifiers:

- REVIT\_CATEGORY\_MAPPING: The definition of the mapping between category ids and category names.- REVIT\_SUB\_CATEGORY\_MAPPING: The definition of the mapping between sub-category ids and sub-category names.- REVIT\_TYPE\_MAPPING: The definition of the mapping between type ids and type names.

Each table will be represented by an Xrecord stored as a DXF resbuf chain.
Each entry in the table represents the data pair with an id of type long and the name of type text.