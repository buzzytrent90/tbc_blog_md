---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.799343'
original_url: https://thebuildingcoder.typepad.com/blog/1337_lifespan_clone_face.html
post_number: '1337'
reading_time_minutes: 6
series: general
slug: lifespan_clone_face
source_file: 1337_lifespan_clone_face.htm
tags:
- csharp
- elements
- family
- geometry
- parameters
- references
- revit-api
- transactions
- views
- walls
title: IFC GUID Access, Life Span and Cloning of Geometry
word_count: 1295
---

### IFC GUID Access, Life Span and Cloning of Geometry

Today, let's talk about the life span of Revit geometry and accessing the IFC GUID of an imported element:

- [Life span and cloning of solids and faces](#2)
- [Accessing the IFC GUID of an imported wall](#3)

First, I'll just mention that I returned safe and sound from the successful fourth and last day of the Milano WebGL workshop, which we concluded with a group session guiding through the
[View and Data API tutorial](https://github.com/Developer-Autodesk/tutorial-getting.started-view.and.data).

I wrote is a short report on that and published it together with a few other current topics on
[The 3D Web Coder](http://the3dwebcoder.typepad.com),
several of which might be of equal interest to you here as well:

- [WebGL workshop report](http://the3dwebcoder.typepad.com/blog/2015/06/webgl-workshop-jobs-anchors-and-functional-javascript.html#2)
- [Working at Autodesk](http://the3dwebcoder.typepad.com/blog/2015/06/webgl-workshop-jobs-anchors-and-functional-javascript.html#3)
- [Resolving the link offset error caused by the Snap responsive design menu bar](http://the3dwebcoder.typepad.com/blog/2015/06/webgl-workshop-jobs-anchors-and-functional-javascript.html#4)
- [Functional JavaScript workshop](http://the3dwebcoder.typepad.com/blog/2015/06/webgl-workshop-jobs-anchors-and-functional-javascript.html#5)

I took a break in the middle of the train trip from Milano back home to Switzerland to meet up with a friend in Frutigen for a hike up the mountains.

We were fortunate enough to be able to spend one night out in the open, enjoy nature, space, tranquility and the light of the half moon, followed by a beautiful day hiking over the
[First](https://en.wikipedia.org/wiki/First_(Kandersteg)) mountain,
providing panorama views over the Bernese alps, overlooking Kandersteg village and the Oeschinensee halfway up (or down) the other side of the valley:

![Jeremy on the First with Kandersteg and the Oeschinensee](/Users/jta/p/2015/2015-06-28_first/590_jeremy_on_first_with_oeschinensee.jpg)

Here is my [First mountain panorama view photo album](https://www.facebook.com/media/set/?set=a.10205983154790852.1073741836.1019863650).

#### Life Span and Cloning of Solids and Faces

Getting back to the Revit API, let's look at this issue raised by Miroslav Schonauer, Solution Architect in Autodesk Consulting.

**Question:**
I’d like to cache a number of Face objects by analyzing Wall/Floor Solids in a Transaction and then use these faces afterwards outside the transaction, i.e.:

```
  FaceCollection instantiate

  -Trans.Start
  Loop Walls/Floors Geometry (Solids)
  FaceCollection populate
  End Loop
  -Trans.End

  FaceCollection use (just read-only geom.access to faces)
```

Questions:

- Is this a supported scenario?
- If NOT, any suggestions other than having to do ALL my work within the Transaction?
- If YES, would it also work with "Dummy Transaction trick" (i.e., I modify Walls/Floors, regen, get faces from such modified Solids and then Abort the Transaction).

To give some more context, I need to find Wall and Floor faces without certain Generic Void family instances that cut them via InstanceVoidCutUtils – I have their element ids, so I can delete them in a temporary transaction – then potentially relocate or manipulate these void instances based on the 'virgin' wall geometry cached in the dummy transaction.

**Answer:**
First, do you even need a transaction? It is not obvious from the code fragment that you need that.

Secondly, and also briefly, your data will not be valid after rolling back the transaction. That is not limited to faces. If you obtain objects within a transaction, then regenerate and roll back the transaction, the validity of your previously obtained objects is uncertain because they may have been scoped out.

It sounds like this is the scenario where you want to make a temporary change and analyse the results of the temporary change. You would have two options:

- Perform the analysis inside the scope of the transaction, before the rollback (but after regenerating the initial temporary change).
- Copy the results of the change so that they are not removed from scope. For example, we have GeometryElement.GetTransformed, which will make a copy of the input. You could pass it an identity transform and then have a copy of the element geometry which will not be affected by transaction scopes (you still need to ensure that you are keeping references to the parent object to avoid conflicts with garbage collection).

**Response:**
I do need the transaction as I am making dummy-changes, as I explained in the description of the context.

Your answer confirms more or less what I suspected.

The reason why the option 1 is not feasible for me is that during my further actions I may need to perform valid editing transactions, so can’t do that within the dummy-one to be rolled back at the end.

Option 2 is what I really need, but I couldn’t find any way to clone faces into equivalent persistent objects. The hint about getting the clones of GeometryElement instances from which I can later retrieve faces (via drilling down its contained geometry objects) sounds like the way forward!

BTW, shame that there is no GetTransformed method on the GeometryObject itself, as that would simplify the process by directly making the clones of faces (and other primitives for other possible uses outside my requirements).

**Answer:**
A Face is too granular to copy and preserve separately; it really represents the confluence of the mathematical definition of the Surface with surrounding Edges, so all of this context would need to be duplicated in order for you to do anything useful with it.

The smallest unit of Revit 3D geometry which could be preserved and used once the original context is gone would be the Solid. We do also have a Solid duplication routine in Revit 2016: SolidUtils.Clone() – so that could be another option, cf.
[cloning a solid](http://thebuildingcoder.typepad.com/blog/2015/05/cloning-a-solid-angelhack-3d-web-fest-and-dubai.html#6) and
[GeometryCreationUtilities for moving and copying solids](http://thebuildingcoder.typepad.com/blog/2015/05/geometry-creation-and-line-intersection-exceptions.html#3).

#### Accessing the IFC GUID of an Imported Wall

**Question:**
I have a wall defined in an IFC file with an id value of '3lDzp1LFjDqwXDAihsyNrA' like this:

```
  #615 = IFCWALLSTANDARDCASE( '3lDzp1LFjDqwXDAihsyNrA',
    #42, '\X2\6A196E9658C1\X0\:(P)PC200:1185289', $,
    '\X2\6A196E9658C1\X0\:(P)PC200:794115',
    #587, #613, '1185289' );
```

How can I retrieve the IFC GUID '3lDzp1LFjDqwXDAihsyNrA' for this wall within the Revit model?

I see it listed in the user interface as the IfcGUID property:

![IfcGUID property](img/ifc_guid_parameter.jpeg)

**Answer:**
This data is stored as a normal parameter on the associated element.

You can look it up by name using the standard `Element` `LookupParameter` or `GetParameters` methods, e.g. like this:

```csharp
  IList<Parameter> ps = e.GetParameters( "IfcGUID" );
// or
  Parameter pGuid = e.LookupParameter( "IfcGUID" );
  string ifc\_guid = pGuid.AsString();
```

The former is preferred, because it ensures you take care of multiple similarly named parameters, cf.
[replacing Element.get\_Parameter by Element.GetParameters](http://thebuildingcoder.typepad.com/blog/2015/06/cnc-direct-export-wall-parts-to-dxf-and-sat.html#2015.2).

While we are at it, I'll also point out that you should absolutely avoid using the language dependent method of looking up parameters by name. You can use the corresponding built-in parameter enumeration value instead, for built-in parameters, or the GUID, for shared parameters. Looking up a parameter by display name is only necessary for custom family parameters.

In this case, the built-in parameters IFC\_GUID and IFC\_TYPE\_GUID will let you access the values for instances and types, respectively, e.g., like this:

```csharp
  Parameter pGuid = e.get\_Parameter( BuiltInParameter.IFC\_GUID );
  string ifc\_guid = pGuid.AsString();
```