---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:13.901837'
original_url: https://thebuildingcoder.typepad.com/blog/0409_shared_type_param.html
post_number: 0409
reading_time_minutes: 3
series: elements
slug: shared_type_param
source_file: 0409_shared_type_param.htm
tags:
- csharp
- doors
- elements
- parameters
- revit-api
- walls
title: Shared Type Parameter
word_count: 637
---

### Shared Type Parameter

The last time we looked at defining new shared parameters for different element categories was a long time ago, in the exploration of doing so for
[model groups](http://thebuildingcoder.typepad.com/blog/2009/06/model-group-shared-parameter.html).
Here is another very basic question on defining shared parameters that we have not previously addressed:

**Question:** I would like to add a new shared parameter to the wall type.
Unfortunately the code I am currently using adds it to the instances instead.
How can I add this parameter to all wall types?
Is it possible?
Can you send me some example code how to do it?
Here is a screen snapshot which shows what I would like to achieve:

![Wall and wall type properties](img/wall_and_wall_type_properties.jpg)

**Answer:** The answer to your query is short and sweet: use NewTypeBinding instead of NewInstanceBinding.

If you look at the description of the Binding class in the Revit API help, you will see the following detailed description:

Binding objects are used to take a parameter definition and bind it to one or more categories. This class is a base class for all types of parameter binding within Autodesk Revit. Once the binding objects are created and added to the document parameters will be added to elements in those categories specified in the binding. There are currently two types of binding available, Instance binding and Type binding. The key difference between the two is that the instance bound parameters appear on all instances of the elements in those categories. Changing the parameter on one does not affect the other instances of the parameter. The Type bound parameters appear only on the type object and is shared by all the instances that use that type. Changing the type bound parameter affects all instances of the elements that use that type. Note, a definition can only be bound to an instance or a type and not both.

To attach a shared parameter to some additional element type, you need to specify its category in the category set used to create the parameter binding.
We explored how to find the category to use in some
[special cases](http://thebuildingcoder.typepad.com/blog/2009/06/model-group-shared-parameter.html) in
the past.
In all of the cases discussed so far, we always used the NewInstanceBinding method.
Your question gives me a welcome opportunity to try out the NewTypeBinding method as well.

The creation application provides the two sibling methods NewInstanceBinding and NewTypeBinding. Both can be fed with the same set of categories. They define a link between a shared parameter definition and the instances and type types of a specific category, respectively.

I updated The Building Coder sample external command CmdCreateSharedParams to add a type binding to walls in addition to the previous instance bindings.
The only really relevant change is the addition of a Boolean argument typeParameter to the CreateSharedParameter method, and the line asking the creation application 'ca' to create the parameter binding, which now looks like this:
```csharp
  Binding binding = typeParameter
    ? ca.NewTypeBinding( catSet ) as Binding
    : ca.NewInstanceBinding( catSet ) as Binding;
```

Here is
[version 2011.0.74.2](zip/bc_11_74_2.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the updated command.

When I run it in a project with no model groups in it, it prints the following to the Visual Studio debug output console:

```
Created a shared instance parameter 'SP1' for the Doors category.
Created a shared instance parameter 'SP2' for the Walls category.
Please insert a model group.
Created a shared instance parameter 'SP3' for the Lines category.
Created a shared type parameter 'SP4' for the Walls category.
```

Afterwards, the wall type has a shared parameter associated with it:

![New wall type shared parameter](img/new_wall_type_shared_param.png)