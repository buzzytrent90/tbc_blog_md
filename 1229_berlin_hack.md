---
post_number: "1229"
title: "Berlin Hackathon"
slug: "berlin_hack"
author: "Jeremy Tammik"
tags: ['elements', 'references', 'revit-api', 'selection', 'views', 'windows']
source_file: "1229_berlin_hack.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1229_berlin_hack.html"
---

### Berlin Hackathon

I arrived safe and sound for the Berlin hackathon. Topics for today, three on the Revit API and three not:

- [Traffic jams and too many cars](#2)
- [Intro to functional programming in JavaScript](#3)
- [Aligning multiple elements](#4)
- [Render to PDF](#5)
- [Displaying transient graphics in a preview control](#6)
- [Berlin hackathon beginning](#7)

#### Traffic Jams and Too Many Cars

Yesterday, on a Friday afternoon – bad timing – my colleague Peter Schlipf and I travelled from Darmstadt to Berlin by car, about 600 km.

It took us eight hours, with several accidents and traffic jams on the way.

I gratefully acknowledge that I do not own a car myself.

I also gratefully realise that all four of my kids have no car either, in spite of – or not in spite of, as an aspect of? – all being totally hip and professional.

It seems to me that more people in Germany travel by car than in Switzerland.
I hope that this kind of individual locomotion is in decline.

I'll try to take the train next time.
I obviously also want to avoid flying, since that is ecologically catastrophical.

I am also grateful to have the luxury of hardly ever needing to use a mobile phone... and not owning a smartphone... although I do admit to rather a lot of Internet activity... :-)

#### Intro to Functional Programming in JavaScript

One thing I am looking forward to when I get back home is the
[intro to functional programming in JavaScript](http://www.meetup.com/basel-js/events/213653852/?a=uc1_vm&read=1&_af_eid=213653852&_af=event) meetup
coming up in Basel, a hands on workshop presented by Lukasz Gintowt to "learn functional techniques to develop and debug much faster, learn to produce clean and testable code and become a JavaScript wizard".

Lukasz presents a js framework he developed for real-time distributed calculations.
He realized that there were some core concepts in the framework that were really important for people to understand beforehand, namely functional programming techniques. Et voila – this prequel was born.

Here are a few links to both easy and serious reading on this topic:

- [Why functional programming matters](http://worrydream.com/refs/Hughes-WhyFunctionalProgrammingMatters.pdf) by John Hughes
- [Can Programming Be Liberated from the von Neumann Style? A Functional Style and Its Algebra of Programs](http://web.stanford.edu/class/cs242/readings/backus.pdf) – classic paper by John Backus
- [Imperative Functional Programming](zip/imperative_functional_programming.pdf) ([original](http://research.microsoft.com/pubs/67066/imperative.ps.z)) – research paper by Simon L. Peyton Jones and Philip Wadler.
- [Why functional languages?](http://stackoverflow.com/questions/36504/why-functional-languages) – functional properties presented like lazy evaluations are universal and always relevant.
- [Haskell tutorial](http://learnyouahaskell.com/chapters) – Haskell is a purely functional programming language.
- [Execution in the Kingdom of Nouns](http://steve-yegge.blogspot.ch/2006/03/execution-in-kingdom-of-nouns.html) – allegorical JavaScript paradigm criticism
- [Object Oriented Programming is Inherently Harmful](http://harmful.cat-v.org/software/OO_programming/) – OO bashing.
- [Bad Engineering Properties
  of Object-Oriented Languages](http://doc.cat-v.org/programming/bad_properties_of_OO) – more OO bashing.
- [Singleton Considered Stupid](https://sites.google.com/site/steveyegge2/singleton-considered-stupid) – pattern bashing.

#### Aligning Multiple Elements

**Question:** I have a product referred to as a Panel that consists of a top cap, end cap, tiles and frames. Each of these are individual elements in the Revit model. For end users, though, all of these comprise of a single Panel.

I need to align this panel (including all the pieces) using the Align post command in one command – meaning I cannot ask users to select top cap, align, then select tile, align, etc.

The Align post command works with one element at a time and does not support window select. Even if we had all the top cap, end cap, tiles, frames, etc. of this panel selected in one click, on Align command launch, it deselects all the selection and prompts users to select reference plane/line and then each individual element.

Is there any way to align all the selected elements that comprise of a panel in single align command? I was thinking of grouping these together before align, and after align finishes, ungrouping them – but since this is done using post commands, it will group, ungroup and then call post command at the end.

How might I achieve this programmatically?

**Answer:** You could group the elements, post the command, and then use an updater to ungroup, I suppose.

If the user never created the alignment the updater would not be called, that makes this a bit tricky to manage.

#### Render to PDF

**Question:** I would like to achieve the following steps using the Revit API.

1. Rendering the current 3D view.
2. Save the result to the project.
3. Print out the saved result as a PDF.

Could the CustomExporter class be used for the rendering part?

**Answer:** The answer depends on what you mean by 'rendering'. If you mean a process of which result is a rendered image of a 3D view, like if the Render command in Revit is used, then the answer is 'No' – the Custom Exporter cannot do that. What the Custom Exporter does is hand over all graphic entities visible in a 3D view to the API caller who invoked the Custom Exporter Execute method. It is up to the caller to decide what to do with the information. They can use it to produce a file (e.g. SVG, WebGL, PDF, etc.) or use it to create an image, assuming they have access to some custom render engine.

#### Displaying Transient Graphics in a Preview Control

**Question:** My add-in uses the PreviewControl class. I can show a normal 3D view in it, of course. I also want to draw other 3D data such as planar polygons in the PreviewControl in my dialogue to simulate something.

Is there currently any way to do that?

**Answer:** The Revit API provides two options for displaying temporary transient graphics in any view, including the Preview control:

1. The Analysis Visualization Framework – if you can create the shape, it can draw it, use the designated colours etc. This is transient and not saved with the document. It does have limitations on how it can be interacted with.
2. The DirectShape element introduced in Revit 2015 – this can be any desired 3D shape, with any material. This is saved with the document so the developer would need to manage its lifetime if they don't want it saved.

#### Berlin Hackathon Beginning

Back to here and now, the Berlin hackathon just started, with the hashtag
[#TMUHack](https://twitter.com/hashtag/tmuhack).
Here are Peter Schlipf and Cyrille Fauvel presenting once again on the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46):

![Peter Schlipf at TMUHack](file:////j/photo/jeremy/2014/2014-10-25_berlin_hackathon/616_peter_schlipf_tmuhack_cropped.jpg)

Here are some of the ideas being worked on:

- Matthias – text api work cloud, upload article, generate work cloud banner on top of article, click on it to get more info.
- Pawan, india – shopping last minute delivery crowd shipping, people collaborate for shopping, shop for each other.
- Achim – financial services, reinvent banking, real-time, mobile, account statement, plus predictive analysis.
- Jan, keyrocket – with stephan, text api.

1. Movie memory game – summaries of movies, match with movie cover, multiplayer game, one card is dvd cover, the other is data extracted from the summary.
2. Style guide – style guide, trawler, random web sites, get all style elements and eliminate less used styles.

- Stephen – text api plus autodesk, datascapes, human interaction, compare real-time data for user, they need a service or product, pull info from internet and generate profile.
- Alex – quiz generator, test learning, editor.

I'm working with the [MovieMemory](http://m3my.github.io) team...