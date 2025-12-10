---
post_number: "0994"
title: "Setting the Compound Structure Core and Shell Layers"
slug: "set_core_layer"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'walls']
source_file: "0994_set_core_layer.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0994_set_core_layer.html"
---

### Setting the Compound Structure Core and Shell Layers

Modifying the compound layer structure of a wall or floor type is pretty well documented, and we already discussed the use of the SetCompoundStructure method to
[update the compound layer structure](http://thebuildingcoder.typepad.com/blog/2012/03/updating-wall-compound-layer-structure.html).

However, how to define which layers are and are not part of the core is less obvious, which lead to the developer query below.

Before I get to that, here is a snapshot of the north-east ridge and summit of the
[Hinderi Spillgerte](http://en.wikipedia.org/wiki/Spillgerte)
([German](http://de.wikipedia.org/wiki/Spillgerte)) that
I recently climbed:

![Hinderi Spillgerte NE ridge and summit](file:////j/photo/jeremy/2013/2013-08-06_hintere_spillgerten/0661_ne_ridge_summit_cropped.jpg)

The climb is easier than it looks, grade II-III+, includes a number of abseil sections, both ascending the north-east ridge and descending the south one, and became a bit risky at the end due to getting caught in a thunderstorm
([more photos](https://www.facebook.com/media/set/?set=a.10201093030740807.1073741828.1019863650&type=3))...

https://www.facebook.com/media/set/?set=a.10201093030740807.1073741828.1019863650&type=3

Anyway, back safe and sound at our desks, let's return to setting the Revit core compound structure layers:

**Question:** I found several methods in the CompoundStructure class to check the position of the core, e.g. GetCoreBoundaryLayerIndex, GetFirstCoreLayerIndex, GetLastCoreLayerIndex and IsCoreLayer, but no method to change it.

How can I achieve that, please?

**Answer:** As mentioned above, you can change the compound layer structure a wall or floor type by creating a new collection of layers and setting them on the type using the
[SetCompoundStructure method](http://thebuildingcoder.typepad.com/blog/2012/03/updating-wall-compound-layer-structure.html).

While this sets the layers, it does not define which of them are parts of the core.
In fact, the sample code uses the GetFirstCoreLayerIndex method to determine the first layer in the core, but presents no method to change it.

Defining which CompoundStructure layers are part of the core is done using the CompoundStructure.SetNumberOfShellLayers method.

For example, if you have 9 layers and want layers 0-3 in the exterior shell, 4-5 in the core, and 6-8 in the interior shell, you could use these two calls to achieve that:

```csharp
  Cs.SetNumberOfShellLayers(
    ShellLayerType.Exterior, 4 );

  Cs.SetNumberOfShellLayers(
    ShellLayerType.Interior, 3 );
```

**Response:** Thank you very much, that is just what I was looking for.

I modified my application and the two-line code example you provided works perfectly.

I had in fact seen the SetNumberOfShellLayers method, but its use was not obvious to me, nor the fact that the so-called 'shell' defines and changes the contents of the core.