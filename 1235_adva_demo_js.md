---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.573110'
original_url: https://thebuildingcoder.typepad.com/blog/1235_adva_demo_js.html
post_number: '1235'
reading_time_minutes: 5
series: general
slug: adva_demo_js
source_file: 1235_adva_demo_js.htm
tags:
- revit-api
- views
title: Functional Programming, View and Data API Demos
word_count: 917
---

### Functional Programming, View and Data API Demos

I spent Monday evening at the interesting meetup workshop on
[functional programming in JavaScript](http://www.meetup.com/basel-js/events/213653852) led
by basel.js meetup group regular Lukasz Gintowt.

All available seats were taken with 20 participants.

Follow the link above to get to all the interesting material we looked at.

It reminded me somewhat of the early days of AutoCAD programming, where the more elegant AutoLISP solutions tended towards functional programming...
Remember car, cdr, mapcar, apply and lambda?

Basically, that is part of what we talked about.

Look up [Haskell Brooks Curry](http://en.wikipedia.org/wiki/Haskell_Curry),
mathematician and logician, with three programming languages named after him,
[Haskell](http://en.wikipedia.org/wiki/Haskell_(programming_language)),
[Brooks](http://en.wikipedia.org/wiki/Brooks_(programming_language)) and
[Curry](http://en.wikipedia.org/wiki/Curry_(programming_language)),
as well as the concept of
[currying](http://en.wikipedia.org/wiki/Currying),
a technique used for transforming functions in mathematics and computer science.

We went through a number of cool hand-on JavaScript exercises listed in Lukasz'
[slides](https://codeeval-c9-syzer.c9.io/functional).

Here are some of the additional notes I took:

- A highly recommended JavaScript book: [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS).
- [Underscore](http://underscorejs.org) is an interesting and well-known JavaScript library providing functional programming helpers.
  [Lo-dash](https://lodash.com/) is similar and better in some respects.
- [Idempotency](http://en.wikipedia.org/wiki/Idempotence) is recommended for all REST call implementations, i.e., repeated calls to the same REST URL should return the same consistent result.
  For instance, if a first call to a URL ending in /delete/id returns the response 'deleted' or 'OK', i.e. 200, then all the further calls to the same URL should return the same value.
  Many implementations return something different, e.g. 'not found' on the second and all following calls.
  That is wrong; they should also return deleted or OK, or the idempotency will be violated.

#### Cool View and Data API Demos

[David In't Veld](http://www.davidveld.nl) posted a
[comment](http://thebuildingcoder.typepad.com/blog/2014/10/autodesk-view-and-data-api-notes-and-samples.html?cid=6a00e553e16897883301bb079df784970d#comment-6a00e553e16897883301bb079df784970d) on
the
[View and Data API notes and samples](http://thebuildingcoder.typepad.com/blog/2014/10/autodesk-view-and-data-api-notes-and-samples.html) asking for some simple demos:

**Question:** Is there any 'Demo' webpage to show-off what this View API can do?

It's a pity there are almost zero sample projects in an actually working Internet environment.
I thought by now this would have been set up.
Because clearly there has been a lot of work been put into this...
The website [developer.autodesk.com](https://developer.autodesk.com) simply points to GitHub but those samples require some study to make them work... even images on the Autodesk website are not working API but simple images.

Our company is investing in new ways of showing our Revit models, and I suggested the View API, however since I could not show any impressive working model.
I cannot convince anyone to invest in this...

Probably not the best place to mention this, but its frustrating every time I see View API in the title.
Would like to see more demo pages...

**Answer:** The best place to ask for more appropriate samples is in the dedicated Autodesk View and Data API discussion forum:
[Autodesk Community](http://forums.autodesk.com) –>
[Web Services API](http://forums.autodesk.com/t5/Web-Services-API/ct-p/94) –>
[View and Data API discussion forum](http://forums.autodesk.com/t5/View-and-Data-API/bd-p/95).

I would suggest submitting this comment there as well and see what the team has to say.

Here is a bunch of publicly accessible demo pages that will hopefully help fulfil your need, partially equipped with source code on GitHub to look at as well:

- [gallery.autodesk.io](http://gallery.autodesk.io)
- [mountcomp-stg.autodesk.io](http://mountcomp-stg.autodesk.io) – [github.com/Developer-Autodesk/client-mountsimulation-view.and.data.api](https://github.com/Developer-Autodesk/client-mountsimulation-view.and.data.api)
- [timeliner.autodesk.io](http://timeliner.autodesk.io) – [github.com/Developer-Autodesk/client-timeliner-view.and.data.api](https://github.com/Developer-Autodesk/client-timeliner-view.and.data.api)
- [octoviewer.autodesk.io](http://octoviewer.autodesk.io)
- [checkoutmymodel.autodesk.io](http://checkoutmymodel.autodesk.io) – [github.com/Developer-Autodesk/workflow-aspnet-webform-view.and.data.api](https://github.com/Developer-Autodesk/workflow-aspnet-webform-view.and.data.api)
- [sapdemo.autodesk.io](http://sapdemo.autodesk.io) – [github.com/Developer-Autodesk/integration-sap-view.and.data.api](https://github.com/Developer-Autodesk/integration-sap-view.and.data.api)
- [viewer.autodesk.io/bingmapintegration/](http://viewer.autodesk.io/bingmapintegration/)
- Steampunk – [autode.sk/m3w](http://autode.sk/m3w)
- Tractor – [s3.amazonaws.com/FastViewer/index.html?file=frontloader/0.svf](https://s3.amazonaws.com/FastViewer/index.html?file=frontloader/0.svf)
- Office – [s3.amazonaws.com/FastViewer/index.html?file=Waltham/0.svf](https://s3.amazonaws.com/FastViewer/index.html?file=Waltham/0.svf)
- Blog – [adndevblog.typepad.com/cloud\_and\_mobile/stephens-test-page.html](http://adndevblog.typepad.com/cloud_and_mobile/stephens-test-page.html)

By the way, I looked at David's [web site](http://www.davidveld.nl) and really like the photos.

Another example of combining photography and Revit API programming, just like my colleague
[Jaime](http://adndevblog.typepad.com/aec/jaime-rosales.html).