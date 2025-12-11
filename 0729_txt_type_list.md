---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.2
content_type: qa
optimization_date: '2025-12-11T11:44:14.474875'
original_url: https://thebuildingcoder.typepad.com/blog/0729_txt_type_list.html
post_number: 0729
reading_time_minutes: 1
series: elements
slug: txt_type_list
source_file: 0729_txt_type_list.htm
tags:
- family
- parameters
- references
- revit-api
- elements
title: A TXT File may be a Type List
word_count: 268
---

### A TXT File may be a Type List

Here is a short note on a little issue to be aware of by Chuck Dodson of
[SGA Design Group](http://www.sgadesigngroup.com):

**Question:** I am stuck trying to figure out the meaning of this error message:

![TXT type list error message](img/txt_type_list_err.png)

I am creating a family in code.
I get the same problem if I create it manually.

Do you have any idea what the message is saying?

Looking online, I find comments about Shared Parameter file corruption, but I'm not using any.

The odd thing is, if I rename the family, it loads fine.
I can drag it from explorer; fine.
I can load it from the open family file; fine.
But if I use the Load Family button, I get the message, although it does actually load.

Any ideas?

**Answer:** I solved this.

I was writing a text file (.txt) in the same folder that my families were being created in.
For some reason, Revit assumes it is a shared parameter file.

I changed the filename extension of the ASCII files I was writing to DEF instead of TXT and the problem is solved.

The '(' in the message above is a reference to a parenthesis starting on the first line of the text file.
Thought you might want to know.

Jeremy adds: yes, I did know, in fact, and so do most family developers, but possibly a number of API newbies do not, so I am sure that this is information worth publishing and sharing.
Thank you very much for that, Chuck!