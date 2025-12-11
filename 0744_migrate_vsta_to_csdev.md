---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: code_example
optimization_date: '2025-12-11T11:44:14.505065'
original_url: https://thebuildingcoder.typepad.com/blog/0744_migrate_vsta_to_csdev.html
post_number: '0744'
reading_time_minutes: 2
series: general
slug: migrate_vsta_to_csdev
source_file: 0744_migrate_vsta_to_csdev.htm
tags:
- csharp
- elements
- references
- revit-api
- schedules
- vbnet
title: Migrating VSTA Macros to SharpDevelop
word_count: 442
---

### Migrating VSTA Macros to SharpDevelop

As you probably noticed, we held a very successful and exciting Revit API training course and DevLab
[recently](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-one.html)
[in](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html)
[Melbourne](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-devlab.html).

More face-to-face hands-on trainings are planned, as you can see from the ADN
[Training Course Schedule](http://www.adskconsulting.com/adn/cs/api_course_sched.php),
entering "Revit API" as the course name.

#### Revit API Hands-On Training in Munich

The next scheduled class that I will be conducting is taking place in Munich, Germany, on April 25-26
([register](http://usa.autodesk.com/adsk/servlet/item?id=6703509&siteID=123112&cname=Revit%20API,%20Munich,%20Apr%2025%202012,%20201213)).

If you are interested in participating in a class, you will benefit enormously by
[preparing appropriately](http://thebuildingcoder.typepad.com/blog/2012/01/preparing-for-a-hands-on-revit-api-training.html).
The material we provide for that can also be used very well to learn the basics of the Revit API for yourself all on your own.

Meanwhile, here are some notes on the changes required to migrate our existing
[My First Revit Plugin](http://thebuildingcoder.typepad.com/blog/2011/05/my-first-revit-plug-in.html)
([VB](http://thebuildingcoder.typepad.com/blog/2011/11/my-first-revit-plug-in-in-vb.html))
code from Revit 2012 to 2013, reported by Saikat Bhattacharya:

#### Migrating VS 2010 Express Code

The only change was to update the following line of code using the obsolete Document.Element property:

```
  Element elem = pickedRef.Element;
```

This needs to be changed to call the Document.GetElement method:

```
  Element elem = doc.GetElement(pickedRef);
```

#### Migrating VSTA to SharpDevelop

There were a couple of steps in this migration:

**1.** Open the RVT files with embedded macros in SharpDevelop.

**2.** Change the .NET framework version to 4.0:
![Set .NET Framework version](img/vsta_to_csdev_1.png)

**3.** Remove existing references to the Revit API assembly DLLs and reference the Revit 2013 ones instead.

**4.** Delete the following line, since VSTA no longer exists in the Revit namespace:
![Remove VSTA add-in id](img/vsta_to_csdev_2.png)

**5.** Make the API changes in the code, essentially just the one line mentioned above for the VS Express change:
![Update Element property to GetElement method](img/vsta_to_csdev_3.png)

**5.** Update the description of the macro in the MacroManager dialogue.

I repeated the same set of steps for all the existing VSTA samples for C# and VB.NET for them to work with SharpDevelop.

Many thanks to Saikat for this report!