---
post_number: "1045"
title: "RoomEditorApp for Revit 2014 on GitHub"
slug: "roomeditorapp"
author: "Jeremy Tammik"
tags: ['csharp', 'references', 'revit-api', 'rooms', 'windows']
source_file: "1045_roomeditorapp.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1045_roomeditorapp.html"
---

### RoomEditorApp for Revit 2014 on GitHub

I discussed my Revit add-in for my cloud-based, real-time, round-trip, 2D Revit model editing application on any mobile device in depth last spring, with the last update on its
[implementation status](http://thebuildingcoder.typepad.com/blog/2013/05/my-cloud-based-2d-editor-implementation-status.html) published
back in May.

Even at that time, since I was mainly concentrating on the exciting cloud aspects of the application, I had added quite a bit of functionality to the Revit add-in that I never got around to documenting, such as

- External application
- Automatic immediate BIM update from cloud database
- Subscription toggle button

The various aspects that I did already discuss are listed in The Building Coder
[Desktop](http://thebuildingcoder.typepad.com/blog/desktop) category.

By popular demand, I also provided an emergency stop-gap update of the source code for the
[latest Revit 2013 version](http://thebuildingcoder.typepad.com/blog/2013/07/sydney-revit-api-training-and-vacation.html#9) during the
[Sydney Revit API training](http://thebuildingcoder.typepad.com/blog/2013/07/sydney-revit-api-training-and-vacation.html) in July.

I now really have to get going with the preparation of my AU presentations, and one of the unavoidable steps is migrating this application to Revit 2014, which is what I discuss here and now.

#### Clone and Compile DreamSeat

One of the prerequisites for interacting with the cloud database is obviously uploading and querying of its data.

I use CouchDB as the cloud database and the
[DreamSeat .NET library](https://github.com/vdaron/DreamSeat) to
access it from my C# Revit add-in.

To create the library for my add-in to reference, I cloned the DreamSeat GitHub repository to my local system:

```
  git clone https://github.com/vdaron/DreamSeat
```

Since it was last built using a newer version of Visual Studio, I had to edit the two first lines in the DreamSeat.sln solution file to be able to use Visual Studio 2010 instead.
I simply changed them to read:

```
Microsoft Visual Studio Solution File, Format Version 11.00
# Visual Studio 2010
```

No other changes required, the library compiled, and I can add a reference to it to my Revit add-in project.

#### RoomEditorApp Migration from Revit 2013 to Revit 2014

I created a new Visual Studio solution for a new Revit add-in named RoomEditorApp and copied the code from my old Revit 2013 GetLoops add-in into it.

After the flat migration, the compilation completes and just reports
[16 obsolete API usage warnings](zip/room_editor_app_migr_2014_1.txt),
which seems pretty good.

I posted the flat migrated version to the
[GitHub RoomEditorApp repository](https://github.com/jeremytammik/RoomEditorApp) and
created an initial version
[2013.0.0.9](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2013.0.0.9) to
provide a starting point from which to track my changes.

I easily fixed all the warnings, as described in numerous previous discussions, and published version
[2014.0.0.10](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.0.10) compiling
with zero errors and warnings.

Oops, I also need to change my namespace from GetLoops to RoomEditorApp.
This became obvious as soon as I attempted to load the external application in Revit, since it complained about the missing RoomEditorApp.App specified in the add-in manifest, which was still named GetLoops.App.
The namespace is correctly specified in
[version 2014.0.0.11](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.0.11).

Oops again; it next complained on loading into Revit about lacking the embedded icon resources.
Luckily, my external application includes assertions that ensure that the required resources are actually found, so I noticed what the problem was.
Here is finally
[version 2014.0.0.12](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.0.12) with
icons added and displaying the user interface successfully in Revit 2014:

![Room editor add-in user interface in Revit 2014](img/roomedit_2014_ext_app_icons.png)

#### Installing CouchDB

For testing purposes, I prefer a local installation of my database in addition to the cloud-based one.

That obviously also requires a local installation of the CouchDB database system itself as well.

To create that, simply go to
[Apache CouchDB](http://couchdb.apache.org),
click on 'download', install, and restart the computer.

I install it as a service and later go into the computer management and set the service to be started manually instead of automatically, so it is only available when actually needed.

#### Replicate the Database

Since I already have my cloud-based application up and running on the
[Iris Couch](http://www.iriscouch.com) cloud
CouchDB server from my last experiments, I can simply replicate that.

The database base URL is
[jt.iriscouch.com/roomedit](http://jt.iriscouch.com/roomedit),
whereas the entry point to access and test it is its main application file index.html at
[jt.iriscouch.com/roomedit/\_design/roomedit/index.html](http://jt.iriscouch.com/roomedit/_design/roomedit/index.html).

I can replicate it using the CouchDB Futon interface on my local machine installation, replicating to a new local database named 'roomedit' from the cloud-based database instance:

![Replicate database](img/roomedit_2014_replicate_to_local.png)

#### Conclusion

I am up and running now in Windows 7 and Revit 2014, as opposed to my rather obsolete previous installation based on Windows XP and Revit 2013.

I still have a lot of undocumented features that I would love to discuss here in the Revit side of things.

As said, I need to prepare my AU presentation and handout on this topic.

Whereas the Tech Summit presentation that I worked on in the spring focused heavily on the cloud aspects, the AU presentation will be significantly longer and cover both the cloud database and the Revit add-in features.

Therefore, I can document the missing bits and pieces in the coming weeks as I get my AU material ready, with the deadline for handing in the material on November 15 looming on the horizon.