---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:13.335952'
original_url: https://thebuildingcoder.typepad.com/blog/0085_mechanical_duct_types.html
post_number: 0085
reading_time_minutes: 1
series: elements
slug: mechanical_duct_types
source_file: 0085_mechanical_duct_types.htm
tags:
- family
- parameters
- revit-api
- elements
title: Mechanical Duct Types
word_count: 192
---

### Mechanical Duct Types

**Question:**
When selecting duct type properties, under the mechanical heading, various fitting property values are specified.
How can I find out what property a given fitting belongs to, i.e. is it an elbow, a transition or cross etc.?
Obviously, the type of fitting is described in the family name, but is there a more definite way of knowing what kind of fitting it is through the API?

**Answer:**
In Revit 2009, you can use the built-in parameter FAMILY\_CONTENT\_PART\_TYPE.
You can access it through the following series of properties:

FamilyInstance > Symbol > Family > Parameter > BuiltInParameter.FAMILY\_CONTENT\_PART\_TYPE.

This parameter stores an integer value. The values for the different types are:

```
kUndefinedPartType = -1,
kNormal            = 0,
kDuctMounted       = 1,
kJunctionBox       = 2,
kAttachesTo        = 3,
kBreaksInto        = 4,
kElbow             = 5,
kTee               = 6,
kTransition        = 7,
kCross             = 8,
kCap               = 9,
kTapPerpendicular  = 10,
kTapAdjustable     = 11,
kOffset            = 12,
kUnion             = 13,
kPanelBoard        = 14,
kTransformer       = 15,
kSwitchBoard       = 16,
kOtherPanel        = 17,
kEquipmentSwitch   = 18,
kSwitch            = 19,
kValveBreaksInto   = 20,
kSpudPerpendicular = 21,
kSpudAdjustable    = 22,
kDamper            = 23,
kWye               = 24,
kLateralTee        = 25,
kLateralCross      = 26,
kPants             = 27,
kMultiPort         = 28,
kValveNormal       = 29,
```

Many thanks to Adam Nagy and Harry Mattison for providing this information.