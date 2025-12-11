---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: documentation
optimization_date: '2025-12-11T11:44:13.280676'
original_url: https://thebuildingcoder.typepad.com/blog/0045_autocad_in_revit.html
post_number: '0045'
reading_time_minutes: 3
series: general
slug: autocad_in_revit
source_file: 0045_autocad_in_revit.htm
tags:
- csharp
- revit-api
title: Running AutoCAD inside Revit
word_count: 506
---

### Running AutoCAD inside Revit

The preceding post on
[RealDWG and object enablers](http://thebuildingcoder.typepad.com/blog/2008/11/realdwg-and-object-enablers.html)
discussed one potential method for making use of AcDb functionality to read and write AutoCAD DWG files inside of Revit.
Assuming that you have a license for a full version of AutoCAD in addition to your Revit one, there is another easy method to achieve this which has been implemented and tested.

It is possible to run AutoCAD as a full-blown process in the background inside Revit.
This functionality is new in AutoCAD 2009, and the code we present is based on a demo by Kean Walmsley on
[embedding AutoCAD 2009 in a standalone dialog](http://through-the-interface.typepad.com/through_the_interface/2008/03/embedding-autoc.html).
Obviously this requires a full version of AutoCAD to be installed on the machine.
Among other things, Kean's code defines a MainForm class,
which can be used to load and run AutoCAD and access its API functionality.

Markus Hannweber of
[SOFiSTiK AG](http://www.sofistik.de)
has made use of this within a Revit plug-in.
Here are some examples of what is possible:

```csharp
MainForm mainform = new MainForm();
AxACCTRLLib.AxAcCtrl c = mainform.axAcCtrl1;
c.Src = "d:\\demo\\embed.dwg";
c.PostCommand("(setvar \"cmddia\" 0) ");
c.PostCommand("(arxload\"sofibldarx\") ");
c.PostCommand("(load\"blist\") ");
c.PostCommand("SOF\_BLIST ");
c.PostCommand("(S:ENDACAD) ");
c.Src = null;
```

The individual lines do the following:

- Instantiate Kean's MainForm class and open AutoCAD in the background.
- Initialise a variable 'c' to address the AutoCAD control. It also helps keep the source code line length under control.
- Load a drawing, object enablers are automatically loaded with it, if set up to do so.
- Suppress AutoCAD command line output.
- Load an ARX application into AutoCAD.
- Load an AutoLISP application into AutoCAD.
- Start any application command, whether defined in Lisp or ARX.
- Run a command to reset the 'drawing dirty' flag, so the next command can quit AutoCAD without user interaction.
- Close AutoCAD.

Obviously, the actions you trigger in your background AutoCAD process must not require any user interaction, or, if they do, you must feed them the appropriate command line user input through something like mainform.axAcCtrl1.PostCommand(...).

Der Grund fr das alles:

- Unsere Stahlliste geht nur ber einen Aufruf aus AutoCAD.
- Da fr die Stahlliste nur die Anzahl aber nicht die Anordnung wichtig ist, brauchen wir uns um einen korrekte Plazierung in AutoCAD nicht zu kmmern. Man sieht die Zeichnung nicht. Kann ich dir vieleicht mal zeigen in Vegas.

Das ganze Projekt kann ich natrlich nicht schicken, der wesentliche Unterschied zu Kean ist, dass ich die Dialogbox mit ACAD (MainForm) nicht aufmache! In MFC wrde ich sagen ich lasse einfach DoModal weg. (.NET Befehl hab ich vergessen)
Bei Kean geht die Dialogbox auf und zeigt ACAD zunchst ohne alle Mens, aber im wesentlichen ist alles bearbeitbar.
Aber einfach, gucke
[da](http://through-the-interface.typepad.com/through_the_interface/2008/03/embedding-autoc.html),
da stehen sogar einige meiner Fragen drauf.