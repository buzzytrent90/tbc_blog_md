---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.625750'
original_url: https://thebuildingcoder.typepad.com/blog/1258_devday_git_stl_obj.html
post_number: '1258'
reading_time_minutes: 5
series: general
slug: devday_git_stl_obj
source_file: 1258_devday_git_stl_obj.htm
tags:
- elements
- revit-api
- views
title: DevDays, GitHub, STL and OBJ Model Import
word_count: 940
---

### DevDays, GitHub, STL and OBJ Model Import

Long time no post.

Sorry, I am rather caught up in the DevDays conferences, accompanying meetups and travel back and forth across the continent.

After the first Western European
[DevDays conference and meetup in Paris](http://thebuildingcoder.typepad.com/blog/2014/12/devday-conference-and-meetup-in-paris.html) on Monday, we continued and repeated in London, UK, and Gothenburg, Sweden.

We had a day's break there, and I took a quick bus and ferry ride out to Styrsö before leaving.

![Styrsö](img/styrsoe_goeteborg.jpg)

Now we have arrived in Munich, Germany, and the last goal for Tuesday and Wednesday is Milano, Italy.

On Sunday, Jim Quanci and I visited Thomas Fink of [SOFiSTiK AG](http://www.sofistik.com) in Nürnberg, the birthplace of
[Albrecht Dürer](https://en.wikipedia.org/wiki/Albrecht_D%C3%BCrer):

![Albrecht Dürer self-portrait at 28, a.d. 1500](img/albrecht_duerer_selfportrait.jpg)

Here we are admiring Jürgen Goertz's 1984 bronze sculpture *Der Hase – Hommage á Dürer (The Hare – A Tribute to Dürer)*, a nod to Dürer's watercolour original *Junger Feldhase* (1502), hinting at the dire results of tampering with nature:

![Jim Quanci, Thomas Fink, the rabbit and me](file:////j/photo/jeremy/2014/2014-12-14_nuernberg/jim_quanci_thomas_fink_jeremy_hare_cropped.jpg)

Although I did not post to The Building Coder in the last couple of days, I was still pretty busy on GitHub.

Check out these new GitHub projects of mine:

- [StlImport](#2)
- [DirectObjLoader](#3)
- [StringSearch](#4)

#### StlImport

The StlImport Revit add-in has been around for some time, although it may not have caught your attention in the past.

It demonstrates some of the Revit 2015 API DirectShape element functionality by importing an interactively selected STL file into a DirectShape element in the Revit project, using the
[NuGet QuantumConcepts STL](http://nugetmusthaves.com/Package/QuantumConcepts.Formats.STL) package
to load and parse the STL file.

It was implemented by Scott Conover of the Revit development team, shown at last year's DevDays Online conferences, posted with the
[DevDays online recording](http://thebuildingcoder.typepad.com/blog/2014/04/revit-2015-api-news-devdays-online-recording.html) and
[migrated to Revit 2015 UR1](http://thebuildingcoder.typepad.com/blog/2014/05/new-revit-2015-sdk-samples.html).

This is the first time I highlight it specifically and publish it on its own, in its very own cosy
[StlImport GitHub repository](https://github.com/jeremytammik/StlImport).

#### DirectObjLoader

The new DirectObjLoader project is the main reason I spent so much time on GitHub and so little time blogging.

It also reminded me of the StlImport sample project and prompted me to publish it, simplifying access to it both for you and me myself.

Inspired by Eric Boehlke of [truevis.com](http://truevis.com) at
the DevHack after the DevDays conference in Las Vegas two weeks ago, I implemented this add-in, similar to StlImport, to load and parse a WaveFront OBJ model and generate a DirectShape element from that:

![DirectObjLoader ribbon panel](img/directobjloader_app.png)

Here is a sample fire hydrant OBJ file:

![Fire hydrant OBJ file](img/directobjloader_fire_hydrant_closed_render.jpg)

DirectObjLoader generates this DirectShape element for it in the Revit model:

![Fire hydrant DirectShape element in Revit](img/directobjloader_fire_hydrant_closed_directshape_rvt.jpg)

Eric requested functionality to specify an input scaling factor, so I added that.

Here is a gargoyle and a half produced by running the command twice, with scaling factors 1.0 and 0.5:

![A gargoyle and a half in Revit](img/directobjloader_gargoyle2.png)

We continued experimenting with larger and more complex OBJ files.

As a final result of that work, OBJ files defining groups now generate a separate DirectShape element for each one:

![Separate DirectShape element for each OBJ group](img/directobjloader_shopping_cart_groups_3.png)

I should certainly describe this project and various interesting aspects of it in more detail soon.

Here are some of the points worth mentioning and discussing in more depth:

- Simple user interaction to select a file
- Simple single button external app user interface
- Using a NuGet package in a Revit add-in
- Reading and parsing OBJ files using [FileFormatWavefront](http://nugetmusthaves.com/Package/FileFormatWavefront)
- Generating a DirectShape element
- Usage of various DirectShape generation options
- The workflow to capture as-built reality and use it to populate a Revit model, i.e. how to get from ReCap to a cleaned-up STL or OBJ model
- Storing user settings in a config file via .NET ConfigurationManager and OpenMappedExeConfiguration

For the moment, you can check it all out yourself in the
[DirectObjLoader GitHub repository](https://github.com/jeremytammik/DirectObjLoader).

#### StringSearch

A recent
[comment](http://thebuildingcoder.typepad.com/blog/2014/10/poipointer-view-depth-override-and-destination-bim.html?cid=6a00e553e16897883301b7c71c2c84970b#comment-6a00e553e16897883301b7c71c2c84970b) raised
the issue of full-text string searches in Revit projects, and that reminded me of the old
[StringSearch plugin of the month add-in](http://thebuildingcoder.typepad.com/blog/2011/10/string-search-adn-plugin-of-the-month.html).

I presented it migrated to Revit 2013 as an example of
[retrieving an embedded resource image](http://thebuildingcoder.typepad.com/blog/2012/06/retrieve-embedded-resource-image.html).

All the Plugin of the Month add-ins were also published on the Autodesk Exchange AppStore, and therefore I migrated it to Revit 2014 as well, but without publishing the updated code here on The Building Coder.

Now all the versions for Revit 2011, 2012, 2013, 2014 and 2015 are available from the
[StringSearch GitHub repository](https://github.com/jeremytammik/StringSearch).

Now we are in the middle of the Munich DevDays conference, with the meetup coming up tonight, [I Love 3D – Munchen](http://meetu.ps/2CG1wx) – I hope to see you there!