---
post_number: "1069"
title: "DevDay@AU, Chronicle, Extensible Storage Removal with Linked Files, View Depth Override and Sound of Noise"
slug: "devday_au"
author: "Jeremy Tammik"
tags: ['references', 'revit-api', 'rooms', 'views']
source_file: "1069_devday_au.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1069_devday_au.html"
---

### DevDay@AU, Chronicle, Extensible Storage Removal with Linked Files, View Depth Override and Sound of Noise

### DevDay@au Chronicle Estorage View Depth Sound of Noise

Monday was the first day at Autodesk University in Las Vegas, dedicated to the forward-looking confidential DevDays conference with the main focus on the
[rEvolution – where desktop meets the cloud](http://thebuildingcoder.typepad.com/blog/2013/09/back-and-preparing-the-au-and-devdays-conferences.html#2).

In spite of the confidential nature of the information discussed, I am sure I can share an at least an outline of the content here in public with you.

I'll also touch on a couple of other interesting topics that cropped up on the past few days, such as the need to
[disconnect from central before removing an updater reference](#9), the
[Autodesk Chronicle product launch](#10), a
[view depth override macro](#11) showing
a nice example of making use of Chronicle, and last but not least the
[sound of noise](#12).

#### DevDay@AU

The Autodesk Developer Network conference discusses confidential future directions of the products and their APIs.

I'll look in more detail at the following agenda items:

- [General Design](#3)
- [Cloud & mobile platform web service APIs](#4)
- [Afternoon breakout sessions](#5) for AEC, Infrastructure & Manufacturing, Media & Entertainment
- [BIM – Architecture, Engineering, Construction and Infrastructure breakout session](#6)

This exciting day was wrapped up with the
[AppHack contest](http://thebuildingcoder.typepad.com/blog/2013/10/revit-2013-api-developer-guide-pdf.html#4) final
and a social evening event at
[El Segundo Sol](http://www.elsegundosol.com).

#### General design

The warm welcome by Jim Quanci was followed by the keynote presentation by Amy Bunsell, VP AutoCAD Products, on AutoCAD evolution on the desktop and connecting to other platforms.
We further discussed the Autodesk cloud & mobile platforms, the future direction of AutoCAD and its APIs, and the full spectrum of the new Autodesk cloud & mobile platform web service APIs.

#### Cloud & mobile platform web service APIs

Here are some of the different web API related topics we covered:

- Authorisation – [ADN OAuth samples](https://github.com/ADN-DevTech/AutodeskOAuthSamples).
- Federated Identity, e.g. for a customer to sign in to your web service using the Autodesk id.
- Viewing – web viewer demo
- Creation
  - [ReCap](http://recap360-staging.autodesk.com) web API to create 3D models from photos, used e.g. by
    [kubit](http://us.kubit-software.com),
    [BuildIT](http://www.builditsoftware.com) for
    mold comparison and
    [twnkls](http://twnkls.com) for
    elevator design based on photos of stairs. The REST API is in pilot right now.
  - Stephen demonstrated a custom AutoCAD 360 app in an iPad using ReCap to generate a 3D model directly in AutoCAD 360 from photos via ReCap.
  - [GrabCad](https://grabcad.com) hosting AutoCAD 360.
  - [Fusion 360](http://fusion360.autodesk.com) – live on the stage, Jim and Stephen created a rotating link on an axle in one minute and a racing boat catamaran in another ten or so.
- AutoCAD 360 mobile apps: jobshop building, dynamic attribute addition, live collaboration feeds.
- Rendering as a service (RaaS) with a REST API.
- [BIM 360 Glue](http://www.autodesk.com/products/bim-360) showing a stadium assembled from gigabytes of data from multiple CAD sources, navigate, access attributes, etc., cf. the
  [Glue update](http://adndevblog.typepad.com/aec/2013/05/a-new-glue-new-look-and-30-day-free-trial.html) and
  [updated ADN DevBlog samples](http://adndevblog.typepad.com/aec/2013/10/bim-360-glue-api-pilot-and-updated-samples.html),
  now also equipped with the dedicated
  [360 View](http://the360view.typepad.com) technical blog by Manu Venugopal.
- [BIM 360 Field](http://bim360field.com) construction site web based communication and sharing of issues, checklists, tasks, models, photos etc.
- PLM 360 supporting simple processes to quickly and easily build tools for custom reports and administration, with sample code available from the ADN
  [PLM360-API-Samples](https://github.com/ADN-DevTech/PLM360-API-Samples).

The OAuth, ReCap, Fusion 360, PLM 360, BIM 360 Field and Glue APIs are ready to use, and companies are out there in the AU exhibition hall demonstrating products based on them.

#### Afternoon Breakout Sessions

In the afternoon, we split up into three separate breakout sessions for:

- BIM – Architecture, Engineering, Construction and Infrastructure
- Manufacturing
- Media & Entertainment

Us Revit add-in developers are obviously most interested in the first of these.

#### BIM – Architecture, Engineering, Construction and Infrastructure Breakout Session

The BIM and AEC session focused on three main topics:

- The future of Revit and its APIs –
  pushing the frontiers of BIM beyond building design and creating opportunities for increasingly complex models ready for detailing and fabrication.
- BIM for infrastructure platforms and new web services APIs –
  create and use intelligent 3D models to plan, design, build, and manage infrastructure.
  InfraWorks combines 3D modelling and visualisation technologies for desktop, web, and mobile to allow collaboration on infrastructure models in new, exciting and productive ways. Use the new InfraWorks APIs to integrate with your applications and enable fast and easy infrastructure data exchange and interoperability
  in InfraWorks, Civil 3D, Revit and other Autodesk products.
- BIM 360 Platforms and new web service APIs –
  BIM in the cloud provides access to common, up-to-date model information anywhere, any time.
  APIs available now for Autodesk BIM 360 Glue and Autodesk BIM 360 Field, including the new display component API, deep data-sharing between our BIM technologies, roadmap for the future of BIM in the cloud.

Apart from this very full day of forward looking news and visions, here are some further little news items that also piled up in the last few days and I would like to share with you:

#### Disconnect from Central Before Removing Updater Reference

If you subscribe to a dynamic model updater in a BIM and later open the model in an environment lacking the add-in implementing the updater, Revit may display a warning message that may be confusing or disconcerting to the uninitiated.

I explained how to
[remove an updater reference](http://thebuildingcoder.typepad.com/blog/2010/12/vsta-to-stay-and-updater-to-go.html#2) way
back in the Revit 2011 timeframe, including
[automating the removal process](http://thebuildingcoder.typepad.com/blog/2010/12/vsta-to-stay-and-updater-to-go.html#3).

The process described there could be executed in batch mode over all your models, or at least over the ones that you wish to send to customers.

In newer versions of Revit, the updater itself can specify programmatically that no warning should appear if the updater is not found, as explained in the list of
[what's new in the Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2012-api.html):

##### Dynamic Update Framework API changes

There are new settings to flag an updater as optional. Optional updaters will not cause prompting the end user when they are opening a document that was modified by that updater but the updater is not currently registered. Optional updaters should be used only when necessary. By default, updaters are non-optional. New methods introduced to support this change are:

- UpdaterRegistry.SetIsUpdaterOptional(UpdaterId id, bool isOptional)
- UpdaterRegistry.RegisterUpdater(UpdaterId id, bool isOptional)
- UpdaterRegistry.RegisterUpdater(UpdaterId id, Document doc, bool isOptional)

New methods were added to access information about currently registered updaters:

- UpdaterRegistry.GetRegisteredUpdaterInfos(Document doc)
- UpdaterRegistry.GetRegisteredUpdaterInfos()

Revit now disallows any calls to UpdaterRegistry from within the Execute() method of an updater. That means any calls to RegistryUpdater(), AddTrigger(), etc. will now throw an exception. The only method of UpdaterRegistry allowed to be called during execution of an updater is UnregisterUpdater(,) but the updater to be unregistered must be not the one currently being executed.

That will not help you with old or existing models, though.

I tested the very simple and logical steps to
[remove an updater reference](http://thebuildingcoder.typepad.com/blog/2010/12/vsta-to-stay-and-updater-to-go.html#2) on
your sample model and they worked absolutely fine for me:

- Open the model in the user interface.
- Select 'continue working and do not warn me again'.
- Save and close the file.

When I reopen the file the next time, the warning is gone.

By the way, you always should disconnect your files from central before sending them to us for testing.

I see the following messages when opening your file:

- 1\_central\_model\_msg.png
- 2\_missing\_updater.png
- 3\_cannot\_find\_central.png
- 4\_save\_not\_synced.png

After performing the removal described above, the updater warning disappears.
On reopening again, I only see the 'cannot find central' message.

The developer responded:
"I tried it again and it didn’t work.
Then I remembered your comment about detaching from central (sorry about that), did that and tried again.
Then it worked."

Motto: disconnect from central to remove an updater.

#### Autodesk Chronicle Launch

[Chronicle](https://chronicle.autodesk.com) is
a free technology preview that enables capturing, sharing, and learning from software workflows, currently supporting AutoCAD, Revit, and Inventor.

It is being launched now at AU.

Chronicle consists of a recording utility to capture recordings, and a web site that displays the recordings as Chronicles, interactive video tutorials. The videos can be shared publically or set to private, so that they can be used as internal training materials for a private office or classroom. In essence, Chronicle allows software experts to showcase their expertise and allows other users to view and learn from their real-world expert examples.

The Chronicle Recording Utility captures workflows from within Autodesk products. It records a continuous video screen capture and optionally voice narration. It is also records the timing and details of workflow information, such as the commands, tools, settings, and dialog boxes used during the workflow. Captured data is uploaded to the Chronicle website. An author can publish the Chronicle as public or private, so the desired user group can view a video of the workflow. Additionally, the captured workflow events are displayed on an interactive timeline, enhancing the viewing and learning experience.

Check out
[Chronicle](https://chronicle.autodesk.com) and
start sharing your knowledge today!

#### View Depth Override Macro and Chronicle Sample

Just by chance, I happened to notice the
[View Depth Override macro](http://puntorevit.blogspot.it/2013/10/view-depth-override-2014-codice-sorgente.html)
([source](zip/ViewDepthOverride2014.cs))
on the
[Punto Revit](http://puntorevit.blogspot.it) blog by
Paolo Emilio Serra and his more recent
[updated 3D version](http://puntorevit.blogspot.it/2013/11/3d-view-depth-override-update.html)
([source](zip/3DViewDepthOverride-Write-Read_2014.cs))
running
a macro in perspective mode, currently still work in progress, and thought that might be of interest to others as well.
Looks like interesting stuff.

By the way, the demos on those pages are recorded using the preview version of Chronicle, so you can see some nice live examples of that functionality in use.

#### Sound of Noise

My son Christopher has been actively creating and publishing music on SoundCloud as
[Allerdings](https://soundcloud.com/allerdings).

One nice little sample is
[Hydrophonie](https://soundcloud.com/allerdings/hydrophonie),
created from nothing but natural sounds recorded using a glass of water and some sesame seeds:

Christopher now set up his own Internet presence at
[SoundOfNoise](http://soundofnoise.de).

Exciting stuff, and congratulations!