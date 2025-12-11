---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.307666'
original_url: https://thebuildingcoder.typepad.com/blog/0637_vs_express.html
post_number: '0637'
reading_time_minutes: 3
series: general
slug: vs_express
source_file: 0637_vs_express.htm
tags:
- csharp
- references
- revit-api
- vbnet
title: Visual Studio, C# and VB Express
word_count: 639
---

### Visual Studio, C# and VB Express

Yesterday Kean mentioned that
[debugging AutoCAD.NET projects using the Visual C# and VB Express editions](http://through-the-interface.typepad.com/through_the_interface/2011/08/debugging-autocad-net-projects-using-express-editions.html) is
now directly supported by the
[latest version of the AutoCAD .NET Wizard](http://images.autodesk.com/adsk/files/AutoCAD_2010-2012_dotNET_Wizards.zip).

That prompted me to have a look and see what the current situation is with Revit add-ins.

Obviously, we poor Revit developers do not have such a sophisticated wizard available ... or so I thought!

I installed
[Visual C# Express](http://www.microsoft.com/visualstudio/en-us/products/2010-editions/visual-csharp-express) and started my experiment:

I launched the new installation and selected New Project > Visual C#.

My intention was to select 'Class Library' next and manually set up the Revit API references and all that stuff, then move on to explore how to define appropriate debug settings.

However, guess my surprise to see the Revit Add-in option already listed:

![Visual C# Express 2010 New Project dialogue](img/vcs_exp_new_project.png)

Apparently, the
[Visual Studio Add-In Wizards for Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/04/visual-studio-add-in-wizards-for-revit-2012.html) that
I have installed for the full version of Visual Studio are also picked up by Visual C# Express.

Well, that simplifies matters a lot, doesn't it?

There is actually nothing else whatsoever to do.

Debugging is already set up by the add-in wizard, and Revit has been specified as the start-up project, although it is not displayed in the Visual C# Express debug properties:

![Visual C# Express debug properties](img/vcs_exp_properties_debug.png)

Simply hitting F5 to start debugging launches Revit, and on opening a project and looking at the Add-Ins tab external tools, I see my new Visual C# Express add-in loaded and raring to go:

![Visual C# Express add-in loaded](img/vcs_exp_add-in_loaded.png)

After setting a breakpoint using F9 in Visual C# Express and picking the external command in Revit, the breakpoint is hit and full debugging functionality is available in Visual C#:

![Visual C# Express debugging the Revit add-in](img/vcs_exp_debugging.png)

Wow.
I expected this exploration and blog post to take an hour or two, but I was done after fifteen minutes.

I am assuming that all of the above remains valid analogously for
[Visual Basic 2010 Express](http://www.microsoft.com/visualstudio/en-us/products/2010-editions/visual-basic-express) as well.
Please let me know if you can verify this, or have any problems with either that or the C# version.

So the short and sweet message is: all you need to do is install Visual C# or Visual VB Express and the appropriate
[Visual Studio add-in wizard for Revit](http://thebuildingcoder.typepad.com/blog/2011/04/visual-studio-add-in-wizards-for-revit-2012.html),
and you will be all set to create stand-alone add-ins and develop and debug them using the Express tools.
I hope you have fun!

**Addendum:** Here are some additonal notes on this by Augusto Gonçalves of Autodesk Brazil, the original implementer of the Revit add-in wizards:

- One issue with Express is that the project is initially created under \Temp\, and therefore the add-in manifest file is also configured to use this path.
  Usually on my trainings (with Express), I recommend immediately saving the project after creating it by clicking on 'Save All', selecting a folder, and then updating the path in the add-in manifest file <Assembly> tag.- Also, keep in mind that Express builds the Debug version when you press F5 (or Star Debugging), and the Release version when you select the command in the Build menu.

Thank you, Augusto, for pointing this out!