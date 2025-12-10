---
post_number: "0615"
title: "VendorId, DevTV, and My First Plugin"
slug: "vendorid"
author: "Jeremy Tammik"
tags: ['revit-api']
source_file: "0615_vendorid.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0615_vendorid.html"
---

### VendorId, DevTV, and My First Plugin

Here is a simple issue from a discussion with Stephen LeCompte of
[FKP Architects, Inc.](http://www.fkp.com) that
several other people people also recently ran into:

**Question:** I followed the steps provided in 'DevTV: Introduction to Revit 2011 Programming - Part I' and incorporated them in my Revit Architecture 2012 64-bit version, but there is a problem.
When I start up Revit 2012 it displays the following error message saying 'Failed to initialize the add-in "RevitNETAddin3.addin" because the add-in registration is missing the required value of VendorId node. The VendorId note identify the vendor of the add-in application. For Revit to run the add-in, you must register the node defined in the manifest "RevitNETAddin3.addin" file.'
![VendorId error message](img/VendorIdErrorMsg.png)

The DevTV video says nothing at all about the need to add a VendorId node.
I researched what this VendorId is through Google and found the page on
[Add-in Registration](http://wikihelp.autodesk.com/Revit/enu/2012/Help/API_Dev_Guide/0000-API_Deve0/0001-Introduc1/0018-Add-In_I18/0022-Add-in_R22)
in the Revit API WikiHelp.

It states I have to register a developer symbol, which I did.

Even after doing so, I still get the same error message from Revit.

How can I solve this, please?

**Answer:** The issue is a lot less complex than it sounds, actually.

It is a good thing for you that you registered your developer symbol, but that does not make any difference to the loading behaviour of the Revit add-in.
For the loading, it is only important that the VendorId tag is present in the add-in manifest file, regardless of its contents.
The manifest file RevitNETAddin3.addin that you attached looks perfectly correct to me.

If you are still seeing the error message above, then the only explanation I can think of is that you have not copied the updated version of the manifest file to the location where Revit is searching for it.
I would suggest doing a global search for this file across your entire system and eliminating all unwanted duplicates.

**Response:** I found the solution!

The problem was due to me using a tag named VendorID, and not VendorId with a lower-case 'd'.

Now I've adjusted my Revit template and the places where the .addin and .dll get created to avoid confusion in the future.
Everything is working great and ready to create some new plug-ins.

Suggestion: The latest Revit DevTV video-Part does not mention or show the  tags, so it is a little confusing when people try with Revit 2012.
Could you add a comment to the download page to warn people of that difference?

**Answer:** Yes, as you discovered the hard way, [XML tags are case sensitive](http://www.w3schools.com/xml/xml_syntax.asp).
Sorry for not noticing that error in your sample manifest file.

Therefore, VendorID was not accepted, and VendorId needs to be specified exactly so.

I am very glad your Revit template works now.
Are you aware of the one I use?

- [Visual Studio add-in wizards for Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/04/visual-studio-add-in-wizards-for-revit-2012.html).- [Updated VB wizard](http://thebuildingcoder.typepad.com/blog/2011/06/implicit-line-continuation-in-vb-2010.html#2) using Visual Studio 2010 line continuation feature.

You are completely correct, the DevTV videos were created using Revit 2011, and do not mention the new VendorId tags required in Revit 2012.
Yes, that feature is new in Revit 2012. It is highlighted in the
[Revit 2012 API news](http://thebuildingcoder.typepad.com/blog/2011/03/revit-2012-api-features.html) and the
[webcast recording](http://thebuildingcoder.typepad.com/blog/2011/06/the-revit-mep-2012-api.html#2).

I do not believe that we will be updating the DevTV training video for Revit 2012, since the differences are small, but I completely agree that this one difference is a real stumbling block and should be prominently highlighted in the download page.

On the other hand, I would actually expect people to have a look at the
[Revit Developer Centre](http://www.autodesk.com/developrevit) and the
[My first Revit plug-in](http://www.autodesk.com/myfirstrevitplugin) first of all, nowadays.
That is based on Revit 2012 and thus includes an explanation of the add-in manifest file VendorId tag.