---
post_number: "0483"
title: "Project Vasari API"
slug: "project_vasari_api"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'revit-api', 'views', 'walls']
source_file: "0483_project_vasari_api.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0483_project_vasari_api.html"
---

### Project Vasari API

I mentioned the
[Autodesk Project Vasari](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-project-vasari.html) last week,
and the first
[question from Fernando Malard](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-project-vasari.html?cid=6a00e553e16897883301348933a6ba970c#comment-6a00e553e16897883301348933a6ba970c) was
whether it also provides an API.

The question makes a lot of sense on this blog, since everything we discuss here is related to the Revit API.
I thought Vasari would be an exception.
Surprisingly, this is not the case.

Ritchie Jackson asked the same thing, and received the following very satisfying reply from Matt Jezyk:

**Question:** Vasari has the potential to be a great product but I can't find the 'Macro' button.
A coding or scripting facility is essential for a conceptual design tool such as this.

For post-graduate students like myself, inclusion of the above facility would make the difference between adoption or rejection of the application.

I find the Revit API extremely useful, as you can see from my recent posts on

- [Blends, Hermite splines and derivatives](http://thebuildingcoder.typepad.com/blog/2010/11/blends-hermite-splines-and-derivatives.html)- [Complexity versus constructability](http://thebuildingcoder.typepad.com/blog/2010/11/complexity-versus-constructability.html)

**Answer:** Thanks for the interest and feedback.
Vasari does not include the VSTA macro feature, but it does have a powerful .NET based API, a subset of the Revit API.
We have many existing customers, students and faculty interested in the exact workflows you are interested in, so we'd love to work closely with you on topics in this area.
Many of the people working on Vasari and the conceptual modelling features have a background in parametric modelling and programming.

Vasari has an API that basically is a subset of the Revit API – it exposes the conceptual modelling and family editor APIs.
If you are familiar with the Revit API you should have no issues.
If you develop something based on the Revit 2011 API that only uses the family editor or conceptual mass APIs it should work in Vasari.
If, however, you try to use the creation methods for walls, floors, beams, ducts, columns in Vasari, then the API will return an exception, because these detailed elements are not allowed to be created in Vasari.

Also note Vasari is a technology preview and ADN does not provide any support for it.

Wow. Sounds good to me.

Many thanks to Ritchie and Matt for this exciting information!

Ritchie went on to explore further, and has the following to say:

#### Vasari and the API

After taking a look at Vasari, I thought that a great package had been let down by the lack of access to the API.
Thanks to Matt Jezyk of Autodesk for pointing out that this was not the case and that a sub-set of the Revit API is in fact accessible.

I found that Vasari automatically creates the following directories on launching:-

- C:\Program Files\Autodesk\Vasari TP1.0\Program\Addins- C:\Documents and Settings\All Users\Application Data\Autodesk\VAS TP1.0- C:\Documents and Settings\myDirectory\Application Data\Autodesk\Vasari\Addins\TP1.0

You need to put Add-In Manifest files into the last one which appears to be the current user directory.
Although the first also has an Addin folder it doesn't recognise user-created items, even if a TP1.0 sub-folder is created.

Vasari will now show an Add-Ins Tab:-

![Interface](img/ritchie-Vasari-01-App.jpg)

The model below was created in Vasari from an external command.
It's fully parametric with the frame splines being evaluated at a chosen number of intervals to set out the 'louvres'.
The projection extents of the latter items are proportional to the iteration numbers:-

![Shelter Model](img/ritchie-Vasari-01.jpg)

Here is the
[DLL for that model](zip/RJ_Vasari_01.dll) as
well as
[a basic "Greetings" test](zip/RJ_Vasari_02.dll) for
anyone to try them out

Jeremy adds: I used
[Reflector](http://thebuildingcoder.typepad.com/blog/2008/10/converting-between-vb-and-c-and-net-decompilation.html) to
decompile Ritchie's Hello World sample, just to prove that it really is a completely normal Revit add-in:

![Vasari Hello World decompiled](img/vasari_hello_world.png)