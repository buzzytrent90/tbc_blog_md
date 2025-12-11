---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: news
optimization_date: '2025-12-11T11:44:17.076783'
original_url: https://thebuildingcoder.typepad.com/blog/1934_revitextensions.html
post_number: '1934'
reading_time_minutes: 3
series: general
slug: revitextensions
source_file: 1934_revitextensions.md
tags:
- doors
- elements
- parameters
- revit-api
- sheets
title: Revitextensions
word_count: 520
---

### Happy New Year with RevitExtensions
Off we go into a new adventurous year of BIM programming:
- [Happy New Year](#2)
- [RevitExtensions](#3)
#### Happy New Year
Happy New Year and welcome to 2022!
I spent a pleasant New Year's Day climbing the Wildhauser Schafberg in warm and dry weather.
![Jeremy figurehead on Wildhauser Schafberg](img/143943_jeremygipfelfahnengallionsfigur.jpg "Jeremy figurehead on Wildhauser Schafberg")
Afterwards, a friend pointed out this rather humorous New Year's greeting from 1883.
It renders better in its original German version than in English:

Neujahrsgebet

Herr, setze dem Überfluss Grenzen

und lasse die Grenzen überflüssig werden.

Lasse die Leute kein falsches Geld machen

und auch Geld keine falschen Leute.

Nimm den Ehefrauen das letzte Wort

Und erinnere die Ehemänner an ihr erstes.

Schenke unseren Freunden mehr Wahrheit

und der Wahrheit mehr Freunde.

Gib den Regierenden ein besseres Deutsch

Und den Deutschen eine bessere Regierung.

Herr, sorge dafür, dass wir alle in den Himmel kommen

Aber nicht sofort!

Lord, please border abundance

and make borders superfluous.

Don't let people make bad money

and don't let money make bad people.

Take the last word away from the wives

And remind the husbands of their first.

Give more truth to our friends

and more friends to the truth.

Give the rulers a better German

And better rulers to the Germans.

Lord, please may we all go to heaven

but not right away!

Parish priest Hermann Josef Kappen of Lamberti church in Münster, 1883

![Neujahrsgruss 1883](img/neujahrsgruss_1883.jpg "Neujahrsgruss 1883")
#### RevitExtensions
In the last post of the previous year, I mentioned
Roman [Nice3point](https://github.com/Nice3point), his huge contributions
to [RevitLookup](https://github.com/jeremytammik/RevitLookup) in the past few months,
his [RevitTemplates update 1.7.0](https://thebuildingcoder.typepad.com/blog/2021/12/revittemplates-update-170.html)
and the invitation to provide feedback on them.
Let's move into this new year with yet another contribution and invitation from Roman:
Hi guys, it's time to pump the Revit API.
I started developing a library that will make it easier to write code using extensions.
In general, instead of `Method(Method(Method(Method(Method()))))`, you can write `Method.Method.Method.Method.Method`.
And, of course, I added a couple of new methods and overloads that are not in the API.
Working with the Ribbon and \*Utils classes has been greatly simplified.
If you have any suggestions for improving the API, write to me about it in
the

[RevitExtensions GitHub repository](https://github.com/Nice3point/RevitExtensions).

> Improve your experience with Revit API now
> Extensions minimize the writing of repetitive code, add new methods not included in the native Revit API, and also allow you to write chained methods without worrying about API versioning:
>

```
  new ElementId(123469)
    .ToElement(document)
    .GetParameter(BuiltInParameter.DOOR_HEIGHT)
    .AsDouble()
    .ToMillimeters()
    .Round()
```

> Extensions include annotations to help ReShaper parse your code and signal when a method may return null, or the value returned by the method is not used in your code.
Many thanks again to Roman for all his tremendous work supporting and enhancing Revit API development!
And, again, Happy New Year to all!