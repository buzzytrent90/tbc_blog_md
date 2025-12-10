---
post_number: "0468"
title: "Creating a Solid"
slug: "create_solid"
author: "Jeremy Tammik"
tags: ['family', 'geometry', 'revit-api', 'views']
source_file: "0468_create_solid.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0468_create_solid.html"
---

### Creating a Solid

Here is question that crops up from time to time and requires pretty special treatment in Revit:

**Question:** Is it possible to build 3D solids in Revit like we can in AutoCAD?
For instance, when you have a bunch of points and you wish to make a mass object similar to an extrusion?

**Answer:** The very words "as in AutoCAD" are extremely dangerous to utter or even think when using the Revit API, so for your own sake and sanity, please try to avoid that :-)

On the other hand, you can create a lot of solid geometry in Revit, albeit in the family context.

Creating solids in the project context would require the creation of in-place families, and that is currently not supported through the API.

To learn more about this, the first step is to understand well the difference between the project and family creation context from a user interface point of view.

Then, for the programmatic approach, please look at some of the various presentations that have been given on
[form creation and conceptual design](http://thebuildingcoder.typepad.com/blog/2009/07/revit-form-creation-api.html).

Zach Kron runs a very impressive blog completely dedicated to both manual and programmatic generation of conceptual design geometry,
[practical notes on impractical things](http://thebuildingcoder.typepad.com/blog/2009/07/practical-notes-on-impractical-things.html).

Harry Mattison of the Autodesk Revit development team gave several presentations on this topic, both at Autodesk University and the AEC DevCamp conferences in 2009 and 2010.

His talk at AEC DevCamp 2010 was named '2.3 Creating Analyzing Conceptual Massing Geometry with Revit API'.
ADN members can access this material by searching on the ADN web site for 'aec devcamp 2010' and selecting the link 'Autodesk DevCamps 2010' to obtain the material.
Here is a
[direct link](http://download.autodesk.com/media/adn/Autodesk_AEC_DevCamp_2010.zip) as well.

To obtain the material for Harry's latest AU class, I entered the search string 'Autodesk University conceptual' in Google and found
[Creating and Analyzing New Conceptual Massing Geometry with the Autodesk Revit API](http://au.autodesk.com/?nd=class&session_id=5252).

As said, this is all within the family creation context, i.e. using the Family API portion of the Revit API.
We presented webcasts to provide an overview of the entire
[Family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html) and
I also already discussed the accompanying
[family creation API labs](http://thebuildingcoder.typepad.com/blog/2009/10/revit-family-creation-api-labs.html).