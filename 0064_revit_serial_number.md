---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.304471'
original_url: https://thebuildingcoder.typepad.com/blog/0064_revit_serial_number.html
post_number: '0064'
reading_time_minutes: 1
series: general
slug: revit_serial_number
source_file: 0064_revit_serial_number.htm
tags:
- revit-api
title: Revit Serial Number
word_count: 79
---

### Revit Serial Number

Here is a short and simple question with a simple answer.

**Question:** How can I determine the Revit product's serial number?

**Answer:** The Revit serial number is available in the Licpath.lic file, which is a text file located in the Revit program folder.
The content of the file is something similar to the following, where the string after # SN is the serial number:

```
## Autodesk Revit
## License Information
# SN 123-45678910
# NSN 000-00000000
# Standalone
```