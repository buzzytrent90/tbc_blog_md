---
post_number: "0369"
title: "RevitAddInUtility for 32 and 64 Bit Systems"
slug: "addinutility_32_64_bit"
author: "Jeremy Tammik"
tags: ['csharp', 'references', 'revit-api', 'windows']
source_file: "0369_addinutility_32_64_bit.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0369_addinutility_32_64_bit.html"
---

### RevitAddInUtility for 32 and 64 Bit Systems

We already discussed the
[RevitAddInUtility](http://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html),
a stand-alone .NET assembly providing its own API for manipulating add-in manifest files and determining Revit installation information.

People are starting to make serious use of it now, which led to the following question that popped up several times in the last couple of days, among others in this
[comment](http://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html?cid=6a00e553e168978833013480d22315970c#comment-6a00e553e168978833013480d22315970c) by Joel, saying:

**Question:** I am trying to create a single installer for both 32 bit and 64 bit but it seems that RevitAddInUtility.dll is compiled for x64 in the 64 bit install and x86 in the 32 bit install.

Is there an 'Any CPU' compilation?

Or am I just missing something?

[Rodney Howarth](http://roddotnet.blogspot.com) ran into the same issue and says:

I'm having some problems trying to use RevitAddInUtility.
If I make the tool, compile my installer for any cpu, and run it on my win7 64 bit machine, it works great.

However if I then copy it to a XP 32 bit machine, it fails saying that it 'could not load file or assembly revitaddinutility, version=2011.0.0.0, culture=neutral, publickeytoken=null' or one of its dependencies. An attempt was made to load a program with an incorrect format.

I'm guessing it is having problems because I got that version of the RevitAddInUtility assembly file from my Revit directory, which is a 64 bit install, and deployed it to a 32 bit machine.

I can get around this by copying the DLL from a 32 bit machine and making a 32 bit packaged installer and a 64 bit one, but is this really what I have to do? Is there a way to just create a universal installer with the one DLL?

**Answer:** You are correct, there are separate versions for 32 and 64 bit systems.
It would theoretically be possible to set the platform target of the RevitAddInUtility assembly to any cpu (clr/safe), so that it can run on both x86 and x64, but this requires it to not depend on any Windows API.
Unfortunately, it uses the Windows API to query the registry.
We will see whether we can replace this part in the future to solve this issue.

Meanwhile, the way Rod ended up getting around this was:

- I got someone on 32 bit to send me their DLL. It would be great if this was hosted somewhere.- I referenced that in my application rather than the 64 bit version.- I set my application to compile in x86 mode only.

So he essentially only has a 32 bit version of the installer, which seems to work on win64bit as well, as long as you don't have any other 64 bit DLL dependencies.

Here is the
[32 bit version of RevitAddInUtility.dll](zip/RevitAddInUtility_dll_32_bit.zip)
in case you are working on a 64 bit system and would like to make use of this workaround.

Joel implemented a different solution. He says: I was able to create a simple console app to resolve this issue for a single installer.

Basically, I installed both the 32 and 64 bit versions of RevitAddInUtility.dll each in their own folder.
Then, based on OS bit version, I copied the correct RevitAddInUtility.dll build from the folder into the same directory as the assembly that was referencing RevitAddInUtility.dll.

Works well in all our tests.

... later: note the additional useful information added in the
[comments](http://thebuildingcoder.typepad.com/blog/2010/05/revitaddinutility-for-32-and-64-bit-systems.html#comments)
by Dave Echols on the availability of both versions of the DLL in every copy of the Revit installation files, and from Guy Robinson on
[using interfaces](http://www.redbolts.com/blog/post/2010/05/20/RevitAddinUtility-x32x64.aspx) to
solve the issue, by attempting too load the DLL yourself using CreateInstanceAndUnwrap and catching the exception thrown if the version does not match you system, in which case you can simply try again with the other version.
This enables you to utilise the appropriate assembly while maintaining a single main application and no file copying.

Very many thanks to everybody for all the ideas you provided, Joel, Rod, Dave and Guy!