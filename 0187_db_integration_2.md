---
post_number: "0187"
title: "Revit Integration with a Database or ERP System"
slug: "db_integration_2"
author: "Jeremy Tammik"
tags: ['family', 'geometry', 'levels', 'parameters', 'revit-api', 'views']
source_file: "0187_db_integration_2.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0187_db_integration_2.html"
---

### Revit Integration with a Database or ERP System

A while ago, I published Miroslav Schonauer's popular high-level
[overview of Revit and database integration](http://thebuildingcoder.typepad.com/blog/2009/01/database-integration.html).
Here is an additional useful note from Anthony Hauck on this topic:

**Question:**
How can I integrate Revit with an ERP solution or an SQL database?

**Answer:**
Revit information can be exported to an ODBC-compatible database.
ODBC export is a native feature of Revit.
The Labs posting
[RDBLink](http://labs.autodesk.com/utilities/revit_rdb)
demonstrates a similar capability using the API, with the additional feature of being able to change some parameters within a Revit model by editing the resulting database and re-importing the changes.
Some of these parameter changes can affect Revit model geometry.

The
[availability of RDBLink](http://thebuildingcoder.typepad.com/blog/2009/06/rfa-version-grey-commands-family-context-and-rdb-link.html#4)
has already been mentioned briefly previously.