---
post_number: "1548"
title: "Room Instance Purge"
slug: "room_instance_purge"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'revit-api', 'rooms', 'sheets', 'transactions', 'views', 'walls']
source_file: "1548_room_instance_purge.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1548_room_instance_purge.html"
---

### ForgeFader UI, Lookup Builds, Purge & Room Instances
Blogging despite having already exceeded my work quota for the week, but there is so much to share...
- [ForgeFader user interface](#2)
- [RevitLookup builds](#3)
- [Purging types, families and materials](#4)
- [Retrieving all family instances in a room](#5)
Enjoy!
#### ForgeFader User Interface
Yesterday, I mentioned the
new [ForgeFader](https://github.com/jeremytammik/forgefader)
and [RvtFader](https://github.com/jeremytammik/Rvtfader) samples
to calculate and display a 'heat map', i.e. colour gradient, representing the signal loss caused by the distance and number of walls between the signal source and target points spread throughout a building using JavaScript
and [three.js](https://threejs.org) and the Revit API, respectively.
[Cyrille Fauvel](https://twitter.com/FAUVELCyrille) implemented
the shaders displaying the calculated colour gradient texture in the Forge model,
and [Philippe Leefsma‏](https://twitter.com/F3lipek) added a neat user interface to control the attenuation values, grid density, and turn on and off the display of the raycasting rays:
> "Forge Fader" WiFi signal attenuation simulation by [@jeremytammik](https://twitter.com/jeremytammik), Shader by [@FAUVELCyrille](https://twitter.com/FAUVELCyrille), [#ReactJS](https://twitter.com/hashtag/ReactJS?src=hash) UI by [@F3lipek](https://twitter.com/F3lipek) <https://t.co/SVm8MrqG3Y> [pic.twitter.com/jMghiK7NBY](https://t.co/jMghiK7NBY)
>
> — Philippe Leefsma (@F3lipek) [April 6, 2017](https://twitter.com/F3lipek/status/849813897469153281)

You can try it out for yourself in
the [forge-rcdb.autodesk.io Fader sample](http://autode.sk/2o06u3n).
![ForgeFader user interface](img/forgefader_ui.png)
#### RevitLookup Builds
Do you use [RevitLookup](https://github.com/jeremytammik/RevitLookup)?
If not, you might be best off to stop whatever you are doing right now and install and test it first.
You will not regret.
It is simply an interactive Revit BIM database exploration tool to view and navigate element properties and relationships.
Peter Hirn of [Build Informed GmbH](https://www.buildinformed.com) very kindly set up a
public [CI](https://en.wikipedia.org/wiki/Continuous_integration) for RevitLookup
at [lookupbuilds.com](https://lookupbuilds.com)
using [Jenkins](https://jenkins.io/index.html) in
a multi-branch project configuration to build all branches and tags from the GitHub repository.
The output is dual-signed with the Build Informed certificate, zipped and published to an Amazon S3 bucket.
For more information, please refer to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [CI for RevitLookup](https://forums.autodesk.com/t5/revit-api-forum/ci-for-revit-lookup/m-p/6947111).
![RevitLookup builds](img/revitlookup_builds.png)
#### Purging Types, Families and Materials
The Revit API provides no direct access to purge, and this has been high on the add-in developer wish list for a long time, cf. the discussions
on [purging unused text note types](http://thebuildingcoder.typepad.com/blog/2010/11/purge-unused-text-note-types.html)
and [determining purgeable elements](http://thebuildingcoder.typepad.com/blog/2013/03/determining-purgeable-elements.html) in general.
Now, Harry Mattison of [Boost your BIM](https://boostyourbim.wordpress.com) provided clever solutions
to [purge types and families](https://boostyourbim.wordpress.com/2016/10/20/rtceur-api-wish-1-purging-types-and-families)
and [purge unused materials](https://boostyourbim.wordpress.com/2016/10/21/purge-unused-materials-for-another-rtceur-api-wish).
The former needs to handle and roll back if an error occurs due to attempting to delete the last type in a system family.
The latter uses the `DocumentChanged` event, rolling back transaction groups, and other good things do find materials that can be deleted without elements being modified.
It takes a bit of time to run because it is deleting all materials and undoing the deletions that modify other elements.
Many thanks to Harry for implementing and sharing these long-standing wishes!
#### Retrieving All Family Instances in a Room
\*\*Question:\*\* I am interested in knowing the Revit family instances that are contained in a room. Apparently, Revit Rooms are currently not handled by the Revit to Forge translation. For a near term workaround, do you know of a Revit plug-in or script that generates a spreadsheet to report for each room in a model, the family instances contained by the room? I heard the Revit API's support this, but have not seen or verified it. The spreadsheet could be as simple as a three-column table that lists Room ID, Room Name, and Family Instance ID, where the ID is the Revit Element ID.
\*\*Answer:\*\* Sure; look at the [RoomEditoreApp sample](https://github.com/jeremytammik/RoomEditorApp), in the
module [CmdUploadRooms](https://github.com/jeremytammik/RoomEditorApp/blob/master/RoomEditorApp/CmdUploadRooms.cs) implementing
the external command to upload the room data to my cloud database.
The [lines L157-L217](https://github.com/jeremytammik/RoomEditorApp/blob/master/RoomEditorApp/CmdUploadRooms.cs#L157-L217) implement
code to retrieve the furniture and equipment contained in given room.