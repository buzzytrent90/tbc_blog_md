---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: qa
optimization_date: '2025-12-11T11:44:13.935373'
original_url: https://thebuildingcoder.typepad.com/blog/0431_add_materials.html
post_number: '0431'
reading_time_minutes: 6
series: materials
slug: add_materials
source_file: 0431_add_materials.htm
tags:
- elements
- filtering
- parameters
- revit-api
- sheets
- views
- materials
title: Add New Materials from List
word_count: 1102
---

### Add New Materials from List

I am happily back from
[Siem Reap](http://en.wikipedia.org/wiki/Siem_Reap) in
Cambodia and the
[ruins of Angkor Wat](http://en.wikipedia.org/wiki/Angkor_Wat)
([more](http://www.facebook.com/album.php?aid=68525&id=1019863650)
[photos](http://www.facebook.com/album.php?aid=68592&id=1019863650)):

![Angkor Wat](/j/photo/jeremy/2010/2010-08-25_siem_reap/p110.jpg)
![Angkor Wat](/j/photo/jeremy/2010/2010-08-25_siem_reap/p133.jpg)
![Angkor Wat](/j/photo/jeremy/2010/2010-08-25_siem_reap/p155.jpg)
![Angkor Wat](/j/photo/jeremy/2010/2010-08-25_siem_reap/p157.jpg)
![Angkor Wat](/j/photo/jeremy/2010/2010-08-25_siem_reap/p188.jpg)

Yesterday I visited the
[Grand Palace](http://en.wikipedia.org/wiki/Grand_Palace) and
the
[reclining Buddha in Wat Pho](http://en.wikipedia.org/wiki/Wat_Pho) in Bangkok
([more photos](http://www.facebook.com/album.php?aid=68594&id=1019863650)):

![Grand Palace](/j/photo/jeremy/2010/2010-08-25_siem_reap/p258.jpg)
![Grand Palace](/j/photo/jeremy/2010/2010-08-25_siem_reap/p272.jpg)
![Reclining Buddha](/j/photo/jeremy/2010/2010-08-25_siem_reap/p298.jpg)
![Reclining Buddha](/j/photo/jeremy/2010/2010-08-25_siem_reap/p299.jpg)

Angkor Wat and Cambodia was absolutely wonderful, and I am really glad that I went there.
The places I visited in Thailand were rather touristy, whereas in Cambodia I had the feeling of getting much more in touch with real people and life:

![Child](/j/photo/jeremy/2010/2010-08-25_siem_reap/p196.jpg)
![Children](/j/photo/jeremy/2010/2010-08-25_siem_reap/p220.jpg)
![Temple](/j/photo/jeremy/2010/2010-08-25_siem_reap/p230.jpg)
![People](/j/photo/jeremy/2010/2010-08-25_siem_reap/p233.jpg)
![Monkey](/j/photo/jeremy/2010/2010-08-25_siem_reap/p241.jpg)

Before going there I was thinking that I am not really all that interested in this kind of travelling, but the connection that I experienced there really motivated me to return again, and possibly visit Laos as well.
One nice idea would be to go for a walk from northern Thailand along the western border of Laos all the way up to China.

Now I am staying with my brother Marcus in Pattaya again, picking up my computer before returning to Switzerland.

So, returning to Revit programming topics after this wonderful break from computers and the heart-warming dive into loving friendliness of Cambodian people...

#### Importing and Creating New Materials from a List

Quite a while back, I discussed programmatically
[adding a new material](http://thebuildingcoder.typepad.com/blog/2009/05/new-material.html) to the Revit project.
I would like to present a useful real-world application that takes this much further, created by Jinsol Kim of
[RCMS GROUP](http:www.rcmsgroup.com) to
add an entire list of materials read from an Excel spreadsheet to a Revit project.
Here is part of the discussions we had during the development and Jinsol's detailed description of the final process:

**Question:** I am trying to add some material lists programmatically and running into some problems here.
The materials are defined in Excel spreadsheets used as input and containing data such as

- Material Name
  - Code- Title- Strength- Graphics
    - RGB colour- Rendering data- Identity
      - Filter criteria- Descriptive information- Custom parameters

I found a way to add materials like this:

```
  string newName = "03 15 00";

  MaterialOther myMaterial
    = doc.Settings.Materials.AddOther(
      newName );

  myMaterial.Color = new Color( 127, 127, 127 );
  myMaterial.Transparency = 0;
```

However, I cannot find out any parameters related to 'Identity'.
In order to classify the material lists, I need to put values on 'Material Class'.
How can I access that value, i.e. Materials > Identity > Material Class (Filter Criteria)?

**Answer:** As said, we did discuss
[adding a new material](http://thebuildingcoder.typepad.com/blog/2009/05/new-material.html) quite
a while back.

For the identity fields, you could simply add your own shared parameters to the model.

Using shared parameters, you can add arbitrary data to all elements of any category.

**Question:** One more thing:
If I create a shared parameter named 'Material Class' for example under the Materials and finishes group, then there seems to be no way for users to use the Material Class filter.
Using the API, I will add up to about 2000 materials at once to avoid typing in every material in each project.
Users might want to see only specific materials which belong to specific material class by filtering them out with material class.
If the material class doesn't support filter by material class, then do you think I need to create a custom filter separately for it?

Here are the expected results, although created manually in this case:

![Added materials](img/add_materials.png)

**Answer:** That looks a bit similar to the view filter, for which there is full API access as demonstrated by the ElementFilter/ViewFilters sample.
Unfortunately, it looks to me as if the materials filter is not accessible in the same way.

**Question:** I chose another way to solve how to maintain the material class name in the identity data of Materials.

First, I created a set of root materials manually with the proper Material Class and then duplicated them accordingly to appropriate master format division.

Here is a more detailed description:

**Step 1:** Set up the basic materials manually within the project file to duplicate with 'Material Class'.

Note: The Material Class name should follow this format: 'CSI XX'.

![Basic materials](img/add_materials_step1.png)

**Step 2:** Check all the material information in defined in the right format.

The input spreadsheet is read from C:\RevitAPI\MaterialList.xlsx.

Note 1: Graphics section doesn't allow an empty space. It should be filled with '0' or N/A.

Note 2: Material Class values should be equivalent to basic material class that will be used for duplication.

![Input spreadsheet](img/add_materials_step2.png)

**Step 3:** Run the add-in and check that the materials were added successfully.

![Run the add-in](img/add_materials_step3a.png)

![Add-in message](img/add_materials_step3b.png)

![Added materials](img/add_materials_step3c.png)

Here is Jinsol's original
[Powerpoint version](zip/AddMaterials.pptx) of
the detailed description, and here is
[AddMaterials01.zip](zip/AddMaterials01.zip) containing
the complete Visual Studio solution and source code of the AddMaterials Revit add-in.

Thank you very much, Jinsol, for creating this solution and documenting and sharing it with us!

**Addendum:** This utility has been updated.
Please check out the
[migration to Revit 2014](http://thebuildingcoder.typepad.com/blog/2014/03/adding-new-materials-from-list-updated.html) and the
[enhancements in release 2014.0.0.1](http://thebuildingcoder.typepad.com/blog/2014/03/adding-new-materials-from-list-updated-again.html).