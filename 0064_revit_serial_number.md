---
post_number: "0064"
title: "Revit Serial Number"
slug: "revit_serial_number"
author: "Jeremy Tammik"
tags: ['revit-api']
source_file: "0064_revit_serial_number.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0064_revit_serial_number.html"
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