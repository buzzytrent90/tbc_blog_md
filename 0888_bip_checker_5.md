---
post_number: "0888"
title: "BipChecker Update"
slug: "bip_checker_5"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'parameters', 'revit-api', 'walls']
source_file: "0888_bip_checker_5.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0888_bip_checker_5.html"
---

### BipChecker Update

BipChecker is a tool that I would recommend every Revit developer to install, just like RevitLookup.
It simply lists the values of all parameter data stored on a selected element.

Similar access is also offered within RevitLookup by the option Parameters > Built-in Enums Snoop...
BipChecker offers much more functionality, though, and is a really powerful Revit database exploration and debugging tool.

The built-in parameter checker has been part of the ADN sample labs for ages now, and the original version is still provided in the
[Xtra materials](http://thebuildingcoder.typepad.com/blog/2012/04/xtra-adn-revit-2013-api-training-labs.html).
The time has come to finally remove it from there, though.

First, since I find it useful on its own, I extracted it into a separate add-in and baptised it
[BipChecker](http://thebuildingcoder.typepad.com/blog/2011/09/unofficial-parameters-and-bipchecker.html).

Secondly,
[Victor Chekalin](http://www.facebook.com/profile.php?id=100003616852588) enhanced
it by implementing grouping of the long list by parameter group, data type, built-in versus standard Parameters collection, read-write, or none.

This tool always suffered one serious problem, though, due to repetitions in the definitions and assignment of some of the built-in parameter enumeration integer values.

Victor now tracked down and fixed that issue as well.

Here is his explanation:

Now I work closely with Materials and use our BiP checker.
I noticed two strange things when I check material parameters using BiP Checker.
The first one, the material cost stored as BuiltInParameter.DOOR\_COST.
The second one – some parameters appear twice or even triple:

![BipChecker duplicate and missing values](img/BipChecker05_problem.png)

I researched this and found the reason.

To iterate all built-in parameters, we used the Enum.GetValues method:

```csharp
var allParameters = Enum.GetValues(
typeof ( BuiltInParameter ) );
```

This makes use of the integer values of the Enum.
If different Enum values have the same integer value the GetValues method doesn’t work properly.

For example, BuiltInParameter.ALL\_MODEL\_COST and BuiltInParameter.DOOR\_COST both have the integer value -1001205:

- DOOR\_FIRE\_RATING = -1001206- ALL\_MODEL\_COST = -1001205- DOOR\_COST = -1001205- ALL\_MODEL\_MARK = -1001203

In this case, BiPChecker shows DOOR\_COST twice and doesn’t show ALL\_MODEL\_COST parameter at all.

#### The Solution

Use Enum.GetValues and Enum.TryParse instead of Enum.GetValues to get the BuiltInParameter value, and use the BuiltInParameter name from Enum.GetValues.
This requires the .NET framework 4.

Now it looks much better:

![BipChecker corrected values](img/BipChecker05_solution.png)

Here is
[BipChecker05.zip](zip/BipChecker05.zip) containing
the full source code, Visual Studio solution and add-in manifest of the updated BipChecker version 2013.0.5.0.

С уважением from Чекалин Виктор.

Many thanks to Victor for his research and important enhancement!

To wrap it up, here is a snapshot listing all built-in parameters on a wall with no grouping enabled, so that the results can be sorted across all groups by any desired criterion:

![BipChecker on my old XP machine running in Parallels on the Mac](img/BipChecker05_jt_xp.png)