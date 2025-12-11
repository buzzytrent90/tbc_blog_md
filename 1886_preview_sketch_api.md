---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: tutorial
optimization_date: '2025-12-11T11:44:16.965247'
original_url: https://thebuildingcoder.typepad.com/blog/1886_preview_sketch_api.html
post_number: '1886'
reading_time_minutes: 3
series: views
slug: preview_sketch_api
source_file: 1886_preview_sketch_api.md
tags:
- elements
- geometry
- revit-api
- sheets
- views
- walls
title: Preview Sketch Api
word_count: 539
---

### Sketch-Based Creation and Editing API Preview
The wish to be able to programmatically access and modify sketch-based elements is long-standing and was voiced back in 2013 in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [editing an existing slab boundary](https://forums.autodesk.com/t5/revit-api-forum/edit-existing-slab-boundary/m-p/10014819).
Oleg Sheydvasser, Software Architect in the Revit development team, just joined that conversation to announce an invitation to take a look at the Revit Preview (aka beta), offering a chance to provide feedback on several new sketch-based element creation and editing APIs:
- [Ceiling creation API](https://feedback.autodesk.com/project/forum/thread.html?cap=cb0fd5af18bb49b791dfa3f5efc47a72&forid=%7b057e532f-e478-43d9-affc-01b3deb82a76%7d&topid=%7b41E11E18-938F-4260-8190-3C3227B9C5FA%7d)
- [Sloped Ceiling creation API](https://feedback.autodesk.com/project/forum/thread.html?cap=cb0fd5af18bb49b791dfa3f5efc47a72&forid=%7b057e532f-e478-43d9-affc-01b3deb82a76%7d&topid=%7b2B45F2E2-F58F-4FC3-9518-50D8E34C4394%7d)
- [Floor creation API](https://feedback.autodesk.com/project/forum/thread.html?cap=cb0fd5af18bb49b791dfa3f5efc47a72&forid=%7b057e532f-e478-43d9-affc-01b3deb82a76%7d&topid=%7b1A358CD2-1C43-457E-A680-9F2DD81E3555%7d)
- [Get Sketch elements API](https://feedback.autodesk.com/project/forum/thread.html?cap=cb0fd5af18bb49b791dfa3f5efc47a72&forid=%7b057e532f-e478-43d9-affc-01b3deb82a76%7d&topid=%7bAE88ED3F-7EF9-4D0C-B599-E45BD5754DCA%7d)
- [Sketch Edit Mode API](https://feedback.autodesk.com/project/forum/thread.html?cap=cb0fd5af18bb49b791dfa3f5efc47a72&forid=%7b057e532f-e478-43d9-affc-01b3deb82a76%7d&topid=%7b9C3D609F-0A86-4766-BB12-6315F49BCF03%7d)
- [Floor Sketch editing API](https://feedback.autodesk.com/project/forum/thread.html?cap=cb0fd5af18bb49b791dfa3f5efc47a72&forid=%7b057e532f-e478-43d9-affc-01b3deb82a76%7d&topid=%7bBD054C86-FFAA-442F-88D0-CAEFA1D89221%7d)
- [Wall and Shaft Opening Sketch editing API](https://feedback.autodesk.com/project/forum/thread.html?cap=cb0fd5af18bb49b791dfa3f5efc47a72&forid=%7b057e532f-e478-43d9-affc-01b3deb82a76%7d&topid=%7bF15302E9-CC66-4BB3-8168-23D615E94558%7d)
The Revit Preview is part of the Autodesk Feedback Community, offering opportunities to provide feedback on future development features.
To join the Revit Preview program, please

[email revit.preview.access@autodesk.com](mailto:revit.preview.access@autodesk.com).

If you are an ADN member, you can also access it through the ADN member portal via the link Software → Beta Portal.
I am looking forward to seeing you there, and the Revit development team is looking forward to your feedback on these new powerful Revit programming possibilities.
Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas already shared a first reaction:
> Those titles look like exciting additions to the API.
If they provide stable functionality of the form noted in the titles then I would consider them to be a major step forward in API functionality (regardless of what lengths I need to go to in using them).
> Currently there are things we cannot do with the API.
When we have a situation where everything geometry wise done in the UI can be done in the API then that is the starting point for me, especially considering increasing Forge use etc.
Those items seem like a great step forward towards that aim.
> Now we just need to find out how to create and edit a scope box...
![Feedback Community Preview Release](img/feedback_community_preview_release.png "Feedback Community Preview Release")