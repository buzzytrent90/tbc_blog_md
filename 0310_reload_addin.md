---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.723394'
original_url: https://thebuildingcoder.typepad.com/blog/0310_reload_addin.html
post_number: '0310'
reading_time_minutes: 6
series: structural
slug: reload_addin
source_file: 0310_reload_addin.htm
tags:
- csharp
- references
- revit-api
- schedules
- structural
title: Reload an Add-In to Debug
word_count: 1248
---

### Reload an Add-In to Debug

I received a very friendly suggestion from
[John Morse](mailto:jmorse@corteksystems.com) of
[CorTek, Inc](http://www.corteksystems.com) to
publish his solution for reloading a Revit add-in into Revit without having to shut down and restart Revit each time.
This can greatly speed up debugging iterations.

In Revit 2008, it was possible to use the Visual Studio 'Edit and Continue' feature to achieve the same thing, but this functionality has been unavailable for Revit plug-ins for some time now, as we already noted a couple of times, for instance in comments from
[Jamie](http://thebuildingcoder.typepad.com/blog/2009/04/installing-the-revit-2010-sdk.html?cid=6a00e553e16897883301156f4a1465970c#comment-6a00e553e16897883301156f4a1465970c) and
[Ed Pitt](http://thebuildingcoder.typepad.com/blog/2008/12/64-bit-revit-api-issues.html?cid=6a00e553e168978833011570d63803970b#comment-6a00e553e168978833011570d63803970b)
([url](http://www.cadsmart.net)),
who also wrote about this topic in his own
[blog](http://revit-programmer.blogspot.com/2009/06/debugging-with-revit-64-bit.html).

There only official work-around that has been suggested for Revit 2009 is to code and debug in VSTA, which allows editing and re-executing code without restarting, and then port to your external command or other plug-in API application.

Another partial work-around that I use sometimes is to run Revit inside of the Visual Studio debugger with my project opened, like normal.
If I really want to edit some of the source files and continue debugging at the same time, I open the same project once again in another instance of Visual Studio and perform the edits there.
However, you must not save the edits immediately, or the first Visual Studio instance will notice that and say that it cannot continue debugging without recompilation.

As also
[mentioned](http://thebuildingcoder.typepad.com/blog/2009/04/installing-the-revit-2010-sdk.html?cid=6a00e553e16897883301156ff87103970c#comment-6a00e553e16897883301156ff87103970c),
the Revit development team have looked at this issue and know how important it is for developers.
It is scheduled to be fixed, but nobody can say exactly when the fix will happen.

Guy Robinson presented a
[solution](http://redbolts.com/blog) to
this problem last June using the VSTA wrappers to reload a plug-in dynamically.
Please be aware that this solution will almost certainly stop working in some future release of Revit.

Now here is John's new solution to this problem:

i came up with a solution that might be beneficial to people that are building/debugging an Add-In for Revit.

i found that when testing/debugging an Add-In, you have to launch and close Revit each time you run your app.
that's a pain.

i came up with a way to dynamically load an app/dll to where you can leave Revit open.

i have a zip of two VS solutions that show this example.
one builds the initial DLL (which is loaded by Revit) and the bridge DLL (which is called by the initial DLL and is used for plugging/reading in external assemblies/dll's). the other
solution is the main form application that gets called/plugged in through the bridge.

here is a RAR file
[reload\_addin.rar](zip/reload_addin.rar) containing the entire solution.

it includes a README file that explains everything.
here is what it says:

---

This solution was built in VS 2005.

Pay attention to paths that are referenced anywhere and everywhere.
This includes the references to the Revit.dll, which I have included in a directory under this solution;
however, you can reference your own path.
I built this solution under the C:\Temp directory.

Under the Revit directory there are two solution directories, one API directory and two files.

##### Contents

- (Directory) Solution: RevitPlugin
  - Project: MainDLL – The main entry point for our add-in to Revit (see INI file entry below).- Project: Bridge – Called by the MainDLL to process a specific DLL that implement our IRevit
      Interface and invoke methods within that DLL. This can be made to systematically
      load multiple DLL's, etc...- (Directory) Solution MainForm:
    - Project: MainForm – This is the DLL that's loaded into the byte array from the Bridge DLL.
      It's the Main Form that get's loaded and the main GUI that gets launched when
      the RibbonPanel button gets clicked.- (Directory) Revit.API – Contains the RevitAPI.dll- (File) Revit.ini – The entry for the Revit.ini file. (Modify the count and reference as needed)- (File) README.txt – This file.

So there are two solutions: RevitPlugin and MainForm.
Once the two projects in the RevitPlugin solution are built and in place, you shouldn't have to rebuild them.
The MainForm solution is the one that you continue to rebuild as you develop and debug.

##### Steps

For a seamless test/run, uncrunch the RAR file to where the Revit directory is under the C:\Temp directory, i.e., C:\Temp\Revit.

1. Modify the Revit.ini file to accommodate the MainDLL.- Build the RevitPlugin solution in a separate instance of VS.- Build the MainForm solution in a different instance of VS.- Launch Revit by itself or by debug in VS from step 2.- If you can click and successfully run the app from the Add-Ins ribbon panel, then you've correctly set everything up the right way.- Now close the MainForm app that was launched from the Add-Ins ribbon button, but LEAVE REVIT OPENED.- Go back to the MainForm solution/project in VS (step 3) and modify as needed. Rebuild the solution.- Launch again from the Add-Ins ribbon button to experience the changes.

##### Advantage

Independently build the MainForm solution/project and test it without having to close and reopen Revit every time.

##### Disadvantage

MainForm cannot be debugged in Visual Studio due to the fact that its DLL is loaded into a byte array from the Bridge DLL.

You can contact me here at the email address given above.
However, please do not call CorTek, Inc asking for me.
Thanks...

---

Thank you very much John for this great solution!

For more information on the implementation details, please have a look at the commented source code.
The main trick to avoid locking the DLL to disk when it has been loaded is to call File.ReadAllBytes to read the DLL contents into memory, and then load the assembly using the Assembly.Load method passing in the memory block instead of the physical assembly filename.

I am sure it will be of great use to many other Revit developers, and hope that it will work with future versions of the Revit API as well, until we someday get the 'Edit and Continue' feature properly and officially supported again.

By the way, in this sample, the RevitAPI.dll assembly is referenced with the 'Copy Local' flag set to true, and that is also why it is included in its own directory in the archive file.
This is normally not recommended, because it can cause issues, especially when trying to debug in Visual Studio.
As mentioned above, however, the approach taken, loading the assembly from a byte array in memory instead of from the assembly file on disk will cause problems for debugging anyway.
This may not be the best solution, per se, but it works for John because it is more important for him to avoid relaunching Revit every single time to debug, and you can dump values to the console or GUI when running the solution to see what's going on in the code.