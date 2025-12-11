---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:16.360578'
original_url: https://thebuildingcoder.typepad.com/blog/1594_record_book_edu_unit.html
post_number: '1594'
reading_time_minutes: 6
series: general
slug: record_book_edu_unit
source_file: 1594_record_book_edu_unit.md
tags:
- elements
- family
- levels
- parameters
- revit-api
- sheets
- transactions
title: Record Book Edu Unit
word_count: 1178
---

### AU Recording, Books, Education and Units
I completed the presentation and recording of my presentation yesterday on *Rational BIM programming using Revit and Forge* at Autodesk University in Darmstadt, Germany, and returned back home again.
Here are a few remarks along the way:
- [\*Rational BIM Programming\* recording](#2)
- [Pleasant walks in the Odenwald](#3)
- [Some of my favourite recent books](#4)
- [Where to continue after finishing school](#5)
- [New AlphaGo Zero is completely self-taught](#6)
- [TotalPressureLoss units](#7)
#### Rational BIM Programming Recording
I recorded my one-hour presentation in German language
on [Rational BIM Programming using Forge and Revit API](https://youtu.be/vbiCj0S4mv8) at
Autodesk University in Darmstadt, Germany:

It expands on previous presentations on \*Connecting BIM on Desktop and Cloud\* and \*Freeing your BIM Data\*.
The full documentation on this class including the English language handout document and slide deck is available from The Building Coder post on [Rational BIM Programming at AU Darmstadt](http://thebuildingcoder.typepad.com/blog/2017/10/rational-bim-programming-at-au-darmstadt.html).
#### Pleasant Walks in the Odenwald
In Darmstadt, I was also able to pay a visit my good old friend George and talk of many things... "of shoes--and ships--and sealing-wax--of cabbages--and kings" [cf. [The Walrus and The Carpenter](http://www.jabberwocky.com/carroll/walrus.html) by Lewis Carroll].
We also took several walks in the [Odenwald](https://en.wikipedia.org/wiki/Odenwald) with his dog Leo:
![Walk in the woods](/p/2017/2017-10-17_au_darmstadt/odenwald_img_4340.jpg)
![Walk in the woods](/p/2017/2017-10-17_au_darmstadt/odenwald_img_4344.jpg)
![Walk in the woods](/p/2017/2017-10-17_au_darmstadt/odenwald_img_4347.jpg)
#### Some of My Favourite Recent Books
Here are some of the books I recently read and enjoyed more than others:
- Noam Shpancer: \*The Good Psychologist\* [en] – a loving, intelligent, illuminating story of a psychologist in practice and teaching, on practical therapy and theory, interwoven with his own personal love story, loneliness and yearning.
- Juli Zeh: \*Unter Leuten\* [de] – utterly brilliant, loving, deeply psychological and fascinating story about different realities and conflict that the people of a small village create by misunderstanding and for mutual manipulation.
- Bernhard Jaumann: \*Die Vipern von Montesecco\* [de] – a brilliant whodunnit in a small Italian village with everybody under suspicion and the village wholeheartedly determined to solve this to preserve its sense of unity, and solve it with no outside interference.
- Antonio Manzini: \*Non e stagione\* [it] – un giallo, exciting detective story featuring Rocco Schiavione, a totally hip super macho cop from Rome transferred to Aosta and unhappy with snow in May, trying to save an abandoned kidnapped girl from dying.
- Antonio Manzini: \*Pista nera\* [it] – I love Rocco Schiavione and Antonio Manzini; radical militant violent personal integrity, regardless of law and order, serving a higher and personal truth.
- Antonio Manzini: \*La costola di Adamo\* [it] – more radical personal integrity; the topic is political and sociological, femmicide; read the last sentence of the acknowledgements before starting the book; cf. this [quote and my translation](http://thebuildingcoder.typepad.com/blog/2015/09/change-type-iterate-elements-create-family.html#5).
- Antonio Manzini: \*Era di maggio\* [it] – number four in this series.
#### Where to Continue After Finishing School
Another topic that came up with several friends and their kids is what to do after finishing school.
Here are my three current favourite education options:
- For people who are unsure what they want: [Knowmads Creative International Business School](http://www.knowmads.nl)
- For young people who know that they want to learn programming in depth: [42](https://en.wikipedia.org/wiki/42_(school)), a private, non-profit and tuition-free computer programming school with campuses in Paris and [Silicon Valley](https://www.42.us.org)
- For anyone wishing to quickly and efficiently launch a web programming oriented career from scratch: [freecodecamp.org](https://www.freecodecamp.org)
My son attended Knowmads a couple of years ago, and now another friend's daughter is there as well, and extremely enthusiastic about it.
#### New AlphaGo Zero is Completely Self-Taught
The AI research community has been very excited indeed lately by the super-human level of performance achieved by [AlphaGo](https://en.wikipedia.org/wiki/AlphaGo), by learning from a combination of human knowledge and self-generated experience:
- [AlphaGo beats grand master](http://thebuildingcoder.typepad.com/blog/2016/01/bim-programming-madrid-and-spanish-connectivity.html#7)
- [AlphaGo beats world champion](http://thebuildingcoder.typepad.com/blog/2017/06/ai-news-and-sub-transaction-regen.html#2)
New research has taken it yet another step further, and
now [DeepMind's Go-playing AI doesn't need human help to beat us anymore](https://www.theverge.com/2017/10/18/16495548/deepmind-ai-go-alphago-zero-self-taught)
– the latest AlphaGo AI learned superhuman skills by playing itself over and over.
Here is a two-and-a-half-minute video about AlphaGo [discovering new knowledge](https://youtu.be/mJ4tEDMksWA):

#### TotalPressureLoss Units
Back to the Revit API, and a question in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160)
on [`TotalPressureLoss` units](https://forums.autodesk.com/t5/revit-api-forum/totalpressureloss-units/m-p/7452507):
I'm using `MEPSection.TotalPressureLoss` to get the pressure drop of the section.
The Revit UI tells me, in this example, the pressure drop is 5853Pa.
When I get the value using the API, or Revit Lookup, I get the number 1784.0906645432:
![TotalPressureLoss units](img/TotalPressureLoss_units.jpg)
What units is this value in?
The API documentation on
the [TotalPressureLoss property](http://www.revitapidocs.com/2018.1/f75e82be-d681-544c-641f-c943765ef2be.htm) says,
"default unit is Kgf per square feet", which has got to be the weirdest unit of pressure ever, but whatever.
When I convert 5853Pa to Kgf/Ft^2 I get 55.45, so that can't be the correct unit:
- [WolfranAlpha convert 5853 pascals to kgf/ft^2](http://www.wolframalpha.com/input/?i=convert+5853+pascals+to+kgf%2Fft%5E2)
Am I missing something?
I know it's standard for Revit to convert to some pretty weird units, that's why I'm trying to figure out what this one is.

`5853Pa`

It's not:
- feet of head (1.958)
- or inwg (23.497)
- or psi (.8498)
- or in Hg (1.728)
- or cmHg (4.39)
- or atm (.05766)
- or bars (58.53)
Which is all the units that Revit supports as far as I can tell.
It's also not:
- lbf/yard^2 (1100)
- or N/yard^2 (4894)
- or N/foot^2 (543.8)
- or N/barn (5.853 x 10^-25)
- or ozf/in^2 (13.58)
I guess it's RPU or Revit Pressure Units or a bug in the code?
\*\*Answer:\*\* The development team responds:
Sorry for the hassle.
It should be:
(kg ft / s^2 ) / ft ^2 or kg / (ft s^2)
The best bet for anyone getting values from Revit parameters which are in a more complicated unit than length is to use the `UnitUtils` method `ConvertFromInternalUnits` to the display unit type you want.