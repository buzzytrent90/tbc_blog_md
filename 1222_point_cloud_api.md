---
post_number: "1222"
title: "Point Cloud Indexing Update"
slug: "point_cloud_api"
author: "Jeremy Tammik"
tags: ['parameters', 'python', 'revit-api', 'views']
source_file: "1222_point_cloud_api.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1222_point_cloud_api.html"
---

### Point Cloud Indexing Update

The Point Cloud API has changed a bit in the past couple of years, and it is time for an update of the previous
[point cloud overview](http://thebuildingcoder.typepad.com/blog/2011/11/au-begins-and-point-cloud-overview.html) from 2011.

My re-exploration of this topic was triggered by Henrik's Revit API discussion forum thread on the
[point cloud indexer API](http://forums.autodesk.com/t5/revit-api/point-cloud-indexer-api/m-p/5306679):

**Question:** Is there an API or another way to programmatically access the Indexer of PointClouds?

I am referring to the dialog that pops up when you try to import a different format point cloud, e.g., e57.

Do you have an example of the Indexer API?
Or is it possible to access the functions through python?
Or command line, maybe?

Besides this we have another question that maybe you can answer as well:
We have had some questions if Revit / Recap, when it converts from e.g. .fls to .rcs, moves points into a grid structure?
It is questions from land surveyors, who believe this is the case, and mention that it is not suitable for their work if they can't count on that the points are in the same position as when they scan.
Do you know if this is the case?

**Answer:** Wow, that was a pretty chase...

For your convenience, I put together a dedicated
[point cloud](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.43)
topic group on The Building Coder listing the discussions published so far.

However, the
[Point Cloud Indexer API](http://thebuildingcoder.typepad.com/blog/2011/11/au-begins-and-point-cloud-overview.html) provided
by the Revit 2012 API is now deprecated.

Before discovering that, Adam Nagy republished his
[Revit 2012 point cloud indexer](https://github.com/adamenagy/RevitPointCloudIndexer)
sample for Revit 2012 on GitHub.

It would in theory be interesting to hear whether it works as is or can be modified to work with Revit 2015.

You could fork Adam's repository, keep track of the changes, and send him a pull request when done to ask him to merge them back in.

As said, though, this API is now deprecated and may not be usable at all.

I found another discussion on
[batch PCG creation using Autodesk point cloud indexer](http://forums.autodesk.com/t5/revit-architecture/batch-pcg-creation-using-autodesk-point-cloud-indexer/td-p/3238878) that confirms this.

Once that was clear, I asked the development team about the current status.

Their initial response was, "We moved away from writing indexers, right? ReCap SDK should be where customers go now?"

Indeed, that just about summarises it.

Arkady Gilman and his colleagues provided some more detail and background information as follows:

The Revit team no longer writes indexers, just uses the ones provided by the Recap team.
We still do indexing of raw data into Recap format in Revit, though.

The Revit 2013 API for alternative point cloud engine is deprecated.
Please do not implement another engine.
Use ReCap instead.

Similarly, PCG is deprecated and should not be used.

Revit has an indexer, i.e. a separate executable that is launched when you want to insert a file other than RCP or RCS (e.g. PTX, PTS of FLS).
This indexer is in fact a 'headless' version of ReCap.
So adding a 'codec' (I think they use this term) for ReCap will automatically make new format available for Revit indexer as well.
In general, the Revit indexer is intended for a novice, sporadic user.
We assume that people using point clouds 'for real' will use ReCap to index and register scans before importing the data to Revit.

ReCap may be making and supporting the standalone indexer at some point.
Revit probably only uses our thin UI for converting old (PCG) point cloud files into the ReCap format (RCP, RCS).

Can you clarify what you are trying to do? Do you want to index some new 'raw' data format that ReCap does not currently recognize?

From the thread it is not clear exactly what you want to achieve.

You ask: We have had some questions if Revit / Recap, when it converts from e.g. .fls to .rcs, moves points into a grid structure? It is questions from land surveyors, who believe this is the case, and mention that it is not suitable for their work if they can't count on that the points are in the same position as when they scan. Do you know if this is the case?

The answer is that ReCap indexing does not alter point coordinates; they are completely sacred.

What do you mean by 'moving points into a grid structure'?
ReCap stores points in a spatial tree, but this is just the method used to quickly find points within a given part of space.

There is whole separate issue w/regard to what happens with different scans when they are registered with respect to one another.

**Response:** First of all thank you very much for your answers this is really great.   :-)

To explain what we are doing: I am part of a research group called CITA (Center for IT and Architecture) at the school of architecture in Copenhagen, Denmark.

We are part of an EU funded research project called [duraark](http://www.duraark.eu), which deals with the life circle of architectural knowledge, in this case ifc and e57 formats.

The project deals with how a long-term archive of architectural knowledge can be set up and integrated into the workflows of the stakeholders involved in the architectural disciplines.

For this we are developing methods of e.g. extracting metadata and verifying quality from point clouds and from comparison of point clouds and BIM models.

Because the majority of the stakeholders are working in Autodesk products, we want to be able to integrate these extraction, comparison and quality checking tools into Revit.

At the moment we are considering using Dynamo for prototyping the integrated workflow of our stakeholders.

At the end of the project, we will hopefully have a plugin for Revit   :-)

To answer your questions:

1. "Can you clarify what you are trying to do? Do you want to index some new 'raw' data format that ReCap does not currently recognize? From the thread it is not clear exactly what you want to achieve."

The first thing we want to be able to do is to visualize .e57 point clouds in Revit without having to go through and open them in Recap to transform them into the rcs format.

One idea I just had now for this was maybe to be able to access "the separate executable that is launched when you want to insert a file other than RCP or RCS" ('the headless' version of ReCap) through command line. And in this way be able to visualize and link e57 point clouds in Revit with full control over the files generated and linked to Revit from within Dynamo and python.

Would a command line solution be possible?

Or can you see another way?

Is there an API for Recap?

2. "... if they can't count on that the points are in the same position as when they scan. Do you know if this is the case?"

I do not know for sure if this is the case.
It was a comment I got from one of our stakeholders.
As you say that "ReCap indexing does not alter point coordinates; they are completely sacred", then I will assume this really is the case.

Again thanks for your replies and I hope this clarifies what we are trying to achieve. And I hope it sounds interesting from your point of view.

**Answer:** Thank you for your answers.

Here is some more feedback from the development team:

Long term preservation of architectural knowledge sounds interesting and I’m glad Autodesk tools are used for this!

As far as I understood, the input format is E57, which ReCap can index and so can Revit (using its standalone indexer).

I am puzzled by the client’s desire not to use ReCap for indexing. I would ask if his scans are already 'registered', i.e. unified in one E57 file per building.

If so, then I understand his desire to write a script to automate E57 to RCS conversion of the large number of files.
Our standalone indexer can be used for that; we had this use case in mind from the beginning, although I don’t know if anybody is actually using such a workflow.

I found the following description:

The indexer program is called AdPointcloudIndexer.exe and is located in the Program folder in Revit installation.

The command line for the indexer is:

> `AdPointcloudindexer [/batch] [/pluginspath=PathToPluginsFolder] InputFile [OutputFile]`

Where:

- InputFile: must be specified; all other parameters are optional;
- OutputFile: If specified is the resulting RCP file from the conversion. If no file is specified, then the resulting file is same base filename as the input file, and placed in the same folder as the input file. If the output file already exists, a prompt will be provided asking the user if they wish to overwrite it or not.

/batch causes the following to be true:

- No progress will be reported as the indexer operates.
- No end of process reporting will be done.
- If the output file already exists, an error is generated to stderr and the indexing terminated.

The /pluginspath argument specifies the folder to look for plugins, and defaults to a folder named 'plugins\Point Clouds Codecs' under the folder where the indexer exe resides.

The thread discussing
[batch PCG creation using Autodesk point cloud indexer](http://forums.autodesk.com/t5/revit-architecture/batch-pcg-creation-using-autodesk-point-cloud-indexer/td-p/3238878) also
refers to this usage.