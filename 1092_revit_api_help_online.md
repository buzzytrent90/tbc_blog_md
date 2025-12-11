---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:15.288704'
original_url: https://thebuildingcoder.typepad.com/blog/1092_revit_api_help_online.html
post_number: '1092'
reading_time_minutes: 2
series: general
slug: revit_api_help_online
source_file: 1092_revit_api_help_online.htm
tags:
- revit-api
- windows
title: Revit API Help Online and Hiking on La Palma
word_count: 492
---

### Revit API Help Online and Hiking on La Palma

Good news for all Revit add-in developers from Peter Boyer of the Dynamo team, working on visual programming for Revit, who brought you the
[Dynamo Revit Unit Test Framework](http://thebuildingcoder.typepad.com/blog/2013/10/the-dynamo-revit-unit-test-framework.html).

He says:

We use the Revit API docs a lot, so I decided to build a website that basically just makes the Revit API help file RevitAPI.chm file provided with the Revit SDK more visible on the web:

[revitapisearch.com](http://www.revitapisearch.com)

I found it makes it easier to point other developers to the documentation and greatly improves search speed and quality of results in comparison to the CHM file.

I hope other Revit add-in developers find this useful as well.

Please post a comment if you run into any issues, or have any questions.

#### Disassembling a CHM and Creating a Searchable Amazon S3 Site

**Question:**
How did you create revitapisearch.com?
Did you have access to the RevitAPI.chm source documents, or did you extract them from the public CHM?
Did you use any interesting tools, tool chain or other techniques worth noting?
Would you mind very briefly outlining the process?

**Answer:** Three easy steps:

- Decompile the CHM
- Upload to Amazon S3
- Set up a Google custom search

It is quite straightforward to
[decompile a CHM file](http://msdn.microsoft.com/en-us/library/windows/desktop/ms524369(v=vs.85).aspx).
All you need to do is enter the following on the (Windows) command line:

```
  hh.exe –decompile targetfolder RevitAPI.chm
```

Try it out.
You will find the output quite easy to deal with – just a bunch of HTML files named by GUID.

From there, I went through the rather arduous process of uploading everything to Amazon S3.
It’s arduous because there are so many individual files.
Then, there are a few little steps to
[set up an S3 'bucket' as a file server](http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html),
but it’s not too tough.

Finally I had to make sure Google had an entry point to index.
I used Google Custom Search, which you can just Google to learn more about.

Many thanks to Peter for his great work and the nice succinct explanation!

#### A Week of Walking

On that happy note, let me bid you farewell for a week.

I am going hiking on
[La Palma](http://en.wikipedia.org/wiki/La_Palma),
the most northwestern of the [Canary Islands](http://en.wikipedia.org/wiki/Canary_Islands),
Spain.

![Caldera de Taburiente, La Palma](img/la_palma_caldera_de_taburiente.jpg)

Get into nature, away from concrete, cars, electricity, masses of people.

![El Roque los Muchachos, La Palma](img/la_palma_el_roque_los_muchachos.jpg)

See some stars, enjoy the full (tonight) and then waning moon.

No computer, no Internet!

I wish you lots of fun while I'm gone, and develop oodles of exciting add-ins!