---
post_number: "0194"
title: "Default Family Template Path"
slug: "family_template_path"
author: "Jeremy Tammik"
tags: ['family', 'revit-api']
source_file: "0194_family_template_path.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0194_family_template_path.html"
---

### Default Family Template Path

**Question:**
Is any API exposed to read and write the default family template path?
It is displayed under 'Default path for family template files' in the RAC 2010 options dialogue:

![Default family template path](img/family_template_path_4.png)

**Answer:**
Unfortunately, there is currently no way to set the path in the options dialog at runtime through the Revit API.
You can read it from the Revit.ini file, though.
It is stored under the FamilyTemplatePath key in the [Directories] section.