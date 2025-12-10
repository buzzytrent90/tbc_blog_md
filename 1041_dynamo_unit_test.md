---
post_number: "1041"
title: "The Dynamo Revit Unit Test Framework"
slug: "dynamo_unit_test"
author: "Jeremy Tammik"
tags: ['revit-api']
source_file: "1041_dynamo_unit_test.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1041_dynamo_unit_test.html"
---

### The Dynamo Revit Unit Test Framework

I have repeatedly underlined the importance of unit testing, and recently presented the
[RvtUnit project](http://thebuildingcoder.typepad.com/blog/2013/07/revit-add-in-unit-testing.html#3) enabling the use of the
[NUnit unit testing tool for Revit add-ins](http://thebuildingcoder.typepad.com/blog/2013/07/revit-add-in-unit-testing.html).

The Dynamo team now presents another one; in case you have not already heard of and explored it, Dynamo is an exciting visual programming tool on top of Revit.

The best explanation of Dynamo is provided in the
[Dynamo GitHub repository](https://github.com/ikeough/Dynamo).
Much of the discussion and suggestions for new features etc., is happening on the github issues page, but there are also interesting discussions, links to the downloads, and blog posts by the Dynamo team on
[www.dynamobim.org](http://www.dynamobim.org).

The Dynamo team just released a new interesting piece of functionality that can provide a huge benefit to all developers writing code against the Revit API.

It's called the
[Dynamo Revit Test Framework](https://github.com/ikeough/Dynamo/wiki/Dynamo-Revit-Test-Framework) and
enables automated testing of Revit API functionality using isolated Revit sessions.
It is used internally for Dynamo testing on top of Revit.
Given a bit more love, it could be extended as a generic Revit automation framework.
And, because all work being done on Dynamo is open source, the source is available and it's completely free!

To learn more, please visit the
[Dynamo Revit Test Framework](https://github.com/ikeough/Dynamo/wiki/Dynamo-Revit-Test-Framework) GitHub wiki page.

The Dynamo team really appreciates all feedback on Dynamo as a tool for both developers and non-developers striving to automate Revit.
Please let them know if you have any questions about the project.