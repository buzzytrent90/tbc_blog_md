---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.096046'
original_url: https://thebuildingcoder.typepad.com/blog/0521_vault.html
post_number: '0521'
reading_time_minutes: 4
series: general
slug: vault
source_file: 0521_vault.htm
tags:
- family
- levels
- revit-api
title: Using Vault with Revit
word_count: 830
---

### Using Vault with Revit

I recently made a brief mention of the
[Navisworks](http://thebuildingcoder.typepad.com/blog/2011/01/create-a-navisworks-file-from-revit.html) product
and how a native Navisworks file can be created from within a Revit add-in.

Today, let's have a look at another Autodesk product, Vault, and how that can be used in conjunction with Revit.

In case you do not know Vault, it is an Autodesk data management product and has a public API.
Here are links to the
[Autodesk Vault product page](http://www.autodesk.com/vault) describing
the software and its capabilities and the
[Vault developer center](http://www.autodesk.com/developvault) describing
its API and programming tools.

Here is a question that recently came up on using Vault with Revit:

**Question:** I need to manage and distribute confidential project files including Revit model data on a server and have some questions regarding this:

- Is Vault a viable solution for this?- Could I drive Vault through the API to control the data storage and access?- What is the Vault licensing situation for normal Revit users?

**Answer:** Yes, Vault can be used to store files of any type, so it can be used to manage the Revit model information as well.

Yes, you can use the Vault API to control the storage of data into Vault.

Vault can be used in the Revit context, as explained in this
[video](http://www.youtube.com/watch?v=V4kzhZoewrI):

To evaluate it, I would suggest having a look at the Vault product itself.

As far as I know, the Vault Explorer is implemented only using the public Vault API, so anything that can be done there through the UI can also be done programmatically.

Actually, I learned that all Vault clients are built on top of the Vault API.
So, in theory, anything you see can be done in the UI can be done programmatically.
However, some things are easier to do than others.
A UI command may run a single API call or it may be a complex algorithm involving several API calls.

A better way of describing the web service API in Vault is that it can be used to get or set any piece of Vault server data.
It's the bridge that lets you talk to the server.

For introductory material on Vault and its API, please refer to

- This [webcast](http://adn.autodesk.com/adn/servlet/item?siteID=4814862&id=15166281&linkID=4901778).- The samples provided in the Vault install folder.- Doug Redmond's
      [Vault blog](http://justonesandzeros.typepad.com).

Regarding the samples, please note that for each given Vault product, there are two Vault installs, a Vault client and a Vault server.
The SDK is part of the Vault server installation only.

As far as I know, for simple project file management, the standard Vault version will be sufficient and you will have no need for Vault Professional.

The Vault product page listed above, which is also accessible through the 'Autodesk Vault' link on Doug's blog, includes a link to the
[product brochure](http://images.autodesk.com/adsk/files/vault_family_detail_brochure_us.pdf) which
provides answers to several of the issues you address.

Regarding the licensing situation, there are four different Vault products.
A basic version of Vault is bundled with various CAD applications at no extra charge.
Furthermore, there are three levels of upgrades that must be purchased: Vault Workgroup, Vault Collaboration, and Vault Professional.

The base version of Vault requires that the client machine have a qualifying CAD application installed on that machine.
In other words, the CAD license is also the Vault license since Vault was bundled with the CAD application.
It is not OK to write a Vault client for users without a qualifying CAD application.
Here is the full list of qualifying CAD applications (all of them are version 2011):

- Autodesk 3DS Max- Autodesk 3DS Max Design- Autodesk Inventor (Stand Alone)- Autodesk Inventor Suite- Autodesk Inventor Professional- Autodesk Inventor Simulation- Autodesk Inventor Routed Systems- Autodesk Inventor Tooling- AutoCAD Mechanical- AutoCAD Electrical- AutoCAD- AutoCAD Map- AutoCAD Civil 3D- AutoCAD Architecture- AutoCAD MEP- AutoCAD P&ID

Unfortunately, Revit is currently not a qualifying CAD application.
In order to use Vault with a pure Revit installation, you cannot use the basic version but have to purchase one of the other three ones.

Obviously, if you have a Revit suite including AutoCAD, then one would assume that it qualifies.
The best test is to run the Vault client.
If no qualifying product exists, then the installer will block the install.
If the Vault client installs properly, it's usually safe to assume that you are licensed to use Vault.

The purchased versions of Vault have different licensing rules.
Network licensing is used and the Vault server keeps track of the usage.
There is no qualifying CAD application rule.
Anyone can connect in to Vault.
If there are no free licenses available, the server will deny the login.