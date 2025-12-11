---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.379105'
original_url: https://thebuildingcoder.typepad.com/blog/0676_api_wikihelp.html
post_number: '0676'
reading_time_minutes: 4
series: general
slug: api_wikihelp
source_file: 0676_api_wikihelp.htm
tags:
- csharp
- revit-api
- vbnet
- views
- windows
title: Revit API Developer Guide on Autodesk WikiHelp
word_count: 849
---

### Revit API Developer Guide on Autodesk WikiHelp

Step by step, online help is ever more prevalent.
A huge step forwarded was achieved with the Revit 2012 product
[WikiHelp system](http://thebuildingcoder.typepad.com/blog/2011/04/revit-2012-wikihelp-overview.html) with
its support for community building and contribution, and the
[Revit API documentation](http://thebuildingcoder.typepad.com/blog/2011/05/wiki-api-help-view-event-and-structural-material-type.html#1) soon followed suit.

The Content Wrangler recently published an interesting
[article and interview](http://thecontentwrangler.com/2011/10/25/in-search-of-operational-efficiency-a-discussion-with-victor-solano-autodesk) discussing the topic in more depth.

Following up on this, here is some more detailed information and suggestions specifically targeted at the Revit API help by Mikako Harada, ADN AEC Technical Lead:

#### Autodesk Revit API Developer Guide on Autodesk WikiHelp

As some of you are already aware, we now have an Autodesk Revit API Developer Guide on Autodesk WikiHelp.
You can access it from
[www.autodesk.com/revitapi-help](http://www.autodesk.com/revitapi-help).
This is the place where you will find the latest, up-to-date Revit developer guide.
Of course, Wiki means this is the place where we all can contribute and become a part of the community.

To get you motivated to look at our Revit API Developer Guide on WikiHelp, and to encourage you to start contributing, we have picked up the following two topics to share with you:

- ['My page'](#2)- [Formatting Code Samples](#3)

More to follow...

#### 'My page' – It's For You

Did you know that you can create content under your own page? Every user has their own location in the WikiHelp, in which you can add pages and edit content. Here is how you can access it:

1. Go to [wikihelp.autodesk.com](http://wikihelp.autodesk.com)- Sign in using Autodesk login ID ('Sign in' button is at the top right):

   ![Wikihelp sign-in](img/wikihelp/image003.jpg)

   - Go to your  at the top right corner of the page, and click 'My page'. This will bring you to your page:

   ![My page](img/wikihelp/image005.gif)

   - Once you are in 'My page', you can Create a new page and Edit page using various edit functions:

   ![Create new page](img/wikihelp/image007.gif)

The image below shows an example of how it looks like when you go to 'Edit page' > 'Edit content' mode. You can easily figure out what each basic function does from icons.

![Editing](img/wikihelp/image009.gif)

As mentioned earlier, all users have the 'My page' location and it is intended as a public page.
For example, on the landing page of the WikiHelp, you can click on any of the user names of the 'Top Contributors' to see their My page profile.

If you are not yet familiar with the WikiHelp environment, this is a good place to start playing with WikiHelp and learn how to write a page until you feel comfortable modifying the existing content.

#### Tips for Formatting Code Samples

If you are a programmer thinking about contributing to any of the Developer Guide section of WikiHelp, you will most likely find formatting in WikiHelp not as straightforward as you wish.
Soon you will start wondering what the best way to add a code sample is.
(In fact, that's what we struggled with when we started!)

Here is one suggestion that we want to share with you to make formatting code less painful:

1. Copy and Paste a code snippet to the Edit window.
   For example, you can copy and paste directly from your Visual Studio IDE to the Wiki page:

![Copy and paste code](img/wikihelp/image011.jpg)

2. Select the whole code area, and then set the [Formatting Styles] to 'Formatted'.

![Formatted code](img/wikihelp/image013.jpg)

3. Place a cursor anywhere within the formatted text.
   Once a block of text is formatted, [Transformations] button becomes active.
   ([Transformations] icon is rather shy looking.
   It's a tiny grayish icon at lower left corner of the tool bar area.)
   Choose the language of the code from the pull-down menu (i.e., 'syntax.CSharp' for C# in our case. Choose 'syntax.Vb' for VB.NET):

![Select source code language](img/wikihelp/image015.jpg)

At this point, the transformation you set does not have any visual effect.
You will see it only after you save it and exit the edit mode.- [Save] the modification. Your code should look like this after save:

![Formatted code](img/wikihelp/image017.jpg)

You still lose the exact colour coding that you had in your Visual Studio IDE.
But with the current functionality of WikiHelp editing, the above seems to be the easiest.
We, including the Revit API team, have decided to stick to following this way from now on.

We look forward to seeing your contribution in our Revit 'API Dev Guide' section very soon.

Acknowledgement: many thanks to Elizabeth Shulok of Structural Integrators, LL C, ADN member, for her valuable feedback about editing in WikiHelp, and many thanks to Mikako for this helpful summary!