---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: news
optimization_date: '2025-12-11T11:44:14.068296'
original_url: https://thebuildingcoder.typepad.com/blog/0505_sheet_set_manager.html
post_number: '0505'
reading_time_minutes: 3
series: views
slug: sheet_set_manager
source_file: 0505_sheet_set_manager.htm
tags:
- parameters
- revit-api
- schedules
- sheets
- views
title: Sheet Manager and Copy Parameters
word_count: 554
---

### Sheet Manager and Copy Parameters

I have started settling back into normal life after the extensive conference tour and a whole month of non-stop travelling.
I still have not been able to start preparing Christmas, though, except for buying lots and lots of food to carry me and my grown-up kids and their partners through the coming days.
I need to start thinking about presents now as well, I guess.

I just had a meeting with my colleagues and heard of Adam's traumatic return journey from Milano to the UK this weekend.
He finally made it, but it really was a bad journey.
The DevDay conference in the UK with Gary and Adam taking over all the presentations went very well, though.
Congratulations and thanks to Gary and Adam on managing that so competently!

Meanwhile, on the Revit API front, I recently mentioned the
[Russian add-in utilities](http://thebuildingcoder.typepad.com/blog/2010/12/%D0%B4%D0%BE%D0%BF%D0%BE%D0%BB%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F-%D0%B4%D0%BB%D1%8F-%D1%80%D0%B5%D0%B2%D0%B8%D1%82-add-ins-for-revit.html)
provided by
[Артур Кураков alias Arthur Kurakov](http://kartautodeskuser.blogspot.com).
Fedor added a
[comment](http://thebuildingcoder.typepad.com/blog/2010/12/%D0%B4%D0%BE%D0%BF%D0%BE%D0%BB%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F-%D0%B4%D0%BB%D1%8F-%D1%80%D0%B5%D0%B2%D0%B8%D1%82-add-ins-for-revit.html#comment-6a00e553e1689788330148c687124b970c) to point out that we missed the most exciting add-in, the sheet manager, so here is a little follow-up from Arthur to rectify that:

#### Sheet Manager – Менеджер листов

This add-in organises sheets to albums.
It controls its own numbering in each album.
It uses a text shared parameter created by the user for numbering.
In addition, the user can add specific parameters, which must be of the same value in each album.
Sheet Manager monitors changes to these parameters and, if this parameter was specified to album, changes it to specified value.
If it was parameter of album sheets numbering, the Sheet Manager controls renumbering of the other one.
It allows user to move sheets up and down in albums.
You can drop sheets from album to album by changing the appropriate parameter.
Sheet Manager manipulates standard Revit parameters, so it easy to create standard Revit schedules and use other parameter based functions.

Actually, there is even more:

#### Copy Parameters – Копирование параметров

Arthur created a new add-in named
["Копирование параметров" (copy parameters)](http://kartautodeskuser.blogspot.com/2010/12/blog-post_20.html).
It copies instance parameters from one instance to others.
It allows you to select which instance parameters you want to copy.

The user interface is minimal and intuitive.
There are only three buttons with the following Russian labels:

- "Выбрать все" - select all- "Отменить выбор" - select none.- "Копировать параметры" - copy parameters.

Finally, here is a link to Arthur's Russian post about the
[DevTV Visual Studio template files to create a new Revit add-in project](http://kartautodeskuser.blogspot.com/2010/12/blog-post.html).

Please note that I also posted a
[DevTV template update](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html).