---
post_number: "0575"
title: "Updated SDK and Colour Coding Source Code"
slug: "updated_sdk"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'python', 'revit-api', 'sheets', 'views', 'windows']
source_file: "0575_updated_sdk.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0575_updated_sdk.html"
---

### Updated SDK and Colour Coding Source Code

Here are several miscellaneous news items that I would like to pass on before heading off for a week's vacation:

- [Updated Revit SDK](#2)- AUGI [Autodesk 2012 product overviews](#3)- [HTML source code colourisation](#4)- [Easter greetings](#5)

#### Updated Revit SDK on Developer Center

An updated version of the Revit SDK has been posted on the
[Revit Developer Center](http://www.autodesk.com/developrevit).
The differences to the version included with the initial public release of the Revit 2012 product are negligible.
Still, you can always rely on finding the most up-to-date version on the developer centre.

#### AUGI What's New in Autodesk 2012 Products

AUGI published their
[AUGIWorld March 2011 Issue](http://augi.typepad.com/augi_news/2011/03/augiworld-march-2011-issue-has-been-released.html) with
the topic 'What's New in Autodesk 2012 Products' – a very handy overview of the new product features in many or all of the Autodesk 2012 products.

#### The Building Coder Source Code Colour Coder

Easter is coming up, so colouring is important ... and now that debugging a Revit 2012 add-in requires Visual Studio 2010, and after
[migrating The Building Coder samples](http://thebuildingcoder.typepad.com/blog/2011/04/migrating-the-building-coder-samples-to-revit-2012.html),
I obviously need to be able to copy and paste colour coded source code to HTML for the blog posts from that environment as well.

I have been successfully using
[CopySourceAsHtml](http://copysourceashtml.codeplex.com) for
a long time now to copy and paste colour coded source code to the blog posts.

This utility is available for Visual Studio 2003, 2005 and 2008, but not for 2010.

Luckily,
[Kean Walmsley recently pointed out](http://through-the-interface.typepad.com/through_the_interface/2011/03/finally-working-with-visual-studio-2010.html) an article on
[enabling CopySourceAsHtml in Visual Studio 2010](http://blogs.microsoft.co.il/blogs/applisec/archive/2010/02/25/copyashtml-in-visual-studio-2010.aspx).
Just like Kean, I was able to follow the steps described and get it working without problems.

By the way, Kean's article lists a couple of other useful Visual Studio utilities as well, for navigating large source code modules and driving multi-target C++ compiler versions, i.e. running the Visual Studio 2008 version of the C compiler from the 2010 version of the IDE.

CopySourceAsHtml can be set up to encode the HTML colourisation in various ways.
I leave the default settings and do not embed the styles in the code:

![Copy source as HTML](img/copysourceashtml.png)

Let's say I use CopySourceAsHtml to copy the following four lines of typical Revit add-in source code:
```csharp
    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
```

That produces the following HTML output with a prepended style definition header:
```csharp
<style type="text/css">
.cf { font-family: Courier New; font-size: 9pt; color: black; background: white; }
.cl { margin: 0px; }
.cb1 { color: blue; }
.cb2 { color: #2b91af; }
</style>
<div class="cf">
<pre class="cl"> <span class="cb1">public</span> <span class="cb2">Result</span> Execute(</pre>
<pre class="cl"> <span class="cb2">ExternalCommandData</span> commandData,</pre>
<pre class="cl"> <span class="cb1">ref</span> <span class="cb1">string</span> message,</pre>
<pre class="cl"> <span class="cb2">ElementSet</span> elements )</pre>
<pre class="cl"> </pre>
</div>
```

I then run a little Python script
[pycolorize.py](zip/pycolorize.py)
of mine.
It copies its input from the Windows clipboard, modifies it, and pastes the result back again.
The modification removes the style definition header, since I have the required style definitions already present in my CSS style sheet:
```csharp
.blue { color: blue; }
.red { color: red; }
.teal { color: teal; }
.maroon { color: maroon; }
.green { color: green; }
.gray { color: gray; }
```

It also removes the extraneous div tag, the repetitions of the pre tag, and replaces the cb1, cb2 etc. classes by colour names which correspond to the names of my globally predefined styles, which results in the following much more legible output:
```csharp
<span class="blue">public</span> <span class="teal">Result</span> Execute(
<span class="teal">ExternalCommandData</span> commandData,
<span class="blue">ref</span> <span class="blue">string</span> message,
<span class="teal">ElementSet</span> elements )
```

That is my version of colouring eggs for Easter :-)

#### Happy Easter!

I celebrated the beautiful full moon with some friends last Monday, which was the
[paschal full moon](http://en.wikipedia.org/wiki/Paschal_full_moon) which actually determines the date of
[Easter](http://en.wikipedia.org/wiki/Easter) this year.

Anyway, starting tomorrow,
[Good Friday](http://en.wikipedia.org/wiki/Good_Friday), I will be away for the coming week, taking my Easter holidays, searching for eggs and other happy surprises.

![Easter bunny postcard](img/Easter_Bunny_Postcard_1907.jpg)

In the meantime, I wish you a wonderful time and much success with your Revit add-in development efforts and all other important aspects of life, such as love, peace, and happiness for all beings :-)