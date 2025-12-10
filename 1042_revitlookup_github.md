---
post_number: "1042"
title: "RevitLookup is on GitHub and You are Invited to Collaborate"
slug: "revitlookup_github"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api']
source_file: "1042_revitlookup_github.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1042_revitlookup_github.html"
---

### RevitLookup is on GitHub and You are Invited to Collaborate

I sincerely hope that everybody reading this is aware of the power and importance of RevitLookup, has already installed it, and is an avid user.

If you do not know what I am talking about, please read these previous posts on the topic, noting that it was previously named RvtMgdDbg, short for 'Revit managed debugging tool':

- [RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html)- [Fixing RvtMgdDbg for MEP Connectors](http://thebuildingcoder.typepad.com/blog/2009/08/fixing-rvtmgddbg-for-mep-connectors.html)
  - [RvtMgdDbg for Revit 2010](http://thebuildingcoder.typepad.com/blog/2009/10/rvtmgddbg-for-revit-2010.html)
  - [RevitLookup Update](http://thebuildingcoder.typepad.com/blog/2010/05/revitlookup-update.html)
  - [Units and RevitLookup](http://thebuildingcoder.typepad.com/blog/2013/03/units-and-revitlookup.html)
  - [Getting Started with the Revit API](http://thebuildingcoder.typepad.com/blog/2013/04/getting-started-with-the-revit-api.html)

RevitLookup is currently included in the Revit SDK or Software Developer Kit that can be downloaded from the
[Revit Developer Centre](http://www.autodesk.com/developrevit).

In a
[comment](http://thebuildingcoder.typepad.com/blog/2013/06/the-adn-sample-adnrme-for-revit-mep-2014.html?cid=6a00e553e1689788330191034be6cc970c#comment-6a00e553e1689788330191034be6cc970c) on the
[AdnRme sample](http://thebuildingcoder.typepad.com/blog/2013/06/the-adn-sample-adnrme-for-revit-mep-2014.html),
Matt Mason of
[IMAGINiT Technologies](http://imaginit.com/software-solutions/building-architecture/IMAGINiT-Clarity-Workshare) suggested
hosting it on GitHub instead, to facilitate community enhancements.

I talked this over with the development team and now finally got around to doing it.

#### RevitLookup GitHub Repository

RevitLookup now lives in its own
[RevitLookup repository](https://github.com/jeremytammik/RevitLookup) on GitHub.

I already integrated some enhancements for structural models provided by Thomas Fink of
[SOFiSTiK AG](http://www.sofistik.com).

I made Matt Mason a collaborator on this project and he added some generic enhancements of his own as well, with no need for me to intervene at all in any way.

The current version on GitHub thus already sports some improvements over the standard SDK one, so I would suggest you upgrade to that right away.
Simply clone it to your desktop, compile, and replace your existing .NET assembly DLL with the new one.

#### Invitation to Collaborate

I am happy to welcome anyone else as well who would like to collaborate on this project.

To begin with, however, to get to know you a little, I would suggest that you use 'Fork & Pull' approach as opposed to the 'Shared Repository' approach, so that I can see what you plan to do.

For details on how to go about this, please look at

- [Using Pull Requests](https://help.github.com/articles/using-pull-requests)
- [How to GitHub: Fork, Branch, Track, Squash and Pull Request](http://gun.io/blog/how-to-github-fork-branch-and-pull-request)

Before you can contribute anything at all, of course, you need to discover a problem and fix it.

Here are some words from an old ADN case on such an issue:

#### Fixing RevitLookup

**Question:** I have used RevitLookup with Revit Architecture without any problems.

When I use it with Revit Structure it works as long as I do not try to snoop a rebar.

Is there a version of RevitLookup that can snoop rebar elements?

Without a software like RevitLookup it would be very time consuming to learn how to access rebar data.

**Answer:** You can fix any problems that you encounter in RevitLookup yourself, if you like.

Simply run RevitLookup in the Visual Studio debugger, select the offending Revit element, and see what is causing the problem.

Normally, an exception is thrown, which terminates the operation.

If you add an exception handler around the offending line, then all the other information will still be displayed.

That is a cheap fix and workaround solution.

A better solution, of course, is to implement an improved handling of the problematic element, which is often also quite simple.

I haven discussed several such improvements in the past on the blog, e.g.,
[fixing RevitLookup for MEP connectors](http://thebuildingcoder.typepad.com/blog/2009/08/fixing-rvtmgddbg-for-mep-connectors.html).

You have the source. The power is in your hands.

With the new repository in place, any useful enhancements can easily be integrated into the main stream for all to share and enjoy.

Good luck and have fun debugging and enhancing RevitLookup together!