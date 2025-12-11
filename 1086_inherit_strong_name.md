---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.277568'
original_url: https://thebuildingcoder.typepad.com/blog/1086_inherit_strong_name.html
post_number: '1086'
reading_time_minutes: 5
series: general
slug: inherit_strong_name
source_file: 1086_inherit_strong_name.htm
tags:
- csharp
- elements
- family
- geometry
- parameters
- revit-api
title: No Inheritance and No Strong Naming
word_count: 938
---

### No Inheritance and No Strong Naming

Here are two little items of general long-standing interest that just came up in a Revit API discussion forum:

- [Why you cannot derive a new class from FamilyInstance](#2).
- [Why the Revit API assemblies are not strongly named](#3).

These two issues sometimes come as a surprise to developers experienced in
[OOP](http://en.wikipedia.org/wiki/Object-oriented_programming) and other CAD system programming.

However, there are good reasons for both.

Here are the detailed explanations:

#### Why You Cannot Derive a New Class From FamilyInstance

**Question:** I would like to extend some classes provided by the Revit API, e.g. by inheriting from them and implementing my own derived classes.
I think it would be very useful to specialise some classes, like FamilyInstance.

**Answer:** Do you have any particular goals behind inheriting from FamilyInstance?
What would you like to be able to accomplish that inheritance or extensibility would allow?

**Question:** I created an add-in that uses a Revit family to delimitate an area.
I call it a 'marker' and can identify it by its symbol.
It helps identify where and how other element need to be placed.

Having a public constructor will allow us to inherit our marker class from FamilyInstance.
In fact, a marker IS a family instance, with just some other properties.
Now we have to copy some data from the FamilyInstance to the marker it belongs to, like its unique id, location, rotation and so on.

It would be easier for us to serialize and deserialize the class being sure correct data is copied to the database.

Also the code understandability would be improved, since calling `new FamilyInstance(x,y,z, rotation, symbol)` is much more obvious than calling a static constructor.

Any programmer would expect that calling a constructor such as this would drop a new instance into the project.

**Answer:** Thank you for the clarification.

The structure of Revit and it’s API is a little bit different than I think you expect, so this is not going to work as well as you hope.

Revit API objects are managed wrappers around native C++ objects used by Revit’s UI and database.
Therefore, a FamilyInstance in the API does not actually contain the properties you see – the API code accesses them through the matching native object’s members. So extending our managed classes would not actually allow for new data to end up in the Revit file during serialization – we don’t know how to get the data from your inherited class.

We don’t supply public constructors for many of these objects for precisely this reason – much more needs to happen than just building the object and populating the x, y, z, etc. – the document tables need updating, dependent elements must be added, relationships established etc. So this is what we have methods like NewFamilyInstance and, for our newer element creation calls, static creation methods taking the document and necessary parameters.

If you want to get additional serialized data onto a Revit element, you should look at
[Extensible Storage](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.23) in the Revit API.

If you want to make code easier to write and want to 'add' additional members to our classes (so you could have a call to a FamilyInstance.GetMarker method or something like that), you can look at
[C# extension methods](http://thebuildingcoder.typepad.com/blog/2010/02/getpolygon-extension-methods.html) –
however, adding such methods will not allow additional data to be stored in those classes unless you implement these members using Extensible Storage.

#### Why the Revit API Assemblies are not Strongly Named

**Question:** Looking at the Revit API documentation, I do not see any mention of
[strong naming](http://en.wikipedia.org/wiki/Strong_key).

Is there a particular reason for that?

This can cause several security issues, and some customers may require us to sign our Revit add-in assemblies, which we currently cannot do.

**Answer:** Yes, this is true.
There are good reasons for this.

Here is an explanation why
[you cannot sign your AutoCAD .NET plug-in with a strong name](http://adndevblog.typepad.com/autocad/2012/07/can-i-sign-my-autocad-net-plug-in-with-a-strong-name.html).

It covers all the technical details and includes some workarounds, which all apply to Revit add-ins as well.

Note that Microsoft modified the strong name verification system so that
[some security measures related to signatures are now bypassed](http://blogs.msdn.com/b/shawnfa/archive/2008/05/14/strong-name-bypass.aspx) by
default.

#### Wise Words

I hope these explanations make sense.

There are quite a few surprises lurking in the Revit API and the entire Revit product paradigm for unwary developers experienced in other less parametric and BIM oriented CAD systems, as we have already repeatedly seen in the past:

- [BIM versus free geometry and product training](http://thebuildingcoder.typepad.com/blog/2012/02/bim-versus-free-geometry-and-product-training.html)
- [Porting an AutoCAD application](http://thebuildingcoder.typepad.com/blog/2012/10/porting-an-autocad-application.html)

In fact, as I often point out, one of the hardest tasks for a budding Revit add-in developer experienced in ObjectARX or other CAD system programming is to forget her accumulated previous experience and open her mind for something completely new and different.

[Learning something new](http://www.wikihow.com/Become-a-Lifelong-Learner) and
[letting go of something old](http://www.marcandangel.com/2013/09/02/5-things-you-should-know-about-letting-go)
can prove very challenging indeed.

And very rewarding.

Good luck!