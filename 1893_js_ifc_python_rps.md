---
post_number: "1893"
title: "Js Ifc Python Rps"
slug: "js_ifc_python_rps"
author: "Jeremy Tammik"
tags: ['csharp', 'geometry', 'levels', 'python', 'revit-api', 'sheets', 'views', 'windows']
source_file: "1893_js_ifc_python_rps.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1893_js_ifc_python_rps.html"
---

### Addin File, Learning Python and IFC.js
Today, we look at the add-in manifest, learning Python and Dynamo, the status of the Revit Python shell and a useful stand-alone IFC viewer:
- [Personalised add-in manifest](#2)
- [Addin manifest using a relative path](#2.1)
- [Learning Python and Dynamo](#3)
- [Jacob's Dynamo learning resources](#3.1)
- [Take Dynamo further using Python](#3.2)
- [Quo vadis, RevitPythonShell?](#4)
- [IFC.js](#5)
#### Personalised Add-In Manifest
Andrea Tassera of [Woods Bagot](https://www.woodsbagot.com) raised an interesting question in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [using relative paths (`%appdata%`) in `.addin` file](https://forums.autodesk.com/t5/revit-api-forum/use-relative-paths-appdata-in-addin-file/m-p/10074984):
\*\*Question:\*\* Is it possible to use a relative path, such as `%appdata%` in the `Assembly` section of the addin file?
I am developing this plugin that will be deployed to other people in the company, and the DLLs will live somewhere in C:\Users\AppData\Roaming\NameOfFolder\... so writing the explicit path with the user name is not a real option. Normally, in Windows, you would use `%appdata%`, but it doesn't seem to be working in the .addin.
Is there a way?
This is what I tried but isn't working:

```
</spanxml version="1.0" encoding="utf-8"?>
<RevitAddIns>
 <AddIn Type="Application">
       <Name>Wb.ModelEstablishment.RevitName>
       <FullClassName>Wb.ModelEstablishment.Revit.RibbonFullClassName>
       <Text>Wb.ModelEstablishment.RevitText>
       <Description>Model EstablishmentDescription>
       <VisibilityMode>AlwaysVisibleVisibilityMode>
       <Assembly>%appdata%\folder\subfolder\Wb.ModelEstablishment.Revit.dllAssembly>
       <AddInId>d06838e1-44e3-4c05-b9f1-f79ca101075cAddInId>
    <VendorId>WBVendorId>
    <VendorDescription>Woods BagotVendorDescription>
 AddIn>
RevitAddIns>
```

But, if I use the explicit path:

```
  <Assembly>C:\Users\sydata\AppData\Roaming\folder\subfolder\Wb.ModelEstablishment.Revit.dllAssembly>
```

... it works fine.
\*\*Answer:\*\* Yes, it is definitely possible to use relative paths in the add-in manifest.
However, `%appdata%` is not a relative path.
That is a variable in an MS-DOS or Windows batch file, or possibly nowadays in a PowerShell script or something suchlike.
Revit add-in manifest files do not support variables, neither MS-DOS nor Windows nor Unix nor any other flavour.
You can read about what is and is not supported in manifest files in the Revit online help section
on [add-in registration](https://help.autodesk.com/view/RVT/2021/ENU/?guid=Revit_API_Revit_API_Developers_Guide_Introduction_Add_In_Integration_Add_in_Registration_html).
Look there for 'In a non-user-specific location in "application data"'.
\*\*Response:\*\* I have read the page you sent, but the whole 'user-specific' or 'non-user specific "application data"' talks about where to save the `.addin` file, not how to point to dlls placed in the `%appdata%` folder through the `Assembly` XML tag inside the `.addin` file.
The .addin file can be in the user folder:
- C:\Users\AppData\Roaming\Autodesk\Revit\Addins\...
That's not a problem.
The problem is that the `Assembly` tag needs to point to a roaming folder:
- C:\Users\AppData\Roaming\...
I don't know how to achieve that without using the absolute path.
I can't write one specific addin file for every user in the office.
Does it make sense?
\*\*Answer:\*\* Yes, absolutely, it makes perfect sense.
As said, the `%appdata%` variable that you are referring to is a Windows specific variable that is not understood or supported by the add-in manifest file.
One simple option I see for you is to (automatically) generate a user-specific add-in manifest for each user and place each one in the appropriate user-specific location.
Another possible approach would be to place one single add-in manifest file for all users into the system-wide global location, and then use other run-time criteria to determine whether or not to make individual add-in functionality and components available to each user on a case-by-case basis, e.g., using the Revit API `AvailabilityClassName`.
Oh, and another thing, much simpler:
I almost never use the add-in assembly DLL path at all.
I just place the add-in assembly DLL in the same location as the add-in manifest `\*.addin` file, and then it is found without specifying any path at all.
If you are already placing the add-in manifest file in the user-specific location, why don't you just put the DLL in the same place?
\*\*Response:\*\* Thanks for clarifying. I get what you mean now!
Unfortunately, the DLLs are in the `%appdata%` folder because they come through `pyRevit`, and the repository gets pulled under:
- %appdata%\pyRevit\Extensions\...
Usually, pyRevit doesn't need addin files, but this is a linkbutton, because I need things to happen at application level (like registering a dockable panel), so it needs to have an addin file to load the DLLs when the Revit application is loading.
It's a bit more complex than usual.
We could maybe write a PowerShell script that moves those DLLs somewhere else, but it seems a bit convoluted.
Creating an automatic generator of the addin sounds interesting though.
From the docs you posted, I don't understand where that code would live though.
It says that "It is intended for use from product installers and scripts".
Does that mean something like a PowerShell script?
\*\*Answer:\*\* You mean the RevitAddInUtility.dll?
The answer is Yes, cf. The Building Coder article
on [RevitAddInUtility](https://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html).
#### Addin Manifest Using a Relative Path
Jason Masters added a more direct and down-to-earth solution:
I believe the answer you're looking for is to just use the relative path modifier for moving up a folder level `..\`.
There's no need to run a script at install just to get to a user's `appdata` folder.
For instance, if I put my add-in DLLs under a different roaming folder like so:
![Add-in manifest relative path](img/addin_manifest_relative_path_1.png "Add-in manifest relative path")
Then, to access those DLL files from my add-in, I can simply use `..\` to navigate up to my roaming folder;
in this case, I'm navigating up 4 levels:
![Add-in manifest relative path](img/addin_manifest_relative_path_2.png "Add-in manifest relative path")
When I launch Revit, you'll see that it's parsed the path correctly and will load the add-in:
![Add-in manifest relative path](img/addin_manifest_relative_path_3.png "Add-in manifest relative path")

![Add-in manifest relative path](img/addin_manifest_relative_path_4.png "Add-in manifest relative path")
Also, `%appdata%` is the specific notation that `cmd.exe` and other Windows utilities use to refer to the `appdata` environment variable, but environment variables are in no way Windows specific and are commonly used on all operating systems.
Revit really should support the use of environment variables in these addin files.
#### Learning Python and Dynamo
Next, let's look at the recurring question of beginners materials for Revit API using Python from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on a [best book for Python for Revit and Dynamo](https://forums.autodesk.com/t5/revit-api-forum/books-or-other-sources-to-learn-python-to-be-used-in-revit/m-p/10063424):
\*\*Question:\*\* I'm starting to learn Python (and Dynamo) and encounter the problem of how to learn to use it for my actual needs in Revit.
I got a whole bunch of Python books from the library.
One problem is the books are either focused on examples to make simple videogames, or are way to advanced (and also not based on examples one would use in Revit).
I have to admit, some children-oriented books were easier to understand :-)
There is
a [Dynamo for Revit: Python Scripting](https://books.google.com/books/about/Dynamo_for_Revit_Python_Scripting.html?id=ROYkygEACAAJ) book,
but that is not currently available.
Are there actually books or sources that would teach Python on examples one would use for Revit/Dynamo?
I'm learning best on an example I actually use.
I also found there is a Revit Python Shell (RPS) on GitHub, but none for Revit 2021.
Is there something similar?
I saw some videos of how one can do some useful things in Revit.
\*\*Answer:\*\* The best and most up to date sources tend to be online these days.
The second source is for RPS as you mentioned but would probably be my starting point for something specific to Revit.
- [IntroductoryBooks – Python Wiki](https://wiki.python.org/moin/IntroductoryBooks)
- [Introduction – Scripting Autodesk Revit with RevitPythonShell (gitbooks.io)](https://daren-thomas.gitbooks.io/scripting-autodesk-revit-with-revitpythonshell/content)
People seem really passionate about Python, don't they.
I'm no expert on this so maybe people have good book suggestions, but I prefer online for all aspects of learning generally.
\*\*Answer:\*\* The “book” you refer to is actually
a [video tutorial by Jeremy Graham on LinkedIn](https://www.lynda.com/course-tutorials/Dynamo-Revit-Python-Scripting-REVISION/779745-2.html).
Probably one of the best ways to get started.
\*\*Answer:\*\* To start with Python and Dynamo, I think this Autodesk University Class
on [Untangling Python: A Crash Course on Dynamo‘s Python Node](https://www.autodesk.com/autodesk-university/class/Untangling-Python-Crash-Course-Dynamos-Python-Node-2017) by
Gui Talarico is a great resource.
If you have some basis but you don´t know how to move on, a good way to learn Python applying to Dynamo is to download **3rd party packages** (Archilab, Clockwork, Lunchbox, Rhythm, Bimorph, Spring nodes, etc.) and read the code that the nodes contain. You will see really good implementations and good practices. You will be able to see how others think about problems and write the solutions.
Have a good understanding of **Object-oriented programming** will help you on this and is a key aspect to understand how to use the Revit API with Python.
About the resources, as is mentioned in the previous posts, I would take a look on LinkedIn learning and also at [thinkparametric.com](https://thinkparametric.com).
There are good courses there.
Good luck!
\*\*Answer:\*\* RevitPythonShell for Revit 2020 works just fine in Revit 2021 as well, cf. [issue #106](https://github.com/architecture-building-systems/revitpythonshell/issues/106).
\*\*Response:\*\* Thanks all.
I'll digest what was offered.
So far the \*Automate the Boring Stuff with Python\* book seems to be the best I could find in the library.
So, I guess I will look more and start to dig deeper when I create some dynamo programs.
I won't be able to install the Python shell in Revit.
Since that is free, they don't really provide the security certificates our IT requires to allow installation.
My hope is Revit will include it at some point (since Autodesk is already approved by IT).
Until then I have to limit myself to the Python in dynamo nodes.
\*\*Answer:\*\* The vast majority API support examples are in C# but Python is sexy.
\*\*Answer:\*\* If you have Revit installed, Python is included: it is supported by the built-in Revit macro IDE (integrated development environment).
\*\*Response:\*\* You mean the macro manager?
I haven't done anything with that yet.
So, if that does what the Python shell does, I use what I already have.
![Macro manager for Python](img/macro_manager_python.png "Macro manager for Python")
\*\*Answer:\*\* The macro manager does not quite do the same thing as RPS.
RPS provides a [REPL, read-evaluate-print loop](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop).
That enables you to interactively program something, line by line, and execute each line as you type it.
The macro environment enables you to create and execute macros.
That means that you have to complete an entire macro and compile it before it can be executed.
In any case, the simplest way forward for you would be to use what you have and work through a Revit macro tutorial first of all.
\*\*Response:\*\* Thanks for the advice. I'll look into that.
I have a (growing) list of features or solutions I need in Revit and will try to work on solutions. Some will require some general Revit trickery, some dynamo, some macros etc. that way I will learn to sue the tools inc. Python.
\*\*Answer:\*\* Oh yes, we mentioned another book just two months ago,
[Más Allá de Dynamo – Beyond Dynamo](),
a Spanish-Language book by Kevin Himmelreich and the first Python manual focused on Dynamo and the Revit API.
Even though it is in Spanish, you can understand most.
It includes a bunch of examples and pretty much everything about the Revit API is covered in Python.
#### Jacob's Dynamo Learning Resources
My colleague Jacob Small reacted to the above and adds:
> Just saw your latest blog post, and thought I'd share a few
more [Dynamo and Dynamo Python educational resources (PDF)](zip/Dynamo_Learning_Resources.pdf),
in case they help. Feel free to share around as needed... Thanks as always for the great content on the blog – they're on my 'always read' list as, even if it's not a need today, they usually come up eventually. :-)
- Dynamo Info / News
- Main: [http://dynamobim.org](http://dynamobim.org/)
- Blog: [https://dynamobim.org/blog](https://dynamobim.org/blog/)
- Dynamo Builds: [http://dynamobuilds.com](http://dynamobuilds.com/)
- Dynamo GitHub: [https://github.com/DynamoDS/Dynamo](https://github.com/DynamoDS/Dynamo)
- Dynamo Learning
- Dynamo Primer: [http://primer.dynamobim.org](http://primer.dynamobim.org)
- Dynamo Forums: [https://forum.dynamobim.com](https://forum.dynamobim.com/)
- Dynamo Dictionary: [https://dictionary.dynamobim.com](https://dictionary.dynamobim.com)
- Dynamo Nodes: [https://dynamonodes.com](https://dynamonodes.com/)
- Design Script:
- Design Script Language Summary: [http://designscript.io/DesignScript_user_manual_0.1.pdf](http://designscript.io/DesignScript_user_manual_0.1.pdf)
- Design Script Language Guide: [https://dynamobim.org/wp-content/links/DesignScriptGuide.pdf](https://dynamobim.org/wp-content/links/DesignScriptGuide.pdf)
- Design Script Presentation: [https://github.com/Amoursol/dynamoDesignScript](https://github.com/Amoursol/dynamoDesignScript)
- Dynamo Python:
- Python for Dyanmo AU Lab Handout: [https://github.com/Amoursol/dynamoPython/blob/master/images/DivingDeeper_ABeginnersLookAtPythonInDynamo_AU_London2018.pdf](https://github.com/Amoursol/dynamoPython/blob/master/images/DivingDeeper_ABeginnersLookAtPythonInDynamo_AU_London2018.pdf)
- Python for Dynamo examples: [https://github.com/Amoursol/dynamoPython](https://github.com/Amoursol/dynamoPython)
- Dynamo Python Primer: [https://dynamopythonprimer.gitbook.io/dynamo-python-primer](https://dynamopythonprimer.gitbook.io/dynamo-python-primer/)
- Revit API
- Revit API Docs: [https://www.revitapidocs.com](https://www.revitapidocs.com/)
- Building Coder: [https://thebuildingcoder.typepad.com/](https://thebuildingcoder.typepad.com)
- Generative Design:
- Info: [https://www.autodesk.com/campaigns/refinery-beta](https://www.autodesk.com/campaigns/refinery-beta)
- Primer: [https://www.generativedesign.org](https://www.generativedesign.org/)
- Beta Site: [https://feedback.autodesk.com/key/RefineryLanding](https://feedback.autodesk.com/key/RefineryLanding)
Many thanks to Jacob for this useful collection!
#### Take Dynamo Further Using Python
Wayne Patrick [@waynepdalton](https://twitter.com/waynepdalton) Dalton adds:
> Hello Jeremy, not sure if this was mentioned in the section on Dynamo and Python (don't think it was?)
> [@Oliver_E_Green](https://twitter.com/Oliver_E_Green) put
together a great resource for [@DynamoBIM](https://twitter.com/DynamoBIM)
and [#Python](https://twitter.com/hashtag/Python)...
thought it might be helpful to flag it for those wanting to learn:
> [Take Dynamo Further](https://dynamopythonprimer.gitbook.io) –
using Python will take your Dynamo definitions to the next level
Thank you, Wayne, for the helpful pointer!
#### Quo Vadis, RevitPythonShell?
I just happened to hear
that [RevitPythonShell](https://github.com/architecture-building-systems/revitpythonshell) is
no longer actively supported by its creator and hitherto maintainer Daren Thomas, cf. our conversation
on [issue #111](https://github.com/architecture-building-systems/revitpythonshell/issues/111).
Says Daren:
> As of next month, I will not have access to Revit anymore, and the project will need a new maintainer.
I am very sorry to hear that, and ever so grateful to Daren for creating RPS in the first place and maintaining it for so long.
RevitPythonShell was the first interactive REPL for Revit, followed by the sexier but less
long-lived [RevitRubyShell](https://github.com/hakonhc/RevitRubyShell).
In case of interest, please vote for [pyRevit issue #1161 – pyRevit Python Shell?](https://github.com/eirannejad/pyRevit/issues/1161)
#### IFC.js
From Python, let's turn to JavaScript and note the very interesting
project [IFC.js](https://github.com/agviegas/IFC.js),
described by [Antonio González Viegas on aechive.net](https://www.aechive.net/agviegas/ifc-js-em4):
> an open source IFC web viewer.
It is fully built on JavaScript and Three.js, and everything is done client-side...
anyone with a browser can navigate IFC files (geometry and information) without depending on native applications...
advantages...
> - Scalability: the non-dependence of connection with a remote service to process an IFC means that there could be thousands of users visualising IFCs simultaneously with no processing cost, since each user supplies her own computational power to visualize her model.
- Flexibility and ease of use: the library enables developers to operate without having to mount an API with HTTP calls. A clear example of this flexibility is having been able to deploy the entire application on github pages, creating a basic IFC viewer compatible with any modern device.
- No internet connection necessary: as pure JavaScript, you can create desktop or mobile apps with React Native or Electron that enable offline IFC viewing