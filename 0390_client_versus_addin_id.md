---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.1
content_type: qa
optimization_date: '2025-12-11T11:44:13.866920'
original_url: https://thebuildingcoder.typepad.com/blog/0390_client_versus_addin_id.html
post_number: 0390
reading_time_minutes: 1
series: general
slug: client_versus_addin_id
source_file: 0390_client_versus_addin_id.htm
tags:
- revit-api
title: ClientId Versus AddInId
word_count: 151
---

### ClientId Versus AddInId

Looking at many other developers' add-in manifest files here at the AEC DevLab in Waltham, I just noticed that the
[ClientId tag](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html#4)
that I have been using in my manifest files has actually been renamed to AddInId to be more consistent.
Most other developers are using the AddInId tag instead of the ClientId one.
The ClientId tag still works, but maybe I should start using AddInId myself as well in future.

I updated my
[guidize utility](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html#5) to
populate and update the contents of both the AddInId and ClientId tags.
Here is version 2 which does that, both the
[complete source code and Visual Studio solution file](zip/guidize_src_2.zip) and also the
[ready-built executable](zip/guidize_exe_2.zip) for non-programmers.