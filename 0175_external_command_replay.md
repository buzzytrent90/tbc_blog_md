---
post_number: "0175"
title: "External Command Replay"
slug: "external_command_replay"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'views', 'windows']
source_file: "0175_external_command_replay.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0175_external_command_replay.html"
---

### External Command Replay

[Matt Mason](http://cadappdev.blogspot.com)
of
[Avatech Solutions](http://www.avatechsolutions.com)
pointed out a few important items related to the preceding post on
[replaying a journal file](http://thebuildingcoder.typepad.com/blog/2009/07/journal-file-replay.html):

One topic that you did not cover – using the journal file with Revit external commands.

Here are some things that I've noticed in the past:

- If your external command has dialog boxes, your application will freeze if Revit is being driven by a journal.- The alternative to this is to use the JournalData StringStringMap in the ExternalCommandData which is passed to you.
    - You can read values from the StringStringMap and use them instead of your dialog boxes.- You can write the values at the end of your command – based on what the user selected – so that your existing commands can be played back.- When last I looked at it (2009?) there were problems playing back your command if it included calls to select elements using PickOne or WindowSelect.

Thank you Matt for these valuable hints!

### Solar Radiation Technology Preview

A
[Solar Radiation Technology Preview](http://labs.autodesk.com/utilities/ecotect)
for Revit is now available on the Autodesk Labs site.
Scott Sheppard provides some additional
[background information](http://labs.blogs.com/its_alive_in_the_lab/2009/07/autodesk-revit-solar-radiation-technology-preview-now-available-1.html) on this as well.