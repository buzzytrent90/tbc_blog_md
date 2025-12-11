---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.471817'
original_url: https://thebuildingcoder.typepad.com/blog/1184_survey_blog_sg_dynamo.html
post_number: '1184'
reading_time_minutes: 6
series: general
slug: survey_blog_sg_dynamo
source_file: 1184_survey_blog_sg_dynamo.htm
tags:
- csharp
- elements
- family
- geometry
- levels
- python
- references
- revit-api
- views
title: Wishlist, Blogging, Smartgeometry, Dynamo and FormIt
word_count: 1235
---

### Wishlist, Blogging, Smartgeometry, Dynamo and FormIt

Here is today's bunch of exciting topics:

- [Revit API wishlist survey results](#2)
- [Blogging tips and tricks](#3)
- [Smartgeometry 2014 conference](#4)
- [Dynamo enhancements](#5)
- [Dynamo and FormIt win Best in Show at AIA 2014](#6)

Completely lacking hardcore API stuff, for a change.

#### Revit API Wishlist Survey Results

The results of the
[Revit API wishlist survey](http://thebuildingcoder.typepad.com/blog/2014/05/revit-api-wishlist-survey.html) are
in.

Most participants:

- Are shipping a Revit add-in used in production
- Work in architectural design
- Automate repetitive tasks
- Ask for more access to 3D model elements and enhancements to the Revit geometry API for geometry creation and analysis

#### Blogging Tips and Tricks

Here are some basic tips and tricks that I consider essential for taking the first steps striving towards ultimate perfection in blogging:

- Spell check your text before publishing, and proof read it, preferably several times over, and at least once before and once after publishing.
  I almost always notice new additional enhancement possibilities after publication.
- Ensure that all source code is well formatted and preferably colour coded.
  You can use
  [google-code-prettify](http://code.google.com/p/google-code-prettify) for most languages.
  It obviously cannot parse the .NET libraries referenced by VB or C# code, though.
  For that, I use the J.T. Leigh [CopySourceAsHtml](http://copysourceashtml.codeplex.com/) utility inside my Visual Studio project environment.
  For more details, please refer to various discussions of The Building Coder
  [source code colourizer](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.36).
- Image justification: place small images inline with the text as you please. Large images that take the major part of the column width are best centred.
- Strive for consistency.
  For instance, if you have a paragraph marked 'Question:' and another marked 'Answer:', ensure that the two markers are formatted similarly.
  They can either live on an own line each, or be prefixed to the paragraph they mark.
  They can be either bold or not.
  Just ensure they are recognisably similar, to make the logical structure of your text as clear as possible.
- Do not mix fonts.
- If you mention any classes or methods, be absolutely sure they exist.
  For instance, when talking about the Revit API FamilyType class, you should not make any mention of FamilyTypes.
  If you need a plural, use normal words instead, e.g. 'family types'.
- Watch out for using special HTML characters.
  For instance, the ampersand character '&' has a special meaning in HTML.
  To use it in your HTMP text, you have to escape it using the special HTML entity `&`.
- Use consistent capitalisation in your titles.

#### Smartgeometry 2014 Conference

Autodesk is sponsoring the
[smartgeometry.org](http://smartgeometry.org)
[2014 conference](http://smartgeometry.org/index.php?option=com_content&view=article&id=239&Itemid=199) in
Hong Kong and will play a key role in the BIM workshops there with a special focus on significant advancements in the open source
[Dynamo](http://dynamobim.org) technology.

![](img/sg2014.png)

For more details on this, please refer to the official Autodesk press release on
[sponsoring Smartgeometry 2014 and advancements for Dynamo](http://inthefold.autodesk.com/in_the_fold/2014/07/autodesk-sponsors-smartgeometry-2014-and-spotlights-latest-advancements-for-open-source-dynamo.html).

#### Dynamo Enhancements

[Dynamo](http://dynamobim.org) provides a visual programming environment accessible to designers, allowing them to visually create logic that drives the geometry and behaviour of elements and data created within Autodesk Revit. A dedicated team at Autodesk actively participates in the Dynamo open source community.

![](img/sg2014_1.jpg)

Visual programming tools like Dynamo can be used to script and quantify anything from furniture to buildings and bridges.

New enhancements for Dynamo include:

- Significant refactoring of the underlying code to help users expand the basic Dynamo capabilities.
  The new code base was rebuilt from the ground up to encourage exploratory programming, readability and scaling to very large datasets.

  ![Surface Modelling in Dynamo driving Revit elements](img/sg2014_2.jpg)

  Surface Modelling in Dynamo driving Revit elements
- Expansion of geometric capabilities: Dynamo's new library of surface and solids tools is built on the same code as Autodesk Fusion and Inventor, enabling the combination of the flexibility and versatility of industrial design and manufacturing tools with the rational and exploratory power of computation.

  ![Nodes are nice, but sometimes people want to write code.](img/sg2014_3.jpg)

  Nodes are nice, but sometimes people want to write code.
- Rich new set of tools for scripting: In addition to Python programming language support in dedicated nodes, Dynamo now allows for
  [direct input of compact and readable code](http://dynamobim.org/cbns-for-dummies) into
  the graph.
  Users can script small or large pieces depending on their comfort level, and interact with the text-based code on the same level as the graph environment.

  ![Dynamo driving an ABB Robot arm after importing the manufacturer's code library.](img/sg2014_4.jpg)

  Dynamo driving an ABB Robot arm after importing the manufacturer's code library.
- Library Import: Enables the direct loading of additional tools (such as analysis and extra modelling capabilities) from user-authored libraries of code.
  This helps increase the rate at which users can expand the functionality of Dynamo to fit their needs and share workflows with others.
  We expect more development here from 3rd party contributors.
  For example, check out how participants at a [hackathon](http://enr.construction.com/video/?video_uuid=c136x005&categoryId=39093) used this functionality to drive robots!

Again, for more details, please refer to the official Autodesk press release on
[sponsoring Smartgeometry 2014 and advancements for Dynamo](http://inthefold.autodesk.com/in_the_fold/2014/07/autodesk-sponsors-smartgeometry-2014-and-spotlights-latest-advancements-for-open-source-dynamo.html).

#### Dynamo and FormIt Win Best in Show at AIA 2014

The Autodesk conceptual design team responsible for Dynamo also works on FormIt.

Both of these applications won the
[Architosh's 'Best in Show' award](http://architosh.com/2014/07/aia-architosh-awards-aia-national-best-of-show-honors-to-software-and-technology-vendors) for
desktop and mobile respectively at the
[AIA convention 2014](http://convention.aia.org) in Chicago.

![Architosh desktop award](img/sg2014_5.jpg)

'[Dynamo](http://dynamobim.org) continues to evolve into an exciting option for visual programming with 3D geometry components and this latest release in alpha runs in stand-alone mode no longer requiring Revit,' says Anthony Frausto-Robledo, LEED AP. 'Dynamo .07 alpha also runs on the Autodesk Shape Manager (ASM) geometry modelling kernel and contains a new scripting interface. Visual scripting and parametric modelling platforms like Dynamo will increasingly serve the field of architecture,' adds Frausto-Robledo, 'as more analytics service the iterative architectural design process and building design in general becomes more performance-based.'

![Architosh mobile award](img/sg2014_6.jpg)

'[Autodesk FormIt](http://autodeskformit.com) won BEST of SHOW last year in this category and since that release has continued to improve nicely,' remarks Frausto-Robledo, AIA LEED AP. 'It remains an exemplar of the combined power of mobile and the cloud with its integrated Autodesk 360 cloud connections and utilization of Google's maps technology to locate and use project sites. In the latest release it also contains early stage building performance capabilities tapping into local climate station data. This is quite exciting to see front-ended analytics right at the site, it is a tremendous example of the power of tablets connected to the cloud.'

####

```csharp
```