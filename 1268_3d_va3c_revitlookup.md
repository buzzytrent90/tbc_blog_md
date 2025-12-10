---
post_number: "1268"
title: "3D Viewing, vA3C and RevitLookup Updates"
slug: "3d_va3c_revitlookup"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'revit-api', 'views', 'walls', 'windows']
source_file: "1268_3d_va3c_revitlookup.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1268_3d_va3c_revitlookup.html"
---

### 3D Viewing, vA3C and RevitLookup Updates

Here are some GitHub updates on the vA3C and RevitLookup projects, and 3D viewing in general:

- [vA3C project going full steam ahead](#2)
- [vA3C hacker R1 is up](#3)
- [Rendering an element in 3D](#4)
- [Small RevitLookup update](#5)

#### vA3C Project Going Full Steam Ahead

The WebGL and three.js based AEC 3D viewer project
[vA3C](http://va3c.github.io) is going ahead at full steam, driven mainly by Theo Armour.

vA3C is a free, open source viewer and navigator for a variety of file types.

The most recent addition is a bunch of sample STL files received from NASA.

The models are viewable using the built-in GitHub STL viewer or the vA3C viewers.
In either case, you may zoom, pan and rotate the models with your pointing device:

- [vA3C viewer](http://va3c.github.io/nasa-samples) (loads a random model at open, so wait a bit before loading another one).
- [GitHub STL viewer](https://github.com/va3c/nasa-samples/tree/master/stl) (click on any file name to view).

Here is Theo's exuberant status report from November last year:

#### vA3C Hacker R1 is up

- [Live](http://va3c.github.io/viewer/va3c-hacker/latest)
- [Code](https://github.com/va3c/viewer/tree/gh-pages/va3c-hacker)
- [Readme](http://va3c.github.io/viewer/va3c-hacker)

Overview:

- The code screams, "steal me!"
- The user interface is written in Markdown.
- The content potential is that place where the map is bigger than the territory.
- Yes, I do suffer from an over-abundance of exuberance: nonetheless I'm exuberant.

I feel like the little boy that cries "wolf!" too many times, except that I scream "yippee!"

More vA3C cool things:

- There's almost no code. And when there is code, there's no mumbo-jumbo.
- A new shoulder to stand upon has been re-discovered: [Three.js Inspector](http://zz85.github.io/zz85-bookmarklets/threelabs.html).
- There's nothing it can't do

It is, however, release 1, so some of the first attempts at functions can bring your computer to its knees.
Do give it 20 to 30 seconds.
If it works for you, note: camera moving, objects moving, load and dispose shaded objects on demand, text-to-speech and more.

And like everything vA3C, Hacker is:

- [FOSS](https://en.wikipedia.org/wiki/Free_and_open-source_software)
- Runs from any free static web page server or locally
- Simple code like JavaScript or C# that can talk to any library, app or SaaS

Your tasks: Click [vA3C Hacker](http://va3c.github.io/viewer/va3c-hacker/latest).
See if your computer survives the adventure.
Report results.

#### Rendering an Element in 3D

The topic of vA3C and other 3D viewers is vaguely related with the following Revit API discussion forum thread on
[how to render families in 3D – ObjectViewer example used as basis](http://forums.autodesk.com/t5/revit-api/how-to-render-families-in-3d-objectviewer-example-used-as-basis/m-p/5468847):

**Question:** I was wondering if anyone could point me in the right direction. I looked at the ObjectViewer sample in the SDK, and incorporated the viewer for objects there into my project, but I have two questions.

1. Can it be used to render previews that are of "better" quality than wireframe?
2. The demo only accepts one object, I tried with a window FamilyInstance and that didn't render properly, only came out grey; I tried with a wall and that looked OK, and I could move it about.

What I am trying to do is previewing/rendering a view of a family instance.

Any thoughts, suggestions?

**Answer:** Since you have the ElementViewer and ObjectViewer source code included in the Revit SDK, you can basically answer these questions yourself.

For your convenience, I'll state that both of those samples make use of an extremely simple little custom renderer that only draws lines.

You would have to replace that and also implement hidden line removal and a bunch of other stuff to do anything more complex.

I would not recommend using this at all for any professional application.

Its main use is to understand and debug the geometry traversal and associated transformations.

There are tons of better rendering options around.

To start with, you might be able to make use of the
[Revit API PreviewControl](http://thebuildingcoder.typepad.com/blog/2013/09/appstore-advice-and-zooming-in-a-preview-control.html#4).

For more advanced rendering interaction outside of Revit, there is also the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46).

We also implemented a simpler WebGL and three.js based viewer at the AEC Hackathon last year, basically a simpler version of the Autodesk View and Data API, called
[vA3C](http://thebuildingcoder.typepad.com/blog/2014/08/threejs-aec-viewer-progress-on-two-fronts.html).

These are just three possibilities out of many.

**Response:** Hi Jeremy and thank you for pointing me in the right direction.

After trying out several things, I figured that the external web viewer is overkill for my needs and the ObjectViewer example as you stated isn't suited for my needs.

I managed to display a 3D view of selected objects with the PreviewControl, but I don't seem to get the perspective I want or the current perspective in Revit.

I was wondering if there was a simpler way to achieve the preview window you get when you try to export in GbXML format?

This is what I'm aiming for, because I'm adding properties to families and the user has to be able to name those properties from visual cues seen in the 3D preview of the family.

And since I'm writing an add-in using Winforms I would rather not use a webpage.

**Answer:** I think maybe you should play around a bit with the various WebGL solutions after all.

Once you have gotten the hang of them, you may discover that they nowadays provide the handiest, most flexible, versatile and low footprint solution of all.

Regarding Winforms versus web pages: it is in fact possible to host an own web browser inside your own Windows form, e.g. using the embedded Chrome library.

It would be easier to just use the pre-installed browser, though.

Are you aware of any computer system of interest to your users lacking a built-in browser?

Regarding how to set up the preview control view: the PreviewControl does not take any input whatsoever beyond what you supply to its constructor.
In the constructor, you specify the Revit view to be displayed.
However, you can create a dedicated Revit view for that specific purpose and set it up using all the Revit API view control functionality.

That should really provide all you need.

I hope this helps.

#### Small RevitLookup Update

I just merged a small update to RevitLookup submitted by Victor, who added CategoryType to the CategoryCollector method, creating a new
[release 2015.0.0.5](https://github.com/jeremytammik/RevitLookup/releases/tag/2015.0.0.5).

Here is a recent question that came up on this as a
[comment](http://thebuildingcoder.typepad.com/blog/2014/04/revitlookup-for-revit-2015.html?cid=6a00e553e16897883301bb07d8e96a970d#comment-6a00e553e16897883301bb07d8e96a970d):

**Question:** Why is it that RevitLookup is not longer included in the Revit SDK, now that a lot of developers use it?

I tried to implement RevitLookup but without any success. I followed Troy Gates
[Revit Lookup 2015 Addin](http://revitcoaster.blogspot.dk/2014/06/revit-lookup-2015-addin.html) instructions.

That resulted in an error when launching Revit 2015 saying:

```
  Revit cannot run the external application "RevitLookup"
  System.IO.FileLoadExecption
  Could not load file or assembly
  ...\RevitLookup.dll or one of its dependencies.
  Operation is not supported.
  (Exception from HRESULT: 0x80131515)
```

I also tried implementing your repository along with the files from Troy and still without any success.
Can you give my any advice on how to fix this?

**Answer:** RevitLookup is no longer included in the Revit SDK for the simple reason to make it easier for all users of it to add their improvements to it.

It is now hosted in its own
[RevitLookup GitHub repository](https://github.com/jeremytammik/RevitLookup).

By coincidence, a new enhancement was added just yesterday.

I created a new
[release 2015.0.0.5](https://github.com/jeremytammik/RevitLookup/releases/tag/2015.0.0.5) for
it.

To get the latest and greatest, you can always simply clone the master branch.

You should normally not use older versions posted somewhere else, since they will be outdated as soon as the one and only main GitHub version is enhanced.

I hope this clarifies.