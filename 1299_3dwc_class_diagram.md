---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.713093'
original_url: https://thebuildingcoder.typepad.com/blog/1299_3dwc_class_diagram.html
post_number: '1299'
reading_time_minutes: 5
series: general
slug: 3dwc_class_diagram
source_file: 1299_3dwc_class_diagram.htm
tags:
- revit-api
- views
title: Split Personality and Revit API Class Diagram
word_count: 1050
---

### Split Personality and Revit API Class Diagram

Developers have been requesting a Revit API class diagram for years.

There used to be one back in the dawn of time, up
[until the Revit 2010 API](http://thebuildingcoder.typepad.com/blog/2012/01/no-revit-api-class-diagram.html).

Now the question came up again with a happy resolution in the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread on the
[2014 Revit object model](http://forums.autodesk.com/t5/Autodesk-Revit-API/2014-Revit-Object-Model/m-p/4357581).

Before I get to that, I have an important personal announcement to make:

#### The 3D Web Coder Commenced, Embarked, Blasted Off and Away

In an attempt to clarify the purpose of my being, or
[multiple beings](https://en.wikipedia.org/wiki/Dissociative_identity_disorder),
you will be glad to hear that I split off my schizophrenic cloud and mobile
[alter](https://en.wikipedia.org/wiki/Alter_ego)
into a new blog,

[The 3D Web Coder](http://the3dwebcoder.typepad.com)

Many of the
[cloud](http://thebuildingcoder.typepad.com/blog/cloud) and
[mobile](http://thebuildingcoder.typepad.com/blog/mobile) related
topics that I discussed here in the past will be handled there by that part of me from now on.

So far, The 3D Web Coder is still getting started and looking at things like this:

- [The web enabled Brackets editor](http://the3dwebcoder.typepad.com/blog/2015/03/the-brackets-editor.html)
- [The Node.js server platform](http://the3dwebcoder.typepad.com/blog/2015/03/the-nodejs-server-platform-icons-3d-and-the-future.html)
- [A JavaScript function to process query strings](http://the3dwebcoder.typepad.com/blog/2015/03/preparing-for-processing-query-string-server-arguments.html)
- [Processing query strings in JavaScript and Node](http://the3dwebcoder.typepad.com/blog/2015/03/processing-query-strings-in-javascript-and-node.html)

The next step will be to run a server live and accessible somewhere on the big wild worldwide web, e.g., using a service such as
[Heroku](http://www.herokuapp.com).

Once I have that in place, I can move on more interesting – e.g., graphical – aspects such as using them to drive 2D and 3D graphics.

For 2D, think exporting a Revit floor plan to a web server displaying it using SVG in conjunction with relevant associated metadata.

Obviously, these two alters will have to talk with each other, at least now and then.

I wish both my alters all the best and lots of fun exploring and documenting their respective domains!

#### The Revit API Class Diagram

Back to the Revit API and the
[Revit API object model](http://forums.autodesk.com/t5/Autodesk-Revit-API/2014-Revit-Object-Model/m-p/4357581) discussion thread:

**Question:**
I am getting back into programming using the Revit API. The last version of the API that I used was 2010. Back then there was a graphical representation of the Revit Object Model in PNG format (Revit API Class Diagram.png). I have not been able to locate one for the Revit 2014 Object model. Does anyone know if one exists?

**Answer:**
I am sorry, we haven't provided an API class chart for years. The Revit API has been growing rapidly in the past years and it became impractical at some point to try to fit everything in one single chart.

The Revit 2010 version is indeed the last one provided officially.

The Building Coder discussed the fact that
[no Revit API class diagram](http://thebuildingcoder.typepad.com/blog/2012/01/no-revit-api-class-diagram.html) is
available, explaining more details, what tools to use instead, and how you might be able to create one for yourself.

**Response:**
Call me a little crazy but I actually like these things (visual diagrams of API).
For a person who doesn't know what exactly they are looking for in an API to do a certain function or item then it's really nice to quickly glance and be able to have a visual of how things are structured.
So the visual person in me likes them and can remember something on a diagram like this, versus going through text documents, a lot easier.
I know it's a lot, but I have used the Inventor object model chart for programs on numerous occasions.

**Answer:**
I did a quick check on the Internet and came up with an
[enhanced version of AutoDiagrammer](https://sachabarbs.wordpress.com/category/autodiagrammer-my-reflector-addin),
mentioned in the discussion listed above, plus a neat suggestion on stackoverflow on
[how to get a nice class diagram for built-in .NET classes](http://stackoverflow.com/questions/8152000/how-to-get-a-nice-class-diagram-for-built-in-net-classes) using
built-in Visual Studio tools, as far as I understand.

Would you like to try it on the Revit API and let us know what you come up with?

**Response:**
I did a quick test of the approach described in the link Jeremy suggested.
I can confirm that this works in Visual Studio 2013 – although I had to drag and drop from a class view instead of the object browser.

I've attached the result generated after a couple of 'expand/layout diagram':

- [ClassDiagram\_2015\_Revit\_DB.pdf](zip/ClassDiagram_2015_Revit_DB.pdf)
- [ClassDiagram\_2015.zip](zip/ClassDiagram_2015.zip)

When I zoom the PDF to about 1600% it's readable, but I'm sure the layout can be improved with a little more effort   :-)   (It covers the 2015 Revit.DB namespace, by the way).

**Answer:**
Well done! Thank you for trying it out!

I added it to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).

The Revit.DB namespace class diagram is available as
[ClassDiagram\_2015\_Revit\_DB.cd](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/ClassDiagram_2015_Revit_DB.cd) in
[release 2015.0.120.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.120.2) and later.

It really is very hard to navigate and read, though:

![Revit.DB namespace class diagram](img/ClassDiagram_2015_Revit_DB.png)

The image above is showing less than a tenth of the width.

Now we just need a volunteer to format it nicely...

Many thanks to Peter
([pjohan13](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/2663907))
for jumping in again and trying this out!