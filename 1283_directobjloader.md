---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 13.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.673365'
original_url: https://thebuildingcoder.typepad.com/blog/1283_directobjloader.html
post_number: '1283'
reading_time_minutes: 32
series: structural
slug: directobjloader
source_file: 1283_directobjloader.htm
tags:
- csharp
- elements
- family
- geometry
- parameters
- python
- references
- revit-api
- views
- windows
- structural
title: "From Hack to App \u2013 OBJ Mesh Import to DirectShape"
word_count: 6394
---

### From Hack to App – OBJ Mesh Import to DirectShape

I already took a couple of looks at the DirectShape element in the past:

- [DirectShape Performance and Minimum Size](http://thebuildingcoder.typepad.com/blog/2014/05/directshape-performance-and-minimum-size.html)
- [DirectShape versus Families, Category and Texture](http://thebuildingcoder.typepad.com/blog/2014/11/directshape-versus-families-category-and-texture.html)

A much deeper investigation was prompted by Eric Boehlke of [truevis.com](http://truevis.com),
knowledgeable in architecture and above all in photogrammetry.

Eric visited the DevHack at Autodesk University in December 2014.
Together, right there and then, we put together a prototype Revit add-in that reads an OBJ file and creates a DirectShape element from it.

Eric continued working on it at his end, adding a user interface and submitting it to the
[Autodesk Exchange AppStore](https://apps.exchange.autodesk.com) as Revit
[Mesh Import from OBJ files](https://apps.exchange.autodesk.com/RVT/en/Detail/Index?id=appstore.exchange.autodesk.com%3ameshimport1421090501_windows64%3aen),
while I maintained a non-UI open source GitHub version named
[DirectObjLoader](https://github.com/jeremytammik/DirectObjLoader).

Here is a rather lengthy and overdue summary of our correspondence on this and all the numerous issues we encountered and solved involving both the Revit API and the AppStore submission.

Please note that Eric is by no means a professional or even experienced programmer, so he deserves extra honours for the feat of implementing and submitting his very own AppStore app!

**Question:**

I have found some code for reading OBJ in C#,
[ObjLoader by ChrisJansson](https://github.com/ChrisJansson/ObjLoader),
but it is over my head.

I got your
[DirectShapeMinSize](http://thebuildingcoder.typepad.com/blog/2014/05/directshape-performance-and-minimum-size.html) add-in
running in Revit.

Some mash-up of the two programs may be the answer.

**Answer:**

I implemented an initial version
[2015.0.0.0](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.0) using the .NET
[FileFormatWavefront](http://nugetmusthaves.com/Package/FileFormatWavefront) NuGet package based on Dave Kerr's
[file-format-wavefront](https://github.com/dwmkerr/file-format-wavefront) GitHub library
that successfully loads the fire hydrant OBJ file:

![Fire hydrant DirectShape](img/directobjloader_fire_hydrant_closed_directshape_rvt.jpg)

**Response:**

This is going to be very useful. I tested a new app that turns laser scan points to mesh (and can export OBJ). Brilliant software!

I gave them a scan and it converted it for me, instantly.

Is there any generic mesh element in Revit? Or maybe we are on the right track and it's some other problem. Do you have access to OBJ import code from other Autodesk apps?

**Answer:**
There is a generic mesh geometric object.
That is not a database element.
The direct shape is the most 'meshy' database element.

Nope, I have no access to any internal Revit code, and I don't want any :-)

Have you done any documenting at all of what you have done so far or where you would like to go with this?

**Response:**

Since you got the software working in one case, perhaps it is a question of giving the right mesh.
I'll have to experiment with meshes.
I really want to get it working, though.

Points in ReCap:

![Points in ReCap](img/directobjloader_1.png)

Mesh in Memento:

![Points in Memento](img/directobjloader_2.png)

**Answer:**

You might want to download the current updated
[version 2015.0.0.1](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.1) that
stores the last selected OBJ folder persistently across sessions.

I may go ahead and publish what we have so far as is.

One idea: I could also add STL import to the same add-in.

**Response:**

I spoke with your colleague, Angel Velez, today.

He is the super IFC importer programmer.
That importer is very similar to our effort.
He recommended seeing his
[IFC source code](http://sourceforge.net/projects/ifcexporter/files/2015/) on
SourceForge, especially "IfcFacetedBrep".

For our code, he suggested changing the "builder.Build" call to:

```csharp
  TessellatedShapeBuilderResult r
    = builder.Build(
      TessellatedShapeBuilderTarget.AnyGeometry,
      TessellatedShapeBuilderFallback.Mesh,
      graphicsStyleId );
```

OR

```csharp
  TessellatedShapeBuilderResult r
    = builder.Build(
      TessellatedShapeBuilderTarget.Mesh,
      TessellatedShapeBuilderFallback.Salvage,
      graphicsStyleId );
```

**Answer:**

Great guy to ask, great suggestions!

I implemented
[version 2015.0.0.2](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.2) to
use mesh + salvage instead of solid + abort when building the DirectShape.

**Response:**

YES!!!!!!

Mesh + Salvage produce this:

![Gargoyle in Revit](img/directobjloader_3.png)

**Answer:**

Congratulations!

Can you do a pull request from GitHub?

I'll write a blog post draft on this, but that may take a while.

Will you write anything at all about your motivation and workflow?

**Response:**

Our baby has just been born. Let's be patient about documenting it.

Can direct shape go into a Family RFA?

I heard a rumour that the current release cannot display Direct Shapes in RFA, but a future version may.

I would like to run with this app, refine it with an options dialog, error checking, etc., and publish in the Autodesk Exchange Store as was recommended to us by the DevDays keynote speaker. It should be of value to many.

It was a great week. I hope you made some good connections.

**Answer:**

I confirmed that it works with the gargoyle.

Okey-doke, fine by me.

Great news, in fact, I am very glad!

Go for it!

I would also suggest adding STL import functionality.

I can do that.

I would like to continue updating the minimalist version on GitHub.

Are you thinking of continuing to work on that same public version, or forking an own version for the appstore thingy?

Do you want it to remain free and open source including your enhancements?

I would like it to remain one version and open source on GitHub.

**Response:**

I think that's a good plan.
I think I'd make my own version with a UI with the goal to publish in the Store.

Another thing that would be easy to implement is: have it try a solid, first, on failure, go to mesh, then salvage. Or, it could be a user option to attempt making a solid to save time as most solids will fail.

A tricky thing to implement would be to get some material/color on the faces. Wikipedia provides
[some OBJ info](http://en.wikipedia.org/wiki/Wavefront_.obj_file).

If the RVT template had 256 materials in it, each with a color corresponding to the old AutoCAD colors, that could get some rough materials on the faces, perhaps. From the MTL file:

- The ambient color of the material is declared using Ka. Color definitions are in RGB where each channel's value is between 0 and 1.

  ```
  Ka 1.000 1.000 1.000 # white
  ```

Hence, I think RGB would be calculated by multiplying the Ka numbers by 256.

**Answer:**

If you look at the code, you will note I added a Config class.

One of its options is "TryToCreateSolids".

Great minds think alike.

I am confident that we can get colours to work fine as well, and maybe textures also.

Revit does not work with 256 predefined colours, but full RGB.

More info on colours in OBJ and Revit:

- [OBJ model exporter with colours](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-colours.html)
- [OBJ model exporter with transparency support](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-transparency-support.html)

**Response:**

Right. I am thinking to pre-make 256 RVT Materials, one for each of the ACAD 256 colors. I wouldn't make a Material for 256^3 colors!

**Answer:**

That is not necessary.
Just make one, or none, because you can then duplicate a material and assign it a new colour on the fly when you need it.

**Response:**

I converted another file and went from an SVG image of a logo, to TinkerCAD, to OBJ, to Memento (to re-size), to our importer app in Revit.

Amazing, eh?

It will need much tweaking before it can be really used easily in projects.
An important feature to add will be a scale factor input.

**Answer:**

The input scaling factor makes a lot of sense, so I went ahead and implemented an input scaling factor stored in config file.

It can be used, for instance, to generate this gargoyle and a half:

![Gargoyle and a half](img/directobjloader_gargoyle2.png)

Check it out on GitHub in the
[version 2015.0.0.3](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.3).

**Response:**

Is there a form somewhere to input the scale factor? I didn't see it.

Here is another import:

![Shopping cart](img/directobjloader_9.png)

I had to process it though Memento for it to import into Revit.

The original OBJ produces this dialog:

![Original shopping cart exception](img/directobjloader_10.png)

I uploaded the two 'cart' files to the Google folder. The first one fails.

There may be a memory limit to build a Direct Shape. Perhaps the IFC software checks for it.

**Answer:**

1. Regarding the 'form somewhere to input the scale factor':

No, of course not.
User interfaces are for wimps.
You have to edit the config file located in the same place as the DLL and add-in manifest:

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <appSettings>
    <add key="defaultFolderObj" value="C:\a\vs\DirectObjLoader\test" />
    <add key="inputScaleFactor" value="0.5" />
    <add key="tryToCreateSolids" value="true" />
    <add key="maxFileSize" value="50000000" />
    <add key="maxNumberOfVertices" value="100000" />
  </appSettings>
</configuration>
```

2. Regarding the two 'cart' files:

It should be possible to handle that error more gracefully.

I took a look at the new sample files and see the cause of it now.

Again, it has to do with the OBJ file format, how it is populated, and our lack of understanding of it.

If you look at the code, the current implementation grabs all the faces from this loop:

```csharp
  foreach( Face f in obj\_load\_result.Model.UngroupedFaces )
```

This provides a sensible result in the cart-exported file, which I successfully imported into Revit like this:

![Shopping cart ungrouped](img/directobjloader_shopping_cart.png)

The problem with the 'unexported' file is not memory.

It has a higher intelligence, that is all.

In the 'unexported' file, the result.Model.UngroupedFaces collection is empty.

Instead, the OBJ model defines 17 named OBJ groups:

![DirectObjLoader groups](img/directobjloader_groups.png)

This is presumably much better, because the groups carry information and structure, whereas ungrouped faces do not.

So your 'export' procedure is probably destroying model structure and valuable information.

The importer is currently ignoring groups, however.

I added an error message on zero faces in
[version 2015.0.0.4](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.4),
so you have a more graceful exit, at least.

I also added a first stab at support for groups in
[version 2015.0.0.5](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.5).

The result for the 'unexported' shopping cart is not good yet, though.

**Response:**

I think it's acceptable to have an error/fail dialog when there is something too complicated in the file for importing. Running the OBJ through Memento to decimate it may often be a good thing.

Looks like the OBJ format can get very sophisticated:
[paulbourke.net/dataformats/obj](http://paulbourke.net/dataformats/obj).

Under the heading Grouping, he explains the syntax and gives several examples of Group usage.

Yesterday, I tried to use [Meshlab](http://meshlab.sourceforge.net) to convert a point cloud to a mesh. I got it to make a mesh regarding the points but need to experiment more because it was very far off. There are all kinds of unexplained parameters in the Meshlab UI.

I haven't heard back from [thinkboxsoftware.com](http://thinkboxsoftware.com),
who were showing some nice software at AU that converts points to mesh.

Ultimately, I imagine selecting points in Revit and converting them to a mesh. Someday...

**Answer:**

I found one improvement, closing the face set off for each OBJ group, and implemented that in
[version 2015.0.0.6](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.6).

The result with that enhancement is still not perfect, though.
It looks like this:

![Shopping cart groups 2](img/directobjloader_shopping_cart_groups_2.png)

Some progress at least.

I also uploaded the [STL importer](https://github.com/jeremytammik/StlImport) to GitHub.

It includes two sample files to load.

Even more to play with!

**Response:**

Excellent. (I think I saw that cart in back of the Market Basket on the train tracks.)

I was thinking, What if there was a separate shape element created for each Group if there are any extant?
Hence, a loop the same as the main mesh for each Group.
Do you think that would work?

**Answer:**

Indeed excellent.

When you put on your thinking cap, you deliver.

This is the result of loading the shopping cart and closing off each OBJ group into an own separate DirectShape element:

![Shopping cart groups 3](img/directobjloader_shopping_cart_groups_3.png)

So it looks like we are done with this for the moment, with
[version 2015.0.0.7](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.7).

**Response:**

Splendid work, Jeremy.
You have gone from Cubism to Realism in your artistic style.

After 14 years, OBJ can be imported to Revit.

May the thinking cap will remain on...

I was working on a facade model for a client yesterday. The client does not want to pay for laser scanning, yet. We went out and took 50 pictures, then uploaded to Recap360 and turned the photos into a mesh. After editing the mesh in Memento, it comes into Revit fine. The problems are scaling and position -- rotation and UCS of mesh. These things are able to be edited in Memento; it's tricky, though.

The units in the API when making the Direct Shape are feet, yes? I need to figure out how Memento is scaling the numbers for the vectors in its exported OBJ.

**Answer:**

Yes, all Revit linear database units are feet.

**Response:**

Memento is very helpful.

Some OBJs do not import into Revit well. Memento can fix holes, etc.

Imported OBJ found online:

![Cessna with holes in Revit](img/directobjloader_4.png)

Imported OBJ fixed in Memento:

![Cessna fixed in Memento](img/directobjloader_5.png)

**Answer:**

Well, good for Memento, and good for you!

Yes, well...

This may be an issue with Revit.

More probably, I believe, it has something to do with the OBJ toolkit library we are using.

Or with the OBJ file format, which we are supporting in an incomplete and untested fashion.

Actually, go ahead and send me the two files and I will see whether I notice any of our assumptions being violated.

For instance, we assume triangles or quadrilaterals. Maybe some of the OBJ polygons have more than four corners?

Notice how the Memento version below has triangulated everything?

**Response:**

I think the culprit is "s" = smoothing group, cf.
[www.martinreddy.net/gfx/3d/OBJ.spec](http://www.martinreddy.net/gfx/3d/OBJ.spec):

> "Smoothing group statements let you identify elements over which normals are to be interpolated to give those elements a smooth, non-faceted appearance. This is a quick way to specify vertex normals."

Even Memento does not seem to handle those groups properly. Hence, I filled the holes using the error-checking functions in Memento.

After initial Open in Memento:

![Cessna initially opened in Memento](img/directobjloader_6.png)

But MeshLab does better! I opened the original OBJ and then exported OBJ -- MeshLab converted everything without any other processing.

![Cessna in MeshLab](img/directobjloader_7.png)

That OBJ into Revit (best of all):

![Cessna in Revit](img/directobjloader_8.png)

I think simply a warning that the OBJ contains a Smoothing group or a Merging group is adequate, currently.

By the way, I just wrote this post about importing a
[photogrammetry mesh into Revit as a point cloud](http://revitlearningclub.blogspot.com/2014/12/photogrammetry-mesh-into-revit-as-point.html).

The method of determining a mesh's scale will be similar for the DirectObjLoader to work correctly, except the scaling factor will ultimately be to feet, not meters.

#### User Interface

I have a UI working except for a Material selector, and the LinkLabel.

It is slow going for me because I am so bad at C#. I am getting through it, though, with much searching, trial, and error.

![User interface](img/directobjloader_12.png)

The category can be picked from a drop down list:

![Category drop down](img/directobjloader_13.png)

This is the way to go for the TessellatedShapeBuilder, I think:

```csharp
TessellatedShapeBuilderTarget.AnyGeometry,
TessellatedShapeBuilderFallback.Mesh,
```

The graphics style comes out differently for AnyGeometry and Mesh, but I think I found a workaround.

I have also found a way to turn "closed" points in a laser scan point cloud into a mesh using MeshLab. Bringing into Revit and changing Material is fun :-)

![Adding materials in Revit](img/directobjloader_11.png)

**Answer:**

I believe that the options used by the [STL importer](https://github.com/jeremytammik/StlImport) support all combinations of target geometry + fallback that make sense.

**Response:**

I made a
[Mesh Import landing page](http://truevis.com/revit-mesh-import/).

I was thinking to have a drop down list of available Revit Materials in the UI, and to assign one Material to the whole shapes as they are created. You have some code snippets on your blog about getting Materials but I couldn't get them to work. I'm sure the IFC importer does that somewhere but the code is too complicated for me.

#### Smoothing

A problem with the importer as you wrote it is that the smoothing or 's' data in the OBJ are not used or used correctly. See attached 'sandal.obj'. When I process an OBJ through Mesh Lab or Memento, the 's' data get converted to faces. That is an adequate workaround at this time. My challenge this week is to get the app published without too much feature creep.

**Answer:**

I fixed the OBJ loader to support faces with more than four vertices in
[version 2015.0.0.9](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.9).

This enables partial loading of the sandal.obj sample file, but with some holes, similar to the untreated Cessna:

You want to watch out with your Memento clean-up... it destroys the OBJ groups and thus looses important model structure information.

Here is the complete Cessna cleaned up by Memento:

![Cessna exported from Memento](img/cessna-exported-from-Memento.png)

Here is the original one with model structure but lacking some faces:

![Cessna with groups and missing faces](img/cessna_with_groups_and_missing_faces.png)

I switched TessellatedShapeBuilder target from Mesh to AnyGeometry as well, and it does indeed look a bit better.

**Response:**

I think AnyGeometry attempts to make a solid.
If a solid is made, dimensions can work on it.

The bigger issue is the smoothing data that are not processed in the original OBJ. I am okay with it for now. From searching around it looks like many apps don't handle smoothing with OBJs. Paraphrasing the founder of LinkedIn: "If you are not embarrassed by the first release of your product, you worked on it too long".

Attached is another difficult OBJ. Running it though MeshLab and turning it to triangles fixes it, though.

**Answer:**

I do not believe the tessellated face builder supports smoothing.

The inverse operation, using the custom exporter to generate an OBJ output, would happily support this, though.

**Response:**

There must be a way to handle the smoothing stuff, and just make facets, since MeshLab and Memento do it.
I just don't know how to do it.

This
[convert\_obj\_three.py](https://github.com/mrdoob/three.js/blob/master/utils/converters/obj/convert_obj_three.py) Python
code from three.js seems to handle smoothing in OBJ.

**Answer:**

As already said, the DirectShape generator provided by the Revit API does not support smoothing.

It is pure geometry.

Smoothing is pure rendering.

No connection, no workaround.

Forget it.

#### Pyramid Problem – OBJ File Vertex Index Error

**Response:**

I have one OBJ that causes an error. It's a pyramid and I think it's causing a 0 size face attempt. But I don't know for sure. I can send the file if you want to check it.

**Answer:**

I can take a look at the pyramid problem, sure.

I also want to look at that plane and check whether it has faces with more than four vertices...

**Response:**

I provided two pyramid OBJs that crash the importer as I have it using AnyGeometry and fallback as Mesh.
I tried to debug it, but didn't solve the problem, yet.

Here is where "pyramid.obj" fails:

```
System.ArgumentOutOfRangeException was unhandled by user code
Message=Index was out of range.
Must be non-negative and less than the size of the collection.
```

Do you get that error, too?

**Answer:**

I would say there is an error in the pyramid OBJ file.

How was it created?

Look:

```
# OBJ file created by ply_to_obj.c
#
g Object001

v  0  0  0
v  1  0  0
v  1  1  0
v  0  1  0
v  0.5  0.5  1.6

f  5  2  3
f  4  5  3
f  6  3  2
f  5  6  2
f  4  6  5
f  6  4  3
```

It defines five vertices.

They are numbered 0, 1, 2, 3, and 4.

The faces refer to faces numbers 2, 3, 4, 5 and 6.

That is wrong.

In addition, though, I ran into a strange problem debugging this.

I fixed the vertex indices in the face by subtracting 2 from each one, but the add-in was still working with the old data.

Currently, I can only imagine that the OBJ parsing library is caching something that it should not.

**Response:**

Yes, I think you are right. MeshLab brings it in without complaining but part of it appears to go to infinity.

That OBJ was from a web page with lots of OBJs.

I'd like to catch the error something like this, and just end the command. Is there a way to break out of the command from inside the loop if that error is encountered?

```csharp
if (vertices.Count > i.vertex)
{
MessageBox.Show("Error: Face and vertex quantities do not match.");
return Result.Cancelled;
}
```

**Answer:**

Check out the new
[version 2015.0.0.12](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.12):
abort and display error message on invalid OBJ file due to face vertex index exceeding total vertex count.

**Response:**

Thanks. Looks like you added messages to the top of the debug messages.
Do you know of a method to cleanly quit an API app when an error is found?

**Answer:**

Do you mean abort and close down and exit the add-in? Or simply abort the current external command execution, but remain loaded for a renewed attempt?

The former is not really common or supported by the Revit API. It would be possible, but why would you want that?

The latter is exactly what I now do.

It is achieved by returning from the external command Execute method.
Its return values are Succeeded, Cancelled or Failed.

**Response:**

I'd like to cleanly show an error message, then exit. Currently one gets a pile of debug stuff in the message when there is an error.

**Answer:**

Regarding exiting cleanly, I already explained:

Return from the external command Execute method.
Its return values are Succeeded, Cancelled or Failed.

The 'pile of debug stuff' comes from an unhandled exception.

You can either add a catch-all exception handler round your entire command and then decide for yourself what information is displayed before you return from Execute, or you can handle each exception separately in a targeted manner.

The latter is only possible once you know what the exception is and what is causing it, though.

I can add a catch-all exception handler to demonstrate, if you like.

I implemented two exception handlers in
[version 2015.0.0.13](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.13).

Please test and let me know how it goes.

**Response:**

What running your release DLL with "pyramid.obj" gets:

![Invalid vertex index error message](img/directobjloader_14.png)

Good.

#### Submitted to Autodesk Exchange Apps and Ribbon Panel Icon

**Response:**

I did it. You can see the Store page preview here.
I'm sure I will update it, but I wanted to get it out to the world as soon as possible.

Thank you for all of your help. I'll give you a copy when it gets approved.

**Answer:**

I think you can remove the PDB files from the distribution:

```
$ unzip -l eb_MeshImportRelease1.zip

Archive:  eb_MeshImportRelease1.zip

  Length     Date   Time    Name
 --------    ----   ----    ----
   119808  01-03-15 20:22   MeshImport.dll
    34304  01-03-15 20:22   MeshImport.pdb
      463  01-02-15 13:21   MeshImport.addin
    17887  12-16-14 21:00   FileFormatWavefront.xml
    56832  12-16-14 21:00   FileFormatWavefront.pdb
    19456  12-16-14 21:00   FileFormatWavefront.dll
 --------                   -------
   248750                   6 files
```

Probably, you can also remove the XML file.

It looks like the FileFormatWavefront library API documentation to me.

It might also possible contain the FileFormatWavefront library error messages, but I think not.

I think this is all you need:

```
   119808  01-03-15 20:22   MeshImport.dll
      463  01-02-15 13:21   MeshImport.addin
    19456  12-16-14 21:00   FileFormatWavefront.dll
```

Good luck getting it approved and launched.

**Response:**

Do you know of any apps that have the code published which have also gotten in the App Store?

I am having trouble getting the ribbon stuff all correct for them.

There doesn't seem to be any template or sample code that shows how to get everything ready to publish. Do you have any advice?

**Answer:**

There is lots of sample code for that.

Many of my apps have that as well.

All you need to do is implement an external application, define a button to invoke the external command that you already have, and remove the external command itself from the add-in manifest.

All the ADN add-ins of the month that we published a whole series of were converted to AppStore apps, and their code is available, e.g. my StringSearch add-in became ADNPlugin-StringSearch.dll.

I can easily implement it for you sometime.

I checked the project, and it already defines an external application as well as the external command.

The external command is visible in the AddIns > External Tools menu, and it should not be.

The external application defines a custom panel in the AddIns tab that launches the same external command as well, and that is what you want.

I removed the external command entry from the add-in manifest in
[version 2015.0.0.15](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.15),
and now all should be well.

**Response:**

I got my add-in file working but haven't figured out how to have a single image as its button.
I don't know how to get the icon to work in a ribbon, yet.
The guy from Exchange Apps is trying to send me some sample code, so I may be able to copy that.

I also need to make ContextualHelp F1 work.

The Building Coder provides some clues. As always, I have to hack around for days before I get it done.

Yes, your SLN now works as a ribbon button with no icon.

I'm sure I am missing something very obvious to you, here.
Where and how do I add the button with its image?
Easy?

**Answer:**

Send the images that you want to use along.

Note they need the right size and resolution, c.f. below.

Two things:

- Populate the button with a bitmap.
- For the sake of practicality, embed the image icon into the DLL.

Otherwise, you have to copy a stupid separate image file around with your assembly DLL.

Everything you need and much more is on the blog, in the
[PushButtonData usage examples](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.26).

**Response:**

Yes, I am sure your blog has all the clues, and I keep using it, but actually putting it all together remains a big challenge for me.

This is also not working for me:

```
  using System.Windows.Media.Imaging;
```

**Answer:**

The using statement can cause an error if the corresponding .NET assembly DLL has not been added to the project references.

**Response:**

Found System.Windows.Media.Imaging!

Hacked, and hacked, and ....... did it!

![Mesh import ribbon panel icon](img/directobjloader_15.png)

I referenced the Revit API Developers Guide Walkthrough:
[Add Hello World Ribbon Panel](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-01F579CB-AB46-4C00-86E4-D189510D3774).

I got the images as "Embedded Resources" in my project. It is crucial to add the PNGs one at a time in VS! I wasted several hours because VS lets you load multiple existing files into the project, but if you do it that way, it doesn't let you change them to "Embedded Resources".

**Answer:**

I never ever heard of such a problem and cannot really believe that such a problem exists.

Anyway, I added the icon you provided as an embedded resource command icon to the GitHub
[version 2015.0.0.16](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.16):

![DirectObjLoader ribbon panel icon using embedded resources](img/directobjloader_16.png)

#### Try/catch suggestion

**Response:**

I submitted an
[Issue #1](https://github.com/jeremytammik/DirectObjLoader/issues/1):

I suggest adding try/catch around builder.AddFace:

```csharp
try
{
builder.AddFace(new TessellatedFace(corners,
ElementId.InvalidElementId));
}
catch
{
// remember something went wrong here..
}
```

**Answer:**

I updated the code with a catch-all exception handler in
[version 2015.0.0.17](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.17).
I would like to narrow it down to the specific exception thrown, though.

#### The App is Live in the Store

**Response:**

The
[MeshImport](https://apps.exchange.autodesk.com/RVT/en/Detail/Index?id=appstore.exchange.autodesk.com%3ameshimport1421090501_windows64%3aen)
app is now live and for sale at $50 in the Autodesk Store.

I couldn't have done it without you.

**Answer:**

congratulations!

Did you pop a bottle of bubbly?

Chin-chin!

Your [Revit mesh import](http://truevis.com/revit-mesh-import) web page still says "coming soon..."

You might want to fix that.

I would like to summarise our discussions and iterative development process as a blog post on the topic "from hack to app".

Please keep me up to date how it goes for you with this.

If there is anything else I can do to help, just give us a ring.

**Response:**

No, just made a post
[announcing Mesh Import](http://www.revitforum.org/commercial-free-add-ins-extensions/23119-mesh-import.html) on
the Revit Forum.

The thread includes some discussion and enthusiasm after publication of the app, plus some really cool realistic renderings.

![Mesh Import bed test](img/mesh_import_bed_test.jpg)

#### Troubleshooting Page

**Response:**

I added a [troubleshooting page](http://truevis.com/troubleshoot-revit-mesh-import) for
the Mesh Import app.

A man from Italy who is doing historic renovation work sent me a photogrammetry-created mesh; I show what I did to get it into Revit.

I should do something to catch the mesh too big error.

**Answer:**

I am still pondering the 'hack to app' blog post :-)

Yes, every potential expected error should be caught and handled gracefully, mainly meaning that a sensible message is presented to the user explaining what went wrong and what steps to take to fix it.
Pointing to a troubleshooting web page is a great solution!

To catch the error and add code to handle it, simply run the app and generate the error inside the Visual Studio debugger.

Visual Studio should tell you exactly what exception is thrown.

Store that information and add a dedicated exception handler for it.
In the exception handler, display a message explaining everything to the user.

I'll happily help.

Just let me know.

Can I download the mesh causing the error anywhere?

**Response:**

I sent you the original file.
It has about 1.2 million faces.

**Answer:**

Got it:

```
Exception generating DirectShape 'Mesh':
cannot create a geometry object in a requested mode as result will be too big to handle
```

One simple check one could make is to post a warning to the user if the number of vertices exceeds 100.000, or 300.000, or whatever limit one would like to set.

The vertices are all collected and counted and scaled anyway, before anything else happens.

At that point, it would be sensible to tell the user:

"Excuse me, but you are loading a mesh defining ... vertices. We suggest using no more than ..., since Revit will refuse to handle such a large mesh anyway. Please refer to the troubleshooting page at <http://truevis.com/troubleshoot-revit-mesh-import> for suggestions on how to reduce the mesh size."

I implemented

- [version 2015.0.0.18](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.18) – implemented Config.MaxNumberOfVertices and graceful exit on too many mesh vertices
- [version 2015.0.0.19](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.19) – implemented contextual help for external command push button in ribbon panel

**Response:**

That was fast. Thanks for the advice, too.

I have sold one copy of the app – that was very exciting.

**Answer:**

Found some more errors and added an immediate bail-out message if the file is to large in
[version 2015.0.0.20](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.20):
implemented Config.MaxFileSize check for huge file sizes and abort before trying to load them.

#### DirectObjLoader Version History

The most up-to-date version of the GitHub version of this app is provided by the
[DirectObjLoader repository](https://github.com/jeremytammik/DirectObjLoader).

Here is a complete list of the versions created so far, illustrating the functionality described above step by step:

- 2014-12-02 [2015.0.0.0](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.0) initial version with Eric Boehlke loaded fire hydrant
- 2014-12-03 [2015.0.0.1](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.1) added config class to store last folder between sessions
- 2014-12-05 [2015.0.0.2](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.2) use mesh + salvage instead of solid + abort when building the DirectShape per suggestion from angel velez via eric boehlke and gargoyle is now successfully loaded
- 2014-12-10 [2015.0.0.3](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.3) implemented input scaling factor stored in config file, cf. gargoyle2.png
- 2014-12-11 [2015.0.0.4](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.4) added face count and error message on zero faces
- 2014-12-11 [2015.0.0.5](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.5) added initial primitive test support for groups as well as ungrouped faces, result directobjloader\_shopping\_cart\_groups.png
- 2014-12-11 [2015.0.0.6](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.6) call OpenConnectedFaceSet for each OBJ group, result directobjloader\_shopping\_cart\_groups\_2.png
- 2014-12-11 [2015.0.0.6](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.6) name DirectShape element same as input file
- 2014-12-12 [2015.0.0.7](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.7) create separate DirectShape element for each OBJ group, add appGuid and name shapes better, result directobjloader\_shopping\_cart\_groups\_3.png
- 2014-12-12 [2015.0.0.8](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.8) capitalise and replace underscore by space in DirectShape element name
- 2015-01-02 [2015.0.0.9](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.9) added support for faces with more than four vertices, enabling successful load of sandal.obj
- 2015-01-02 [2015.0.0.10](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.10) switched TessellatedShapeBuilder target from Mesh to AnyGeometry
- 2015-01-02 [2015.0.0.11](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.11) set TessellatedShapeBuilder LogString and LogInteger properties
- 2015-01-02 [2015.0.0.12](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.12) abort and display error message on invalid OBJ file due to face vertex index exceeding total vertex count
- 2015-01-05 [2015.0.0.13](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.13) added two exception handlers for loading OBJ file and generating DirectShape
- 2015-01-12 [2015.0.0.14](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.14) removed external command from add-in manifest, leaving only the external application
- 2015-01-12 [2015.0.0.15](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.15) fixed a logical error handling nFaces and nFacesTotal count
- 2015-01-15 [2015.0.0.16](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.16) display command button icon stored in embedded resources
- 2015-01-23 [2015.0.0.17](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.17) wrapped call to AddFace in an own exception handler and added a debug log reporting count of faces added and failed
- 2015-02-15 [2015.0.0.18](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.18) implemented Config.MaxNumberOfVertices and graceful exit on too many mesh vertices
- 2015-02-15 [2015.0.0.19](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.19) implemented contextual help for external command push button in ribbon panel
- 2015-02-15 [2015.0.0.20](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.20) implemented Config.MaxFileSize check for huge file sizes and abort before trying to load them
- 2015-02-17 [2015.0.0.21](https://github.com/jeremytammik/DirectObjLoader/releases/tag/2015.0.0.21) published blog post