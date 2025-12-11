---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.751318'
original_url: https://thebuildingcoder.typepad.com/blog/0328_add_item_to_ribbon.html
post_number: 0328
reading_time_minutes: 2
series: general
slug: add_item_to_ribbon
source_file: 0328_add_item_to_ribbon.htm
tags:
- parameters
- python
- revit-api
title: Adding Non-Commands to the Revit Ribbon
word_count: 317
---

### Adding Non-Commands to the Revit Ribbon

In between all the hullabaloo about the new version of Revit, here is some news dealing with the old one and the
[Revit Python shell](http://thebuildingcoder.typepad.com/blog/2009/12/revit-python-shell.html)
implemented by Daren Thomas.
Daren now found a way to add new items such as command buttons to a Revit ribbon panel without actually having any real underlying external command implementation to hook them up to.
He uses Reflection.Emit to simulate the missing IExternalCommand implementation.
Here is Daren's own description:

I just published a post on a technique for
[adding IronPython scripts as commands to a Revit ribbon panel](http://darenatwork.blogspot.com/2010/03/adding-canned-scripts-to-revit.html).
I'm not sure if this technique could be used for other purposes, as it mainly allows dynamically created commands to be accessible by Revit, but you never know...

The technique uses Reflection.Emit to create types that subclasses IExternalCommand with an instance field set to a different value for each type in the parameterless constructor.
These types are added to a dynamic assembly, stored to disk and can then be instantiated by Revit in the usual fashion.

Thought you might be interested...

P.S.: This still doesn't solve the problem of dynamically adding commands after IExternalApplication.OnStartup to the Ribbon, but it's a start...

P.P.S. from Jeremy on the topic of
[dynamically reloading and debugging Revit add-ins](http://thebuildingcoder.typepad.com/blog/2010/03/dynamically-load-and-debug-plugins.html) and the
[AddInManager](http://thebuildingcoder.typepad.com/blog/2010/03/addinmanager.html), which takes us back to the hullabaloo around the new release:
if you are interested in either of these two topics, you really must check out the new 2011 version of the Add-In Manager... it provides a solution! More on that soon...