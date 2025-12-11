---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.794654'
original_url: https://thebuildingcoder.typepad.com/blog/1335_responsive.html
post_number: '1335'
reading_time_minutes: 2
series: general
slug: responsive
source_file: 1335_responsive.htm
tags:
- references
- revit-api
- sheets
title: The Responsive Building Coder
word_count: 339
---

### The Responsive Building Coder

I suffer from acute and chronic responsiveness.

That is unpleasant for me, although hopefully pleasing and useful for the rest of humanity.

What I mean is, I tend to answer other people's questions and needs before taking care of my own.

I am forever trying to stop doing that.

One of these days, I certainly will.

The topic I refer to above is not me, personally, though, but The Building Coder blog.

Nowadays, web pages should be based on a
[responsive web design](https://en.wikipedia.org/wiki/Responsive_web_design) or
their Google search ranking will suffer.

Kean Walmsley explained the
[growing usage of mobile devices](http://through-the-interface.typepad.com/through_the_interface/2015/02/reading-from-mobile-devices-time-for-a-redesign.html), leading to the recent
[responsive redesign of Through the Interface](http://through-the-interface.typepad.com/through_the_interface/2015/02/reading-from-mobile-devices-time-for-a-redesign.html).

Following his lead, I switched The Building Coder to a responsive design now as well, as you can see.

I hope you like it!

Just like Kean's blog, I use the Typepad 'Snap' template with Lato and Open Sans fonts.
The latter requires a reference to the Google font stylesheet.

I struggled a bit with the banner image, especially to suppress the default header text that Typepad generates automatically and puts on top of the image.

I finally found the following simple CSS solution for that:

```
.jumbotron h1, .jumbotron h2 { display: none }
```

The next main issue I had was with the Microsoft Bing translation widget.

I set it up on their web page and added the resulting HTML and JavaScript to my blog design.

It did weird things, and even after significant tweaking was still displaying two user interfaces on top of each other.

After too much of that mess, I opted for the Google translation service instead.

I hope you like the result, especially if you are on a mobile device.

Please do let me know what you think. Thank you!