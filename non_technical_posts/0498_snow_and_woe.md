---
post_number: "0498"
title: "Snow and Woe with Manifest Files"
slug: "snow_and_woe"
author: "Jeremy Tammik"
tags: ['revit-api', 'schedules', 'views', 'windows']
source_file: "0498_snow_and_woe.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0498_snow_and_woe.html"
---

### Snow and Woe with Manifest Files

I am still on the road toward Paris.
The departure of our flight in Moscow was delayed, so we missed the connection to Paris in Vienna.
In addition, I heard that the Paris airport was closed all day until five in the afternoon due to snow.
We are still booked to fly and arrive Thursday night, though, so the Paris conference scheduled for Friday is not yet endangered.
Right now, the ETA is quarter to twelve, so it will be a short night before the conference begins...

Actually, by the time I get to post this, it is early Friday morning.
We arrived safely in the hotel at two in the morning, slept for a couple of hours, and reached the office, still completely empty at this time of the day.

Before I continue, here are a couple of pictures from the developer conference in Moscow, taken by my colleague Partha Sarkar.
Here is the view from the Autodesk office over snowy Moscow:

![View from the Autodesk office in Moscow](file:////j/photo/jeremy/2010/2010-12-08_devdays_moscow/img_0016.jpg)

A view of the conference opening session by Jim Quanci with one of our translators at work:

![Jim Quanci presenting with translator](file:////j/photo/jeremy/2010/2010-12-08_devdays_moscow/img_0013.jpg)

And here the DevTech group having dinner after a hard day's work and lots of interesting discussions;
from left to right, Marat Mirgaleev, Philippe Leefsma, Adam Nagy, Karl Osti, Jim Quanci, Partha Sarkar, and me:

![DevTech dinner](file:////j/photo/jeremy/2010/2010-12-08_devdays_moscow/img_0028.jpg)

### Updated Revit API Track at Autodesk University 2010

I updated the list of
[Revit API related classes at Autodesk University 2010](http://thebuildingcoder.typepad.com/blog/2010/12/the-revit-api-track-at-au-2010.html),
adding new classes and uploading more materials for direct download.
You really must check out this material, there are several real gems in there!
Yesterday I looked through Iffat Mai's Diary of a Wimpy BIM Manager CP430-1, and I love it!
Looking through all this material is absolutely worthwhile and should keep you occupied for a while...

### Add-in Manifest Woes

In the last couple of days I have talked with several people who had problems installing Revit add-ins, mainly because they ran into issues with the add-in manifest file.
One Russian developer says he has been struggling with these problems for months now.

Please do read and follow the instructions in the developer guide 'Revit 2011 API Developer Guide.pdf' carefully and precisely!
Section 2.2.5 'Create a .addin manifest file' describes the simple case for the Hello World walkthrough, and 3.4 'Add-in Registration' provides the full detailed description.
We already discussed these issues when looking at the
[manifest file in general](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html#1) and specifically for the
[pipe to conduit converter](http://thebuildingcoder.typepad.com/blog/2010/05/pipe-to-conduit-converter.html#3).
An overview of other manifest related topics is given in the discussion of
[network access and ribbon icons](http://thebuildingcoder.typepad.com/blog/2010/08/network-access-to-add-in-manifest-and-icons.html).

Some of the problems people seem to be running into and that are not highlighted in the documentation:

- The ProgramData folder is hidden by default on Windows 7.- The ProgramData folder is **not** "Program Data"; there is **no space** in the folder name.- The encoding of the XML format add-in manifest file must correspond to what you specify in the XML header tag.- The assembly path that you specify for your add-in DLL must be correct.- The full class name consists of the namespace prefix with a point separator and the class name appended to it.

#### ProgramData Folder Issues

Some Windows 7 systems seem to have another folder "Program Data" with a space in it in addition to the ProgramData folder without a space.
Placing the manifest file in a subdirectory under the one with a space will not work!
You can either use the command line cmd.exe to change directories to the correct location, or configure the Windows Explorer not to hide system folders by selecting Organize > Folder and search options > View > Hidden files and folders > Show hidden files, folders, and drives.

**Addendum:** A safer way to ensure you get to the right place is to use the DOS environment variables in the Windows explorer address bar, e.g. type %programdata% for C:\ProgramData Or %Appdata% for C:\Users\<user>\AppData\Roaming.

#### Add-in Manifest File Encoding

Several of the developer guide add-in manifest examples use UTF-8 encoding.
To reuse them as shown, you need to ensure that your editor really does save them in that format.
Alternatively, you can use ANSI encoding, if you have no need for non-ASCII characters, or any other encoding acceptable for XML files.
Whatever you do, please ensure that the encoding used matches the one you specify in the XML encoding attribute.
Several people ran into this issue right in the beginning, when add-in manifest files were first introduced, and I discussed it in detail in the post on
[manifest files and Guidize](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html#3).

#### Assembly Path

Please do not try to type in your assembly path manually.
Use copy and paste instead!
The assembly path is listed in the Visual Studio output window when compilation completes, so you can copy it from there.

#### Full Class Name

In one example, I saw someone entering a file path here...
Go to the source code of your external command class implementation, determine what the namespace and class name is, and concatenate the two using a '.' separator to define the full class name.
Here again, please use copy and paste, do not try to type things in manually.

#### DevTV Add-in Template

Actually, the simplest way to avoid all these problems is to use the Visual Studio
[DevTV add-in template](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html#template_update),
which creates a valid add-in manifest file for you and automatically adds a post-build event to the generated Visual Studio solution which copies it to the right destination folder and also ensures that it remains up to date.