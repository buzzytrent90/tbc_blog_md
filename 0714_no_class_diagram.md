---
post_number: "0714"
title: "No Revit API Class Diagram"
slug: "no_class_diagram"
author: "Jeremy Tammik"
tags: ['csharp', 'references', 'revit-api', 'views']
source_file: "0714_no_class_diagram.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0714_no_class_diagram.html"
---

### No Revit API Class Diagram

Quite a few people have asked for a Revit API class diagram in the past, and the request continues to pop up every now and then, so here is a post that I can point to in future when that happens.

**Question:** I am looking for a graphical diagram of the Revit API object model, because I cannot find it in the SDK.
Where can I obtain that, please?

**Answer:** Unfortunately, I do not have any very satisfying answer for you.

Once upon a time, the Revit SDK included a class diagram such as you are looking for.
The last version was the
[Revit API Class Diagram.png](zip/) provided
in the Revit 2010 SDK:

![Revit 2010 API Class Diagram](file:///C:/a/lib/revit/2010/SDK/Revit API Class Diagram.png)
That is of course rather outdated by now.

The class diagram has not been updated for the Revit 2011 or 2012 API, nor are there currently any plans to do so.

I personally find the interactive navigation in the Visual Studio object browser and the runtime code analysis and Intellisense more useful for my everyday needs, anyway.

Here is what the development team responded regarding the current situation:

Our tools for creating this diagram would have needed extensive updating to work with the new version and we did not think it was worth the effort.
We also questioned how useful this diagram was to our customers.
If you file a request for this to be reinstated, please include an explanation of you plan to use it, as there might be other ways for us to provide the needed information.

The most reliable browsable information on the Revit object model is available by using the object browser built into Visual Studio:

In the solution explorer, open the 'References' node of your project and right click on 'RevitAPI', then select 'View in Object Browser'.

You can also open the class view node Project References > RevitAPI, navigate to various classes, right click on them and select 'View Class Diagram'. Doing this step by step assembles a partial class diagram for you.

It would theoretically also be possible to use .NET reflection to automatically generate a complete diagram of all classes defined by the Revit API.

An example of such a tool including complete source code is presented in the Code Project article on the *100% Reflective Class Diagram Creation Tool*
[AutoDiagrammer](http://www.codeproject.com/csharp/AutoDiagrammer.asp) and
its updated version
[AutoDiagrammerII](http://www.codeproject.com/KB/WPF/AutoDiagrammerII.aspx).

I tried to apply this tool to RevitAPI.dll years ago, and it did not work right out of the box back then.
On the other hand, it has been updated since then.
In any case, it might be possible to adapt the source code provided so that it works and completes.
I am sure there are many other solutions out there to try, as well.

Regarding the relationship of object types to one another, the derivation relationships of all classes are explicitly noted in the Revit API help file RevitAPI.chm, and the Visual Studio object browser can provide them to you as well.
The developer guide is a good source of background information including other dependencies between the different classes.

What I myself use most of all is the F12 key in the Visual Studio IDE.
It takes me to the definition source code of any variable, method or class, and from there I can continue using F12 to navigate further upwards through the hierarchy graph to its parent and more distant ancestors.

#### Happy Birthday, Autodesk!

Autodesk is 30 years old this month!

[Fourmilab](http://www.fourmilab.ch) provides
all the information from the horse's mouth, sorry, from the founder John Walker, and here is some
[more detailed info on it from Kean](http://through-the-interface.typepad.com/through_the_interface/2012/02/happy-nfy.html).