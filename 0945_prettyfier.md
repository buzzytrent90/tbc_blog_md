---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: news
optimization_date: '2025-12-11T11:44:14.964156'
original_url: https://thebuildingcoder.typepad.com/blog/0945_prettyfier.html
post_number: 0945
reading_time_minutes: 3
series: general
slug: prettyfier
source_file: 0945_prettyfier.htm
tags:
- csharp
- references
- revit-api
- rooms
- views
title: Source Code Formatting and Google Prettifier
word_count: 596
---

### Source Code Formatting and Google Prettifier

As you know, I format my source code to pretty short lines in order to avoid having them truncated by the narrow blog post view column.

I also like to present the code colour coded, as it appears in Visual Studio and many other programmer editors, to make it more readable.

For .NET code, I use
[CopySourceAsHtml](http://thebuildingcoder.typepad.com/blog/2011/04/updated-sdk-2012-products-and-source-code-colourisation.html#4) inside of Visual Studio for that.

I tried using other tools outside of Visual Studio instead in the past, including building my own, but they have one big disadvantage: unless they read and analyse all the referenced .NET assemblies to determine the classes they define, they cannot always tell whether a given word represents a variable or a class.
Classes are highlighted in a different colour in Visual Studio, and I find that pretty helpful.

For other languages, though, it would be really nice to be able to colourise the source code independently.

My colleague Cyrille Fauvel now pointed to two online colourising tools that he has used: the
[syntax highlighter](http://alexgorbatchev.com/SyntaxHighlighter) and
[Google prettify](https://code.google.com/p/google-code-prettify).
Both of them are not really useful for C#, for the reasons explained above, but do a really good job on other languages.

I tested the Google prettifier on some JavaScript, HTML and JSON code in my recent post on
[my cloud-based editor home page implementation](http://thebuildingcoder.typepad.com/blog/2013/05/my-cloud-based-2d-editor-implementation-status.html#5) and
am very happy with the results.

For instance, here is a screen snapshot of the main JavaScript snippet before integrating the prettifier:

![Before Google Prettify](img/prettify_before.png)

Afterwards, it looks like this instead:

![After Google Prettify](img/prettify_after.png)

I only have to do two things to achieve that.

1. Add a reference to the Google Prettify loader:

```
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js">
</script>
```

2. Add the 'prettyprint' class to my HTML 'pre' tags:

```
<pre class="prettyprint">
```

That's not much :-)

It comes in really handy right now, since I will be publishing more JavaScript, HTML and JSON for the final stages of my
[cloud-based 2D room editor](http://thebuildingcoder.typepad.com/blog/2013/05/my-cloud-based-2d-editor-implementation-status.html#2) in
the next few days.

Many thanks to Cyrille for pointing this out!

#### Think Global, Act Local, Control Freak

After mulling over the above during the night, I decide to take control myself rather than go off and ask Google for help to render every page I post (and pass them every snippet of code to mine for analysis purposes, by the way).

So I downloaded the minimised version of the Google prettifier and now serve it up locally from The Building Coder typepad page itself.

In other words, I include the following script load statement instead of the one listed above:

```
<script src="http://thebuildingcoder.typepad.com/google-code-prettify/run_prettify.js">
</script>
```

Beyond that, nothing changes.

#### More Magic

Oh yes, and another magical little thank you to Cyrille for pointing out the Apple Magic Mouse to me.

I have been using it for a few days and am enthused.

I first thought it was a bit too slim for my chunky hand, but that is not the case, and I really love the perfect smoothness and full control it gives, better than any other system I tried.

Thank you again, Cyrille :-)