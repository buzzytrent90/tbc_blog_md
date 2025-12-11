---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: news
optimization_date: '2025-12-11T11:44:13.749791'
original_url: https://thebuildingcoder.typepad.com/blog/0327_api_wishlist_survey.html
post_number: '0327'
reading_time_minutes: 2
series: general
slug: api_wishlist_survey
source_file: 0327_api_wishlist_survey.htm
tags:
- elements
- family
- levels
- parameters
- revit-api
- selection
- views
- walls
- windows
title: API Wish List Survey Results
word_count: 337
---

### API Wish List Survey Results

Picking up on Kean's announcement of the
[availability of the 2011 family of Autodesk products](http://through-the-interface.typepad.com/through_the_interface/2010/03/autocad-2011-is-now-shipping.html) I thought I might mention how the list of
[Revit 2011 API features](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-is-coming.html) that I published yesterday ties in with the
[Revit API wish list survey](http://thebuildingcoder.typepad.com/blog/2009/07/api-wish-list-survey-reminder.html)
that we held last summer.

Here are the results of the survey, with the areas that are addressed by the Revit 2011 API highlighted in bold:

1. **Better integration into Revit user interface**.
2. **Interactive user selection of element ends, intersections and faces**.
3. **Ability to post and handle custom errors**.
4. Access to the Revit project browser.- Creation of docked windows in the Revit user interface.- Open a Revit document in the user interface.- Access to views.- Encode more complex formulas into family parameters.- Change existing element behaviour, such as the join behaviour of a wall.- **Event handlers for elements when added, deleted, changed, moved or selected**.

Add-ins do not yet have full access to all aspects of the Revit user interface, but it been substantially improved by many of the new features, e.g. task dialogues, add-in manifest file functionality, the new element selection and point picking, the failure API, and many more.

I have always been surprised that the wish for element level event handlers ended up last on this list, since I personally would have placed it much higher, and also saw a huge number of requests for it from the developer community.
Therefore it pleases me all the more that it ended up being addressed, and not just by one new feature, but by two, the dynamic model update and elements changed event.
It will be very exciting to see what use can be made of these and all the other new possibilities!