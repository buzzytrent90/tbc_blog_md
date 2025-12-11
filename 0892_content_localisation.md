---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.809138'
original_url: https://thebuildingcoder.typepad.com/blog/0892_content_localisation.html
post_number: 0892
reading_time_minutes: 5
series: general
slug: content_localisation
source_file: 0892_content_localisation.htm
tags:
- elements
- family
- filtering
- levels
- parameters
- revit-api
- schedules
- views
title: Content Localisation
word_count: 955
---

### Content Localisation

How to manage families and content creation is a crucial topic for a large number of Revit add-ins.

Brian Johnson, technical specialist on structural engineering solutions at Autodesk in Austin, Texas, discusses how to efficiently address some important aspects of this below.

First, however, let me wish everybody a Happy New Year of the Snake!

![Year of the Snake](img/year_of_the_snake.jpeg)

A year for reflection, planning and searching answers.
A good time for shrewd dealings, political affairs and coups d'état.
People will be more likely to scheme and ponder over matters before acting on them.
An auspicious year for commerce and industry.

Let's hope so, indeed, and return to the issue of content localisation:

#### Content Localisation

**Question:** We have a large number of library items that we would like to work with in Revit, and they are used in many different countries and languages.

How can we manage this efficiently, please?

We would like to avoid duplicating any of the content, of course.

The differences include things like product name, description, part code, cost, and many other properties.

As far as I see, a Revit family cannot hold different values for a single property such as the item description.

What are your advice and the best practice for creating and maintaining a Revit family library for thousands of items and numerous languages?

**Answer:** Just as you say, Revit is not equipped to localise parameter names and types nor family names themselves.

Some level of automation will probably help significantly to solve or at least simplify this task.

A proof-of-concept for generating content from data and organizing content can be found in the Content Generator Extension available from the Autodesk Subscription centre.
Note that all the Revit Extensions are bundled into a single download:

![Content Generator Extension](img/localise_content_1.jpeg)

The content generator is intended for structural content from around the world using the section and material libraries (XML files) that also form the libraries for Robot Structural Analysis Professional and AutoCAD Structural Detailing:

![Content Generator Extension](img/localise_content_2.png)

I describe some of the inner workings of the Content Generator on pages 7-9 in the hand-out of my Autodesk University 2012 course
[SE3904 – Linking Revit Structure and Robot Structural Analysis: Beyond the Basics](http://au.autodesk.com/?nd=class&session_id=10570).
For your convenience as a valued reader of The Building Coder, here is a
[direct download link](zip/localise_content_se3904_handout.pdf).

I suggest installing the Extensions and trying it out.
Your version of this might pull country specific data from your libraries/databases and generate a specific element for a specific language.

When considering creating your own custom families, my suggestion is to create the content with the best-suited family.
If you use a specialised category like Structural Framing, you will get built-in parameters and behaviours that you may or may not want.
The Generic Model set of family templates is the most basic and may be the most appropriate.
I suggest populating the Identity Data and creating Shared Parameters for each family type and use those to filter, sort, and group the Schedule views in Revit.

As always, it is of utmost importance to discuss your plans with some of your Revit users and to get some training on how things are done in the typical workflows in Revit.
This will help you understand what can be done with Revit out-of-the-box and what parts might use more automation.

Thank you, Brian, for the valuable advice!

#### MEP Family Editor Part Type Classes

While we are talking about content creation, here is another MEP specific question on the different classes of fitting types:

**Question:** I would like to know the definition for some of the part types of the fittings in the family editor.

Simple classes of fitting types like cap, cross, elbow, multiport, tap, transition, tee and union is clear, but I am uncertain about the more complex and less clear-cut parts.

Could you please clarify the following?

1. What is the difference between wye and pants?- Lateral tee and lateral cross are probably tees with an angle other than 90 degrees. They can also be rounded. Is this correct?- Offset is probably a kind of S-bend. Is this correct? Are eccentric transitions also included in this group?

**Answer:** Sure:

**1.** What is the difference between wye and pants?

It is no difference between creating wye and pants families.
We just choose a part type that is close to the family name, which will makes sense if users check the part type.

Examples for wye:

![Wye](img/mep_part_type_classes_1.jpeg)

Examples for pants:

![Pants](img/mep_part_type_classes_2.jpeg)

**2.** Lateral tee and lateral cross are probably tees with an angle other than 90 degrees. They can also be rounded. Is this correct?

Again, lateral tee and lateral cross are used when the family name has similar words.
The rule for using 'Lateral tee', 'Lateral cross' and 'Wye' is that the branch angle will be fixed when using the family in the project, so if the branch pipe has a different angle with the fitting, an elbow will be automatically added.
If the part type is 'tee' or 'cross', the fitting will have a flexible angle branch itself.

Examples for lateral tee:

![Tee](img/mep_part_type_classes_3.jpeg)

Example for lateral cross:

![Cross](img/mep_part_type_classes_4.jpeg)

Lateral tee and lateral cross are also used as part types for pipe fittings, so they can be round as well.

**3.** Offset is probably a kind of S-bend.
Is this correct?
Are eccentric transitions also included in this group?

Examples for offset:

![Offset](img/mep_part_type_classes_5.jpeg)

An eccentric transition should use the 'Transition' part type.