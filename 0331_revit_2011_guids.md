---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:13.756759'
original_url: https://thebuildingcoder.typepad.com/blog/0331_revit_2011_guids.html
post_number: '0331'
reading_time_minutes: 4
series: general
slug: revit_2011_guids
source_file: 0331_revit_2011_guids.htm
tags:
- revit-api
- schedules
- selection
title: Revit 2011 Product GUIDs
word_count: 802
---

### Revit 2011 Product GUIDs

We discussed issues related to the
[Revit install path and the product GUIDs](http://thebuildingcoder.typepad.com/blog/2009/02/revit-install-path-and-product-guids.html) for
versions up to and including Revit 2010 last year.
The install path is less of an issue nowadays, since there is no longer a need to determine the location of Revit.ini to install an add-in, because you can use an add-in manifest file instead.
Another more exciting reason is the availability of the new
[add-in utility DLL](#1)
described below.
Still, the product GUID may be important for many other uses, so here is the updated list including the GUIDs for the Revit 2011 versions:

|  |  |  |  |
| --- | --- | --- | --- |
| 2011 | 64 bit | RAC | 94D463D0-2B13-4181-9512-B27004B1151A |
| RME | C31F3560-0007-4955-9F65-75CB47F82DB5 |
| RST | 23853368-22DD-4817-904B-DB04ADE9B0C8 |
| 32 bit | RAC | 4AF99FCA-1D0C-4D5A-9BFE-0D4376A52B23 |
| RME | CCCB80C8-5CC5-4EB7-89D0-F18E405F18F9 |
| RST | 0EE1FCA9-7474-4143-8F22-E7AD998FACBF |
| 2010 | 64 bit | RAC | 2A8EEE2F-4A9E-43d8-AA07-EC8A316B2DEB |
| RME | A1BD042B-8A6F-4e37-92A3-78921BB45B05 |
| RST | BC9C0A08-DEA4-4138-A7FB-8F68866DB0C1 |
| 32 bit | RAC | 572FBF5D-3BAA-42ff-A468-A54C2C0A17C3 |
| RME | 5C8281B1-B927-495a-A0FF-AB4BDFAE505C |
| RST | 939D29FC-B82D-42a7-BB1E-8E3F121505CC |
| 2009 | 64 bit | RAC | D2466208-7348-4214-B01E-7BC8729E2BD3 |
| RME | 4A98F976-01B5-40e8-A496-AEFD85C3A446 |
| RST | B354FCF5-CF64-4fa2-AA84-9D9B2A6FA649 |
| 32 bit | RAC | A3A37DA6-70C0-497C-BCB1-148E9EC1D32E |
| RME | E3781DCB-A650-4E66-9B74-67A1B17F052C |
| RST | C4B3B3C3-2EE9-48D3-9BF5-4443F7ECF759 |
| 2008 | RAC | 4A11206C-4377-49E8-911E-B11548658FF3 |
| RME | 60A2743E-C881-4880-94ED-96445E38616F |
| RST | 8D0AE0BB-4FE5-491D-A284-3B363F02E639 |
| 9.0 | Revit Building | D11DB6CB-0332-4735-B312-B919741D975E |
| 3 | RST | 3F11CEE0-D30D-41ce-8522-922B5D8BB324 |
| 8.1 | Revit Building | 7EBC0489-5E47-498D-BE31-B094484612E9 |
| 2 | RST | BE814F63-629D-4fd8-B628-1437AC10F9D4 |

ADN members may also refer to the technical solution
[TS87598 [How to detect where Revit has been installed?]](http://adn.autodesk.com/adn/servlet/devnote?siteID=4814862&id=9647178&linkID=4901650)

#### RevitAddinUtility

In the past, the Revit product GUIDs were often used to determine the Revit installation location.
In Revit 2011, this can be achieved a lot simpler and safer by making use of the new RevitAddinUtility functionality.

RevitAddInUtility.dll is a new .NET utility class assembly which lives in the Revit Program folder, in the same location as Revit.exe, Revit.ini, and the Revit API DLLs.

The Revit SDK provides documentation on how to use it in its own little help file RevitAddInUtility.chm, as well as a sample application RevitAddInUtilitySample demonstrating its use.

The latter is located in the ExternalCommand2011 folder, which contains two separate very interesting sample applications:

- RevitAddInUtilitySample- ExternalComandRegistration

Here is an excerpt from the documentation of these two in 'ReadMe\_ExternalCommand 2011.docx':

Two samples with the following functionality demonstrate how to use the new external command registration more effectively:

- RevitAddInUtilitySample: Show how to use RevitAddInUtility to create and edit an add-in manifest file, retrieve information from the manifest file, and retrieve installed Revit product information.- New features of external command registration:
    1. Visibility mode: demonstrate how to control the visibility of each external command based on the different product and document types.- IAvailabilityClass: demonstrate how to dynamically enable or disable individual external commands bases on the current user's selection or other application information.- Icon and tooltip: demonstrate how to define an external command's ribbon icon and tooltip.- Localization: demonstrate how to localise strings in the add-in manifest file.

#### Enthusiasm and Namespaces

Guy Robinson gives vent to some
[enthusiasm](http://redbolts.com/blog/post/2010/03/25/Yeah-Baby!!-Revit-2011-API-e28093-the-unofficial-4th-Discipline.aspx) about
Revit 2011 and the new API
and provides a lot of interesting background information on and a pointer to a powerful tool for handling the
[namespace refactoring](http://redbolts.com/blog/post/2010/03/27/Revit2011-API-e28093-Whate28099s-in-a-name.aspx) in
the Revit 2011 API, which might save a significant amount of porting time and effort.

#### Revit 2011 API News Webcast

Kean Walmsley points out that we have yet another group of events that I forgot to mention on Sunday besides the
[DevCamp, Devlabs and API training classes](http://thebuildingcoder.typepad.com/blog/2010/03/devcamp-devlabs-and-updated-api-training-schedule.html):
[free online sessions on the 2011 products and their APIs](http://through-the-interface.typepad.com/through_the_interface/2010/03/free-online-sessions-on-the-2011-products-and-their-apis.html).
The session on what's new in the Revit 2011 API is taking place on April 21st, and once again you can visit our
[training schedule](http://www.adskconsulting.com/adn/cs/api_course_sched.php) (also accessible via
[autodesk.com/apitraining](http://autodesk.com/apitraining) > Schedule) to attend.
He also points to some other product related sessions that may be interesting to you.