---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: news
optimization_date: '2025-12-11T11:44:17.096285'
original_url: https://thebuildingcoder.typepad.com/blog/1943_purge_etransmit.html
post_number: '1943'
reading_time_minutes: 6
series: general
slug: purge_etransmit
source_file: 1943_purge_etransmit.md
tags:
- family
- references
- revit-api
- sheets
- views
title: Purge Etransmit
word_count: 1191
---

### Purge Unused and the Autodesk Camel
We return once again to the topic of programmatic purging and learn the history and latest news about the venerable Autodesk Camel:
- [Purge unused using eTransmitForRevitDB.dll](#2)
- [The Purge via API method](#2.1)
- [The Autodesk office Camel](#3)
- [Node.js reference architecture](#4)
#### Purge Unused using eTransmitForRevitDB.dll
Last month, we
discussed [several approaches to automating the purge command](https://thebuildingcoder.typepad.com/blog/2022/02/purge-unused-and-etransmit-for-da4r.html).
Now Emiliano Capasso,
Head of BIM at [Antonio Citterio Patricia Viel](https://www.citterio-viel.com),
shared an even better one,
explaining [why the Revit Purge​ command is an act of love](https://www.linkedin.com/pulse/why-revit-purge-command-act-love-via-api-emiliano-capasso),
or *Perché il comando "Purge" di Revit è un atto d'amore*:
I'm writing this hoping it will be useful for all BIM Managers, Head of BIMs, BIM Directors, etc. around the world.
The Life-Changing Magic of Tidying is the mantra of Marie Kondo, and the Japanese chain MUJI even published a book named “CLEANING” with hundreds of photos of cleaning activities all over the world.
An organised life is key and so why our models should be a mess?
Purging, cleaning the models is the key of maintaining a high-quality standard, and for us delivering state of the art BIM Models is the priority.
Messy models are problematic, all of our systems work around the clock with automations and data extractions to our BI dashboards to monitor KPI.
But as most of our architects are deeply caught in the design process and they forget to keep a tidy model, but our BIM department is here to help.
But with more than 40 active projects, how could we purge (3 times) manually every one of the 500ish models that we have?
#### The Purge via API Method
Tons of words have been spent about Purging in Revit via API, which is apparently impossible, cf.,
the [latest post on the topic](https://thebuildingcoder.typepad.com/blog/2022/02/purge-unused-and-etransmit-for-da4r.html)).
There are lots of workarounds, such as **PerformanceAdvisor** and **PostableCommand**, but none of which was satisfying me.
So I was wondering, how the marvellous **eTransmit** addin made by Autodesk is actually Purging the models whilst transmitting?
So I went looking into the folder, found the .dll and in our addin referenced the **eTransmitForRevitDB.dll** and looked into its **public** methods:
![eTransmitForRevitDB.dll public methods](img/purge_unused_etransmit_1.jpeg "eTransmitForRevitDB.dll public methods")
Wow.
Could that be so easy?
![eTransmitForRevitDB.dll public methods](img/purge_unused_etransmit_2.jpeg "eTransmitForRevitDB.dll public methods")
Yes.
Below the snippet:

```
  public bool Purge(Application app, Document doc)
  {
    eTransmitUpgradeOMatic eTransmitUpgradeOMatic
      = new eTransmitUpgradeOMatic(app);

    UpgradeFailureType result
      = eTransmitUpgradeOMatic.purgeUnused(doc);

    return (result == UpgradeFailureType.UpgradeSucceeded);
  }
```

Just create an instance of `eTrasmitUpgradeOMatic` passing the `Application` and call its method `purgeUnused` passing the `Document`;
that will return an `UpgradeFailureType`.
Now you have your model purged (3 times also).
So satisfying.
Many thanks to Emiliano or his research and nice explanation!
#### The Autodesk Office Camel
Nicolas Menu and other colleagues from Neuchatel shared the story of the Autodesk office Camel on LinkedIn.
This is an important story, so I take the opportunity to preserve it here as well:
After many years at Autodesk European headquarter office in Neuchatel, Switzerland, the Camel (our office's mascot) moved tonight to the Swiss Siberia.
![Autodesk Camel](img/autodesk_camel.jpeg "Autodesk Camel")
History...
- 1982 Autodesk, Mill Valley, Marin County, CA, USA
- 1991 Autodesk European Headquarters, Marin, Switzerland
From Marin County, to... Marin, Switzerland!
The office later moved a few km away to Neuchatel, and the camel remained in the new office as a mascott, a landmark.
- 2017... Autodesk European headquaters close and move to Dublin.
But The Camel didn't like it and decided to remain fidel and stay right here...
For about 5 years he stays with Coral and Francesco, and tonight it has find a new home... guess where :)
The Camel is now with our family in the coldest village of Switzerland... Long Life to the Camel!
Claudio Ombrella adds: Nice. Let me take the opportunity to share the story of the camel.
It was January 1993 when I joined the Marin, Switzerland office from Autodesk in Milano.
The Autodesk office was located at the first floor of the number 14b of the Av. Champs Montant.
All people driving to find the office did it from the West side: at the ground floor there a Persian carpet shop named *Tapis d'Orient*.
They had put a camel, better say *the camel* outside the office.
It quickly became the 'signpost' to arrive to the office: 'our entrance is by camel' used to say our colleague Sheila Ahles to any visitor needing instructions.
One day the shop closed and the camel became orphan.
But not for so long, because Autodesk employees pulled the camel into the office.
And it became the mascot with its own badge that was regularly reprinted at every company logo change!
In 1994 we moved to the Puits Godet office and the camel moved too.
With the years it deteriorated a bit and then Bodo Vahldieck loaded on his VW Multivan and took it home for a complete restore.
Then it came back to the office in perfect shape.
In 2015 my team opened a job and a young person applied for it, so we invited for an on-site interview.
When the person saw the camel he said to me: “and you have the camel? I am the son of the shop Tapis d’Orient, I am so surprised to see it again”.
Thank you, Nicolas, for giving a new life to our old friend, the camel.
John C.: The camel was first on the list of assets to be moved from Marin to Puits-Godet!
Thanks for bringing such good memories!
Lisa Senauke: James Carrington and I arrived when the Marin office had just opened and we were among the first 5 or 6 to arrive
– Kern (and John Walker) Hans, Sheila, James, and I – who else? Creighton?
When we were campaigning to be chosen to work in Switzerland, one of my proudest moments was when Hansi said to me, while I was pondering if we would get to go, and if there would be a job for me as well.
Hans looked at me and said, "Lisa, where I go, YOU go!" 🥰 And I did!
The Camel was standing proud and tall by the entrance of the building, welcoming us to our new home!
For me, it was a Jungian peak experience!
Raquel Aragonés: I had completely forgotten about the camel.
I was there from 1992-3 to 1997!
Long live the camel!
#### Node.js Reference Architecture
Moving from the desktop to the cloud, could you use some advice on which components to employ for your Forge app?
Or are you working with other cloud applications and confounded by the plethora of available libraries?
Maybe the [Node.js Reference Architecture](https://github.com/nodeshift/nodejs-reference-architecture) by
IBM and Red Hat will help make a well-founded decision.