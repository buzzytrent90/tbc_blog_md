---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.4
content_type: qa
optimization_date: '2025-12-11T11:44:15.400582'
original_url: https://thebuildingcoder.typepad.com/blog/1146_ifc_exporter_wiki.html
post_number: '1146'
reading_time_minutes: 4
series: general
slug: ifc_exporter_wiki
source_file: 1146_ifc_exporter_wiki.htm
tags:
- family
- revit-api
title: IFC Exporter Open Source Wiki
word_count: 717
---

### IFC Exporter Open Source Wiki

As you certainly know, the
[Revit IFC exporter is open source](http://thebuildingcoder.typepad.com/blog/2011/09/revit-ifc-exporter-released-as-open-source.html) and
available from the sourceforge
[IFC Exporter for Revit](http://sourceforge.net/projects/ifcexporter) project
repository.

It has now been updated to Revit 2015, obviously, covers import as well as export, and enhanced with a
[FAQ wiki](http://sourceforge.net/p/ifcexporter/wiki/Troubleshooting%20-%20FAQ) to
address the most frequently asked questions.

Below, I would like to present both the main IFC exporter project [description](#3) and its new [FAQ](#4).

#### Autodesk Revit Wikipedia Entry

Before getting to that, let me briefly add that I just discovered the Wikipedia entry for
[Autodesk Revit](https://en.wikipedia.org/wiki/Autodesk_Revit) itself,
which I was previously unaware of, and took the liberty of immediately adding a new short section on
[Revit programming](https://en.wikipedia.org/wiki/Autodesk_Revit#Programming) to
it.

Back to the IFC exporter news.

#### IFC Exporter Includes Import

Starting with 2015, the 'IFC Exporter for Revit' project is actually misnamed: it should be renamed 'IFC for Revit', because it includes import as well as export.
The old name still remains for the moment, since it is not trivial to change the SourceForge project name.

In the interest of file disclosure and easier understanding of the architecture and implementation: although the 'Open IFC' command goes through the open source code, it is almost entirely processed in native Revit code. The 'Link IFC' command, on the other hand, is almost entirely processed in the open source code. The long-term plan is to synchronise the two more.

#### IFC Exporter Project Description

This is the .NET code used by the Revit 2015 family of products to support IFC, and the Revit 2012-2014 families of products to support IFC export. The open source version can override the version that comes standard with shipped Revit. Revit 2013-2015 contain both IFC export and IFC export UI overrides, which contains options not available in the regular UI, such as ifcXML and ifcZIP support. The Revit 2014 and 2015 versions also work for Revit LT 2014 and 2015, respectively.

#### IFC Exporter Project FAQ

This page addresses some common problems when installing the exporter and alternate UI. Note that starting with 2015, some of these problems are no longer relevant, as the importer, exporter, and alternate UI are bundled together.

**Q:** I installed the exporter and UI, but I don't see any buttons in the add-in tab. How do I use it?

**A:** The exporter seamlessly replaces the existing "out of the box" exporter. Simply use Revit Button > Export > IFC as usual. Similarly for import, use Link IFC in the insert tab, or via the Manage Links button.

**Q:** I downloaded the exporter, but I still have the old UI. What happened?

**A:** For 2013 and 2014, there are two separate products to install: the exporter and the alternate UI. Although they work independently, they are intended to work together, and you may get only partial improvements with just one.

**Q:** I upgraded to the Revit 2014 exporter from Revit 2013, but now my exports are worse! What happened?

**A1:** The newest versions of the Revit 2014 exporter need at least Revit 2014 UR1 to run. If you are using Revit 2014 without any Update Release patches, it will silently fail and revert to the standard functionality.

**A2:** You may not have the right permissions to use the add-in. If the add-in was installed by an administrator, and you do not have administrative privileges, it may not run.

**Q:** How can I know what version of the exporter and UI I am using?

**A:** For the UI, you can find the version number on the dialog box when you export. To know if you have the right version of the exporter running, you can either look at the Control Panel in the Add/Remove Programs list, or you can do a trivial export and open up the file in Notepad. Search for FILE\_NAME and you will see the version of the Exporter and the Alternate UI. If you see an odd version number there, look through the other items on this troubleshooting page for more information.