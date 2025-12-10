---
post_number: "1749"
title: "Pyrevit Invoke Script"
slug: "pyrevit_invoke_script"
author: "Jeremy Tammik"
tags: ['references', 'revit-api', 'sheets']
source_file: "1749_pyrevit_invoke_script.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1749_pyrevit_invoke_script.html"
---

### Rapid Prototyping, Debugging, Testing and Deployment
The [pyRevit rapid application development environment](https://github.com/eirannejad/pyRevit) can
be used for the entire add-in lifecycle, supporting rapid debugging, testing and deployment as well.
Here is a very short post to highlight Ali Tehami's enthusiastic and
inspiring [comment](https://thebuildingcoder.typepad.com/blog/2018/09/five-secrets-of-revit-api-coding.html#comment-4447694964) on
Joshua Lumley's [five secrets of Revit API coding](https://thebuildingcoder.typepad.com/blog/2018/09/five-secrets-of-revit-api-coding.html) that
is certainly of interest to many others, extolling the virtues of `pyRevit`:
> I successfully implemented invoking an external command defined in a stand-alone Revit plugin assembly from [pyRevit](https://github.com/eirannejad/pyRevit)!
> It's proving extremely useful... managed to easily maintain a single DLL assembly on the network server and distributed the functionality through pyRevit's amazing capabilities to almost everyone at my firm.
> It feels amazing when you can make updates on the fly to a single DLL and have it live-ly updated in real-time to all active Revit users in the whole office.
> Another very useful outcome of this implementation was the ease of debugging and testing whether the code base would fail in any different Revit versions... I tested the plugin for all version from 2016 to 2019 in seconds!
> For future reference of everyone, I put an example on my GitHub recycling Joshua's provided sample code into a pyRevit `.pushbutton` in
my [pyRevit beta ideas repository on GitHub](https://github.com/alitehami/pyRevitBetaIdeas_Public/tree/master/aliTehami.extension/BetaConcepts.tab/invoking%20Assemblies.panel/invoke.pushbutton).
Many thanks to Joshua for his great tips, and many thanks to Ali for making such great use combining them with pyRevit and highlighting this for the community!
![Rapid application development](img/rapid_application_development.jpeg)