---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.937634'
original_url: https://thebuildingcoder.typepad.com/blog/1398_bim_programming.html
post_number: '1398'
reading_time_minutes: 5
series: general
slug: bim_programming
source_file: 1398_bim_programming.md
tags:
- csharp
- parameters
- python
- revit-api
- rooms
- schedules
- sheets
- views
- windows
title: Bim Programming
word_count: 1088
---

### BIM Programming Madrid and Spanish Connectivity
I spent this week in Madrid, presenting at
the [BIM Programming](http://www.bimprogramming.com) conference and teaching the subsequent two-day workshop on the Revit API and \*Connecting the desktop and the cloud\*.
- [BIM Programming mainstage presentation](#2)
- [The Spanish nature of connectivity](#3)
- [Castafiore](#4)
- [Zazen](#5)
- [Matins](#6)
- [AlphaGo, machine learning and intuition](#7)
[![BIM Programming Madrid](https://farm2.staticflickr.com/1566/24591730181_6902d72677_n.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157663375201559 "BIM Programming Madrid")
#### BIM Programming Mainstage Presentation
My mainstage presentation addressed the following topics:
- The future of making things, [IoT](https://en.wikipedia.org/wiki/Internet_of_Things), [Forge](http://forge.autodesk.com)
- WebGL and the [View and Data API](https://developer.autodesk.com/api/view-and-data-api/)
- BIM Programming
- Connecting the desktop and the cloud
- [2D cloud-based Revit room editor](https://github.com/jeremytammik/RoomEditorApp)
- [FireRating in the cloud](https://github.com/jeremytammik/FireRatingCloud)
Everything went very well indeed, with a rather Spanish schedule meaning late hours, so I ended up eating dinner between ten o'clock in the evening and midnight, and falling into bed between one and two in the morning every night.
I am exhausted!
#### The Spanish Nature of Connectivity
So it turned out to be a pretty crazy week with very late hours compared to my usual habits, little sleep, and many exciting technical discussions.
I probably talked more here in the last few days than I have in the entire last few months back in Switzerland, at least as far as programming is concerned.
Connecting the desktop and the cloud is so utterly easy!
The BIM and developer community here is Spain is incredibly enthusiastic about the possibilities this offers.
It has been a great pleasure and honour to work together so closely and intensively with Alberto Arteaga Garcia and above all Jose Ignacio Montes of [Avatar BIM](http://avatarbim.com).
In the past days, Jose and I implemented [FireRatingClient](https://github.com/jeremytammik/FireRatingCloud/tree/master/FireRatingClient),
a new stand-alone [fireratingdb](https://github.com/jeremytammik/firerating) client,
a Revit-independent Windows forms-based sibling of
the [FireRatingCloud Revit add-in](https://github.com/jeremytammik/FireRatingCloud).
You can check it out right away.
The GitHub readme tells you all you need to know to understand it.
You can also check out the [to-do list](https://github.com/jeremytammik/FireRatingCloud#todo) to get an idea of the direction we are headed.
Furthermore, we are working on improvements to the [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp),
which we intend to migrate from CouchDB to node.js and mongodb.
#### Castafiore
On my last night here, Alberto took us
to [La Castafiore](http://www.lacastafiore.net) ([twitter](https://twitter.com/LaCastafioreav)).
A nice dinner accompanied by opera arias and ending in a rather unusual yet also very Spanish manner:
![La Castafiore](/p/2016/2016-01-27_madrid/888_castafiore_600x360.jpg)
#### Zazen
This coming Sunday morning, I will be leading a short [Zazen](https://en.wikipedia.org/wiki/Zazen) meditation session, so here are some notes by [Thích Nhất Hạnh](https://en.wikipedia.org/wiki/Th%C3%ADch_Nh%E1%BA%A5t_H%E1%BA%A1nh) on [how to sit](http://www.lionsroar.com/thich-nhat-hanh-sit):
- Set aside a room or corner or a cushion that you use just for sitting.
- The sound of a bell is a wonderful way to begin sitting meditation. If you don’t have a bell you can download a recording of the sound of a bell onto your phone or computer.
- When you sit, keep your spinal column quite straight, while allowing your body to be relaxed. Relax every muscle in your body, including the muscles in your face. Consider smiling slightly, a natural smile. Your smile relaxes all your facial muscles.
- Notice your breathing. As you breathe in, be aware that you are breathing in. As you breathe out, notice that you are breathing out. As soon as we pay attention to our breath, body, breath and mind come together. Every in-breath can bring joy; every out-breath can bring calm and relaxation. This is a good enough reason to sit.
- When you breathe in mindfully and joyfully, don’t worry about what your sitting looks like from the outside. Sit in such a way that you feel you have already arrived.
- It’s wonderful to have a quiet place to sit in your home or workplace. If you are able to find a cushion that fits your body well, you can sit for a long time without feeling tired. But you can practice mindful sitting wherever you are. If you ride the bus or the train to work, use your time to nourish and heal yourself.
- If you sit regularly, it will become a habit. Even the Buddha still practiced sitting every day after his enlightenment. Consider daily sitting practice to be a kind of spiritual food. Don’t deprive yourself and the world of it.
#### Matins
To end the sitting, I plan to read this nice morning poem by the late [John O'Donohue](http://www.johnodonohue.com) together:
\*\*Matins – Eternal Echoes\*\*
II.
I arise today
In the name of Silence

Womb of the Word,

In the name of Stillness

Home of Belonging,

In the name of the Solitude

Of the Soul and the Earth.
સ
I arise today
Blessed by all things,

Wings of breath,

Delight of eyes,

Wonder of whisper,

Intimacy of touch,

Eternity of soul,

Urgency of thought,

Miracle of health,

Embrace of God.
સ
May I live this day
Compassionate of heart,

Gentle in word,

Gracious in awareness,

Courageous in thought,

Generous in love.
સ
It is especially nice in German, or maybe I am just more used to that version nowadays:
\*\*Morgengedanken\*\*
ich erhebe mich heute

im namen des schweigens – schoss des wortes

im namen der stille – heim des zugehörens

im namen der einsamkeit – der seele und der erde
સ
ich erhebe mich heute

gesegnet von jeglichem ding

schwingen des atems

wonne der augen

staunen des flüsterns

nähe der berührung

dringlichkeit des gedankens

wunder der gesundheit

gottes umarmung
સ
möge ich verleben diesen tag als mensch

mitfühlenden herzens

gütigen wortes

freundlichen achtens

mutigen sinns

freigebiger liebe
સ
#### AlphaGo, Machine Learning, Machine Intuition?
Talking about Japanese culture, the [AlphaGo](https://en.wikipedia.org/wiki/AlphaGo) Go computer program has now beaten a grand master of Go.
Machine learning is gradually conquering areas that cannot be cracked by pure combinatorial analysis, like chess, but require learning and something akin to intuition to solve.