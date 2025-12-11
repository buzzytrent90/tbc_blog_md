---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: news
optimization_date: '2025-12-11T11:44:16.187569'
original_url: https://thebuildingcoder.typepad.com/blog/1510_nuget_revit_api.html
post_number: '1510'
reading_time_minutes: 3
series: general
slug: nuget_revit_api
source_file: 1510_nuget_revit_api.md
tags:
- references
- revit-api
- sheets
title: Nuget Revit Api
word_count: 661
---

### NuGet Revit API Package
Most of the work of the year has been done, and it is time to settle down and clear out for the new.
Tomorrow is the [winter solstice](https://en.wikipedia.org/wiki/Winter_solstice),
followed by Christmas and
[Yuletide](https://en.wikipedia.org/wiki/Yule), ending with
the [Twelfth Night](https://en.wikipedia.org/wiki/Twelfth_Night_(holiday)).
For me, this is a time of retreat, calm, reflection, and renewal of energy.
![Sunrise at Stonehenge on the Winter Solstice](img/220px-StonehengeSunrise1980s.jpg)
Today brings a nice gift from Andrey Bushman:
- [NuGet Revit API package](#2)
- [RevitLookup using the NuGet Revit API package](#3)
- [Creating a NuGet package from assembly DLLs](#4)
- [More NuGet packages](#5)
#### NuGet Revit API Package
[Andrey Bushman](http://bushman-andrey.blogspot.ru) ([twitter](https://twitter.com/AndreyBushman)),
aka Андрей Бушман, made numerous contributions to the AutoCAD developer community, provided and documented in
his [AutoCAD.NET laboratory](https://sites.google.com/site/bushmansnetlaboratory).
Now he has provided something new for Revit as well,
in [pull request #18 for RevitLookup](https://github.com/jeremytammik/RevitLookup/pull/18),
saying:
> I replaced the explicit Revit API assembly references by the [Revit 2017 x64 .NET API NuGet package](https://www.nuget.org/packages/Revit-2017x64.Base).
> I did it because some users (who aren't programmers) had problems with Revit references when compiling this project code.
> Therefore, I wrote a [four-and-a-half-minute video](https://youtu.be/N0itQZDUEeA) for them explaining the steps to replace the explicit Revit API assembly references by the NuGet package:

Here is a link to the [original Russian version of the video](https://youtu.be/ajgSGp6gp5E).
> I created it for those who aren't familiar with Visual Studio and the handling of the Revit API assembly references.
> If you plan to apply my correction, then this video will stop being useful.
I have in fact applied the correction, and hope that the video remains useful anyway, at least in parts :-)
#### RevitLookup using the NuGet Revit API Package
I followed Andrey's advice and merged
his [pull request #18](https://github.com/jeremytammik/RevitLookup/pull/18) into
the [RevitLookup GitHub repo](https://github.com/jeremytammik/RevitLookup) master branch to
create [release 2017.0.0.7](https://github.com/jeremytammik/RevitLookup/releases/tag/2017.0.0.7),
which automatically updates its Revit API assemblies from NuGet.
#### Creating a NuGet Package from Assembly DLLs
What happens when I need to update to a new release of the Revit API, or if I wish to support an older version?
I asked Andrey:
How easy is it to create an updated NuGet package each time the assemblies change?
> It is very simple and convenient.
To create new NuGet package takes only a couple of minutes.
Updating NuGet package through Visual Studio takes ten seconds.
I can create a video if you want.
NuGet is a very useful and popular tool.
It completely fixes the problems with the lost references.
The project can be copied on any computer and is right there sent to compilation without the need for editing references.
NuGet also allows to monitor the updates for these assemblies and to quickly apply them if the programmer wants.
Do you have a step-by-step documentation for that also, by any chance?
> You can download and read free books about NuGet,
e.g., \*[Pro NuGet – your salvation from dependency hell](http://www.allitebooks.com/pro-nuget-2nd-edition/)\*
and the \*[NuGet 2 essentials](http://www.allitebooks.com/nuget-2-essentials/)\*;
I read the former myself.
#### More NuGet Packages
Here is a link to [all of Andrey's NuGet packages for Revit](https://www.nuget.org/packages?q=owner%3A%22Bush%22+tags%3A%22revit%22).
NuGet packages for Revit 2015 and 2016 are being added soon.
Very many thanks to Andrey for creating, sharing and documenting this useful approach!
I wish everybody a wonderful Christmas time and a peaceful and happy end of the year!