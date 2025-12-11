---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:13.439576'
original_url: https://thebuildingcoder.typepad.com/blog/0145_core_structural_layer.html
post_number: '0145'
reading_time_minutes: 1
series: structural
slug: core_structural_layer
source_file: 0145_core_structural_layer.htm
tags:
- elements
- revit-api
- walls
- structural
title: Core Structural Layer
word_count: 194
---

### Core Structural Layer

**Question:**
How can I identify the core structural layer in a Revit host element such as a floor or a wall?

**Answer:**
A heuristic method for this might be searching for the thickest layer.
Here is a VB example of searching for the thickest layer of a given floor type and identifying its material, using its CompoundStructureLayerArray layers:

```
Dim m As Material = Nothing
Dim layers as CompoundStructureLayerArray = floorType.CompoundStructure.Layers
Dim thicknesses(layers.Size) As Single
For i As Integer = 1 To layers.Size
  thicknesses(i) = layers.Item(i - 1).Thickness
Next
Dim maxIndex As Integer
Dim maxV As Single
MaxValOfIntArray(thicknesses, maxIndex, maxV)
m = layers.Item(maxIndex - 1).Material
```

Unfortunately, the core structure layer may not always be the thickest.
Sometimes some other layer such as an insulation of type ThermalOrAir or similar may be thicker.

A more reliable method is to use the CompoundStructureLayer.Function property, which indicates the actual usage of the layer.
The core structure layers property value is CompoundStructureLayerFunction.Structure.
You can go through the wall type or floor type layers as in the example above and determine the layer whose CompoundStructureLayer.Function equals CompoundStructureLayerFunction.Structure.