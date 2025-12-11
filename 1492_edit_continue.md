---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:16.149394'
original_url: https://thebuildingcoder.typepad.com/blog/1492_edit_continue.html
post_number: '1492'
reading_time_minutes: 7
series: general
slug: edit_continue
source_file: 1492_edit_continue.md
tags:
- filtering
- references
- revit-api
- sheets
- windows
title: The Building Coder
word_count: 1327
---

### AI, Edit and Continue
I am still in Munich supporting the
one-week [Forge accelerator](http://autodeskcloudaccelerator.com) workshop,
returning back to Switzerland by train tonight.
For ecological reasons, I prefer to avoid flying whenever I possibly can.
Lots of exciting development is going on here, and we are making great progress.
I have another important detail to report from the
the [RTC Revit Technology Conference Europe](http://www.rtcevents.com/rtc2016eur) in
Porto last week, and other little titbits to share as well:
- [Use AddInManager and attach to process to edit and continue](#2)
- [Resurrect](#2.1)
- [Can I replace myself by artificial intelligence?](#3)
- [Autodesk Design Graph](#4)
- [Artificial intelligence recognises conversational speech](#5)
- [Artificial intelligence teaching assistant](#5.1)
- [Good portable loudspeakers](#6)
#### Using AddInManager and Attach to Process to Edit and Continue
As I mentioned in the notes from
the [Revit API discussion panel](http://thebuildingcoder.typepad.com/blog/2016/10/rtc-revit-api-panel-idea-station-edit-and-continue.html),
one of the topics we touched upon was how
to [edit and continue, aka debug without restart or live development](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.49),
i.e., enable the full cycle of debugging an add-in, discovering an error, editing the code to fix it, and continuing debugging in the same Revit model without being forced to terminate and restart Revit and reload the project.
This is not immediately achievable, because if you use the Visual Studio debugger to launch Revit, it will lock the DLL that you are debugging, and you will not be able to edit it.
The standard suggestion to work around this that I made in the past is to convert your add-in code to a macro for debugging, and then convert it back again when done.
Various other suggestions have been made (and are listed in [The Building Coder topic group 5.49](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.49) mentioned above) to enable reloading the add-in without converting the code to a macro, e.g., by loading it from an in-memory byte stream instead of a file.
Michael Porskær of [Orbicon A/S](http://www.orbicon.dk) pointed out that he uses a much simpler and more direct method by running Revit.exe in normal standalone mode instead of launching it from the Visual Studio debugger, attaching the debugger to the Revit.exe process, and using the AddInManager provided with the Revit SDK to load the add-in.
Here are the exact steps:
1. Load add-in project in Visual Studio.
2. Compile/build DLL.
3. Start Revit externally.
4. In Visual Studio, click on Debug > Attach to Process.
5. Find and select the Revit.exe process in the list.
6. Load the DLL-file through Add-In Manager in Revit (you will find Add-In Manager in the SDK).
7. Click on the command you would like to debug and click Run.
8. Debug your code.
9. Stop debug.
10. Fix your code and start from number 4 again.
Michael published the [52-second video \*DebugCommandWithoutRestartRevit\*](https://youtu.be/I3NA2VUB8Hc) showing these steps live:

Thank you very much, Michael, for documenting and sharing this!
#### Resurrect
I am very glad that Sander Obdeijn of [Itannex](http://www.itannex.com/) is here participating in the Munich Forge accelerator.
He has extremely deep Revit experience, both in the user interface and API.
He uses the approach described above to debug his Revit add-ins and pointed out a handy tool to make it yet more efficient:
> You can use the [Resurrect Visual Studio extension](https://visualstudiogallery.msdn.microsoft.com/043c7c91-e127-4616-bce0-39b869cee4b3) ([source on GitHub](https://github.com/jwaliszko/Resurrect)) to automatically re-attach the debugger to previously debugged processes with one single click.
![Resurrect](img/resurrect_icon32.png)
#### Can I Replace Myself by Artificial Intelligence?
I answered a lot of repetitive questions in the last couple of weeks and months – not to mention years and decades.
That led me to realise recently that I am pretty sure that at least 10-20% of my work could be automated.
The tasks that seem most easy to address are responses to simple questions on getting started raised in email messages and discussion forum threads.
I would need tools to:
- Monitor all my computer interaction, specifically email and discussion forum thread responses.
- Observe and learn which cases lead to similar and simple replies.
- Be able to automatically and autonomously scan my email input and certain discussion forums.
After a couple of months of training, I think the system should be able to:
- Monitor my incoming email and a couple of forums for issues to arise that it recognises.
- Automatically generate a response and rate the probability that it is appropriate.
Initially, I would expect it to rate all probabilities at zero percent.
At that point, I should be able to start training it.
Some of the auomatically generated reponses I would classify as acceptable and use as is.
Others (or all?) might need some tweaking, and could also be used after slight improvement.
Same might be way off, of course.
Training would begin.
Who knows, maybe I am already much further, and the text you read here now was AI generated?
Check out these other AI, CAD and communication related items:
- [AlphaGo](https://en.wikipedia.org/wiki/AlphaGo) (already [mentioned in January](http://thebuildingcoder.typepad.com/blog/2016/01/bim-programming-madrid-and-spanish-connectivity.html#7))
- [Autodesk Design Graph](#4)
- [AI recognises conversational speech](#5)
- [AI teaching assistant](#5.1)
#### Autodesk Design Graph
[Design Graph](https://dg.autodesk.com) provides a new way to explore your 3D design data using shape-based machine learning to recognize and understand parts, assemblies and entire designs. It learns to identify the relationships between all parts within and across all of your designs, irrespective of whether cross-references exist. It learns to interpret designs in terms of those parts and provides a way to navigate your data using simple text search, learned categories of parts, shape similarity, usage patterns and even smart filters for part numbers, materials and other properties.
#### Artificial Intelligence Recognises Conversational Speech
Microsoft announced progress and
a [historic achievement reaching human parity in conversational speech recognition](http://blogs.microsoft.com/next/2016/10/18/historic-achievement-microsoft-researchers-reach-human-parity-conversational-speech-recognition).
#### Artificial Intelligence Teaching Assistant
Matt Taylor pointed out the interesting report on an [artificial intelligence course creating an AI teaching assistant](http://www.news.gatech.edu/2016/05/09/artificial-intelligence-course-creates-ai-teaching-assistant).
The students didn’t even know their TA was a computer.
The key phrase for me is that "... the number of questions increases if you have more students, but the number of \*different\* questions doesn’t really go up ... the same questions [are repeated] over and over again."
That is exactly what I have been noting more and more in my work recently, and what spawned my interest in this.
"The students ... studying artificial intelligence ... were unknowingly interacting with it... The goal is to have the virtual teaching assistant answer 40 percent of all questions by the end of year."
Thank you very much, Matt, for pointing out this perfect example of what I am after!
#### Good Portable Loudspeakers
On a completely different topic...
I went to a nice little [open floor](http://openfloor.org/) dance event last Sunday evening immediately after my arrival here in Munich, in
the [Mohr-Villa Kulturzentrum](http://www.mohr-villa.de/), organised
by [Juliana Barrett](http://www.moving-awareness.com).
I only mention it to point out (and as a note to self) that she was using a portable sound system with pretty good loudpseakers,
the [HK Audio Lucas Nano 600](http://hkaudio.com/products.php?id=414).
A picture from my last morning walk to the office across the Isar:
[![Isar](img/789_500.jpg)](https://flic.kr/s/aHskKjyb4g)