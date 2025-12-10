---
post_number: "0271"
title: "Revit Python Shell"
slug: "revit_python_shell"
author: "Jeremy Tammik"
tags: ['elements', 'python', 'revit-api', 'windows']
source_file: "0271_revit_python_shell.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0271_revit_python_shell.html"
---

### Revit Python Shell

Here is number two of my collection of Christmas presents.
This one may actually be the biggest, and should give us all ample material to play with over the holiday season and indeed also use for productive work indefinitely onwards.

Daren Thomas of the [Professur für Gebäudetechnik, Institut für Hochbautechnik](http://www.gt.arch.ethz.ch) at the technical university ETH Zürich has published a
[Python Shell for Revit](http://darenatwork.blogspot.com/2009/12/introducing-revitpythonshell.html).
It was implemented using IronPython and is used to automate the running of daily tests of a building energy analysis package.
Hosting IronPython via a Revit plug-in is a now solved problem.
Daren's intention is to continue publishing samples of what you can do with it pretty regularly in the future.

The
[source code](http://code.google.com/p/revitpythonshell) is
available under the GNU General Public License v3 from a
[Subversion](http://subversion.tigris.org) repository on
[code.google.com](http://code.google.com).
It is documented, though there is currently little other stand-alone documentation.
One would probably want to look for the IExternalApplication entry point and go from there.
What the application does is:

- Add a button to the Revit ribbon to start the python shell.- Execute a python script entered in a text box, with output going to a separate window.

Daren describes one of the main scripts like this: it "will open up an interactive interpreter loop known as a REPL that will let you explore the Revit API.
And by explore, I mean type a statement, hit enter, see the results, carry on. It doesn't get easier as this!"

If you know
[RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html)
([for 2010](http://thebuildingcoder.typepad.com/blog/2009/10/rvtmgddbg-for-revit-2010.html))
and appreciate the possibilities it offers to manually dive into the Revit database via the Revit API by using hardcoded queries to explore elements and their properties, imagine enhanced functionality with complete freedom to interactively execute any arbitrary method on the elements, store the results, and continue processing those in any desired manner.

One main objective in releasing this code is the hope that it will make exploring the Revit API a lot easier and thus help push its bounds as far as possible.

#### Feedback and Participation

If you decide to take a look at the package, please also provide feedback to Daren, especially answers to the following two questions:

- Are the installation instructions adequate?- Have you encountered any problems?

Daren also points out that he would be happy to grant commit access to the source repository for any serious contributors.