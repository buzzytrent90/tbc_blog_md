---
post_number: "1285"
title: "Keyboard Shortcuts and Other News"
slug: "keyboard_shortcut"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'views', 'windows']
source_file: "1285_keyboard_shortcut.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1285_keyboard_shortcut.html"
---

### Keyboard Shortcuts and Other News

A couple of questions on stackoverflow and the Revit API discussion forum prompted me to pick up the age-old issue of
[keyboard shortcuts](http://thebuildingcoder.typepad.com/blog/2009/04/addin-keyboard-shortcut.html) again.

Plus, I'd like to point out some news from the AEC Hackathon in New York and the AECbytes newsletter:

- [AEC Hackathon in New York](#2)
- [AECbytes newsletter](#3)
- [How to programmatically retrieve keyboard shortcuts](#4)
- [How to assign a keyboard shortcut to an external command](#5)
- [How to programmatically create a keyboard shortcut](#6)

#### AEC Hackathon in New York

I attended the AEC Hackathon in New York
[last year](http://thebuildingcoder.typepad.com/blog/2014/05/aec-hackathon-from-the-midst-of-the-fray.html),
and one of the numerous results was the vA3C WebGL three.js-based 3D AEC viewer:

- [RvtVa3c – Revit vA3C generic AEC viewer JSON export](http://thebuildingcoder.typepad.com/blog/2014/05/rvtva3c-revit-va3c-generic-aec-viewer-json-export.html)
- [RvtVa3c assembly resolver](http://thebuildingcoder.typepad.com/blog/2014/05/rvtva3c-assembly-resolver.html)
- [Three.js AEC viewer progress on two fronts](http://thebuildingcoder.typepad.com/blog/2014/08/threejs-aec-viewer-progress-on-two-fronts.html)
- [Custom user settings storage and RvtVa3c update](http://thebuildingcoder.typepad.com/blog/2014/10/berlin-hackathon-results-3d-viewer-and-web-news.html#7)
- [3D Viewing and vA3C updates](http://thebuildingcoder.typepad.com/blog/2015/01/3d-viewing-va3c-and-revitlookup-updates.html)

This time around, my colleague Jaime Rosales Duque participated and shared his experiences on the
[AEC Hackathon New York City 2.0](http://adndevblog.typepad.com/aec/2015/02/aec-hackathon-new-york-city-20.html).

For more details on the innovative projects created there, see the
[AEC Hackathon NYC recap](http://2build.wordpress.com/2015/02/03/aec-hackathon-nyc-recap).

#### AECbytes Newsletter

For an overview over a couple of exciting AEC happenings that I have not mentioned previously, I find Lachmi Khemlani's semi-annual
[AECbytes AEC technology newsletter](http://www.aecbytes.com/newsletter/2015/issue_73.html) a
very worthwhile read, mentioning Trimble ProjectSight, AllTrak Cloud and Rapid Positioning System, Newforma SmartUse, Conworld and Aconex Dynamic Manuals.

#### How to Programmatically Retrieve Keyboard Shortcuts

A question was raised on the Revit API discussion forum on
[how to programmatically retrieve keyboard shortcuts](http://forums.autodesk.com/t5/revit-api/how-to-get-short-key-collection/td-p/5513905):

**Question:**
Please suggest how can I get list of Revit short keys programmatically.

**Answer:**
The user interface provides a command to
[export your shortcut keys](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-0A6F242A-EBEF-4B9B-B974-23006DC2E882) to XML.

This command may possibly be launched programmatically as well using
[PostCommand](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.3).

It might be tricky to drive programmatically, though.
Maybe, you can either use the Revit API PostCommand method, the or .NET UI Automation library to drive the following sequence:

- View > Windows > User Interface > Keyboard Shortcuts > Export... > filename input > Save > Cancel

If you use PostCommand, you will have to
[handle all the dialogues](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32) that
pop up via the Revit API.

**Revitalizer** suggests:
Just read this file:

- C:\Users\[YOUR\_USER\_NAME]\AppData\Roaming\Autodesk\Revit\Autodesk Revit 2015\KeyboardShortcuts.xml

As you can see, this is a per-user-file, so there may be several of them on your system.

**Question:**
Is this file always generated automatically, or do you have to launch some Revit command to export it?

Does Revit keep it up to date automatically every time you make a change to the keyboard shortcuts?

**Answer:**
Yes, Revit updates this file just if I change the shortcuts in the GUI.

I don't know if there exists such a file if the user didn't change them yet, but I suppose so since there are not only user defined but also predefined shortcuts, e.g. WF for wireframe display mode.

Further testing shows that the XML file is not present before the use user sets up her first keyboard shortcut.

If it is missing and you use the GUI to make a change to the existing shortcuts, e.g. by adding one, it appears.

Since shortcuts are per user, this information needs to be stored, anyway.

If the user never changed her shortcuts, you can assume that she is using default settings.

To generate a default settings file, you can make a change, retrieve the XML file, and use your newly generated file as a template, minus your change.

Since the KeyboardShortcuts.xml file is localised (CommandName and Paths attributes), one will need to create one template per language.

The CommandId and Shortcuts attributes are language independent.
In a German XML file, for example, there is a default MD for "modify", "ändern" in German.

If you just want the CommandId-Shortcuts relationship, you need only one file version, since this relation is language independent.

#### How to Assign a Keyboard Shortcut to an External Command

Another question on the Revit API discussion forum on
[how to assign a keyboard shortcut to an external command](http://forums.autodesk.com/t5/revit-api/how-to-assign-keyboard-shortcut-to-external-commands/td-p/5511028):

**Question:**
How to assign keyboard shortcuts for external commands in the Revit 2015?

For example, I have created an external command to display the element location and want to execute this commend using a keyboard shortcut "CP".

**Answer:**
Its simple.
Just go to the Revit Menu and click on options at the bottom right of the popup.
Then click on User Interface tab and then select the Keyboard Shortcuts in the middle of the dialog on the right side.
From there you can find your custom command and assign a shortcut to it.
You can also adjust existing shortcuts and make new shortcuts there for most of the commands in Revit.

**Response:**
Thanks for your reply, but I want to do this programmatically.

**Answer:**
Here is an old description on
[how to achieve that back in 2009](http://thebuildingcoder.typepad.com/blog/2009/04/addin-keyboard-shortcut.html).

Things have changed since then, though.

The Revit 2011 API introduced
[keyboard shortcut support for API buttons](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2011-api.html):

#### Keyboard shortcut support for API buttons

API buttons found on the Ribbon can be assigned a keyboard shortcut. Buttons created by applications registered using manifest files now use an id based on their application id and button name to provide a unique identifier for the button. The keyboard shortcut will be maintained even if the order of registration of API applications changes.

This functionality was overhauled in the
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html):

#### Support for Keyboard Shortcuts and Quick Access Toolbar

API commands may now be assigned keyboard shortcuts, and those assignments will be preserved even if add-ins are added, removed or changed in subsequent sessions.

API commands may also be moved to the Quick Access Toolbar, and that assignment will also be preserved even if add-ins are added, removed or changed in subsequent sessions.

Unfortunately, it looks as if the actual assignment still is done manually as described in the Revit help topic on
[Keyboard Shortcuts](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-39D549F2-75EE-4C06-8B6A-3DADE1FBEF59).

You will obviously be facing the same issues as described above driving this programmatically.

#### How to Programmatically Create a Keyboard Shortcut

A query on stackoverflow on how to
[programmatically create a keyboard shortcut in Revit](http://stackoverflow.com/questions/9472227/programmatically-create-a-keyboard-shortcut-in-revit):

**Question:** Can I programmatically create a keyboard shortcut for my Revit add-in?

**Answer:** As explained by the help topics listed above, this can easily be achieved manually.

Programmatically, again, you will have to tackle the issues described above, as far as I know.