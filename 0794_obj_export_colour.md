---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.606828'
original_url: https://thebuildingcoder.typepad.com/blog/0794_obj_export_colour.html
post_number: 0794
reading_time_minutes: 3
series: general
slug: obj_export_colour
source_file: 0794_obj_export_colour.htm
tags:
- elements
- revit-api
- views
title: OBJ Model Exporter with Colours
word_count: 683
---

### OBJ Model Exporter with Colours

I discussed the
[first take of my OBJ exporter](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-exporter-take-one.html) last
week, still lacking colour support.

Adding colours turned out to be more complex than I had expected, and the result as it currently stands includes a couple of kludges, I'm afraid.

Still, for a quick hack it does a decent job of getting a rough idea across.
For instance, here is the result of exporting the basic sample project rac\_basic\_sample\_project.rvt included in the Revit product distribution:

![Basic sample model in OBJ](img/obj_export_basic_model_inst.png)

The resulting OBJ file size is 363 KB, and here is the element count for that export operation:

![Basic sample model element counts](img/obj_export_basic_model_inst_msg.png)

I added code to access the element category's material and colour, as well as the material and colour of an individual face of the solid, in case it has been overridden.
I also added support for a default colour, in case my algorithm produced nothing useful.

Some of the places I looked to learn about OBJ materials were the
[OBJ file format](http://www.cs.clemson.edu/~dhouse/courses/405/docs/brief-obj-file-format.html), a
[very detailed MTL format](http://local.wasp.uwa.edu.au/~pbourke/dataformats/mtl) description and a
[shorter one](http://people.sc.fsu.edu/~jburkardt/data/mtl/mtl.html) including pointers to sample MTL files.
Unfortunately, the specific link of greatest interest to me, to definitions of simple primary color materials, does not work.

Just like the vertices, the colours also need to be added to a lookup table in order to avoid duplication.
For an OBJ model, colours are defined by materials, which are specified in a separate material library file with the extension MTL, so the exported no created two parallel files.
For simplicity's sake, I just defined the ambient and diffuse colours for my materials and named them using the hexadecimal representation of the Revit colour byte values.
The first couple of entries in my material file look like this:

```
newmtl C0C0C0
Ka 0.75 0.75 0.75
Kd 0.75 0.75 0.75
newmtl DBDBDB
Ka 0.85546875 0.85546875 0.85546875
Kd 0.85546875 0.85546875 0.85546875
newmtl 453B2C
Ka 0.26953125 0.171875 0.23046875
Kd 0.26953125 0.171875 0.23046875
newmtl 7F7F7F
Ka 0.49609375 0.49609375 0.49609375
Kd 0.49609375 0.49609375 0.49609375
newmtl 000000
Ka 0 0 0
Kd 0 0 0
newmtl A7CAE4
Ka 0.65234375 0.890625 0.7890625
Kd 0.65234375 0.890625 0.7890625
newmtl DCCBAA
Ka 0.859375 0.6640625 0.79296875
Kd 0.859375 0.6640625 0.79296875
```

In the OBJ file, I start out by specifying the material library with this initial statement:

```
mtllib jbasic2.mtl
```

The usemtl statement specifies which material to use, followed by faces referring to the vertex indices:

```
usemtl DBDBDB
f 210 212 310
f 310 212 311
f 311 312 313
f 311 313 310
f 314 315 211
f 211 209 314
```

Here is
[ObjExport.zip](zip/ObjExport.zip) including
the entire source code, Visual Studio solution and add-in manifest for the OBJ exporter in its current state.

I would be glad to hear from you if you find this useful, and even more so if you discover possible enhancements to it.

The next step will be to explore how to make the model view ubiquitously available to mobile devices.

One tool for cloud-based access to 3D content that I just heard of is
[Sunglass](http://sunglass.io),
supporting integration of desktop CAD and storage utilities such as Dropbox and, especially interesting for developers, including API access.

**Addendum:** Do not use the obsolete ObjExport code provided above, since it does not handle multiple solids in the way required for a valid and complete export.
Please refer to the updated version 2 supporting
[multiple solids per element](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-multiple-solid-support.html) instead.