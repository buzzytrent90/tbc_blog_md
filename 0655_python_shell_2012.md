---
post_number: "0655"
title: "Python Shell for Revit 2012 and Vasari 2.1"
slug: "python_shell_2012"
author: "Jeremy Tammik"
tags: ['geometry', 'python', 'references', 'revit-api', 'walls']
source_file: "0655_python_shell_2012.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0655_python_shell_2012.html"
---

### Python Shell for Revit 2012 and Vasari 2.1

I was so happy to tell you that work on Daren Thomas'
[RevitPythonShell](http://code.google.com/p/revitpythonshell) was
recently intensified and endorsed by Zach Kron of
[buildz](http://buildz.blogspot.com) to
support
[running in Vasari](http://thebuildingcoder.typepad.com/blog/2011/07/python-shell-in-revit-and-vasari.html).
Well, the project has been taken a step further:
[RevitPythonShell now runs in both Revit 2012 and Vasari 2.1](http://buildz.blogspot.com/2011/09/python-scripting-in-vasari-21.html).

Zach's post includes simple yet detailed instructions for installing RevitPythonShell in Vasari in 5 minutes, and similar steps also work for Revit 2012.

Here are the modified steps I performed to use it on Revit instead of Vasari:

- Install Revit 2012, any flavour (well, that may take a bit longer; let's assume that has already been done).- Install IronPython: <http://ironpython.net>.- Install RevitPythonShell for Revit 2012: <http://code.google.com/p/revitpythonshell/downloads/detail?name=Setup_RevitPythonShell_2012.msi>.- Open Revit > Add-Ins > RevitPythonShell > Configure...- In the Search Paths tab, add the Lib folder of your IronPython installation to enable the autocomplete feature. In my case, this is "C:\Program Files\IronPython 2.6 for .NET 4.0\Lib".

I actually have IronPython 2.6 installed, not the latest version 2.7.
Similarly, I am still running an old version of Python itself, 2.6.2, from Apr 14 2009, no less.
Still, all works well, as far as I can tell so far:

![RevitPythonShell console in Revit 2012](img/revitpythonshell_console_2012.png)

It shows a simple sample script that demonstrates that I have immediate interactive access to the Revit API functionality.
In this trivial case, I just use it to display the document path and read the width of the first wall type in the document WallTypes collection:

```
>>>clr.AddReference('RevitAPI')

>>>from Autodesk.Revit.DB import *

>>>doc = __revit__.ActiveUIDocument.Document

>>>doc.PathName
'C:\\Program Files\\Autodesk\\Revit Architecture 2012
\\Program\\Samples\\rac_basic_sample_project.rvt'

>>>iter = doc.WallTypes.ForwardIterator()

>>>iter.MoveNext()
True

>>>wt = iter.Current

>>>wt
<Autodesk.Revit.DB.WallType object at
0x000000000000002B [Autodesk.Revit.DB.WallType]>

>>>wt.Width
1.15625
```

Very many thanks to Daren Thomas, Aki Hogge and Zach Kron for all their effort in creating and improving this powerful tool!

Chatting with Zach, the issue of using this and increasing exposure and access to the Revit API came up.
We hope this is going to help bring new coders into the Revit API.
Obviously, that is easier if you already know Python.
Zach added:

My optimism comes from the big push to Python that is going on in the
[Rhino](http://www.rhino3d.com) community,
since they moved from rhinoscripting.
They have really wonderful introductory documentation, the
[Python for Rhino 101 Primer](http://python.rhino3d.com/content/130-RhinoPython-primer),
that actually helped me start to understand how to use the Revit API.

Rhino is the platform of choice in many architecture graduate schools in the US, but I always hear students sigh and say 'I GUESS I should really learn Revit so I can get a job'.

Sections from the primer include:

1. What is it all about?- Python Essentials- Script anatomy- Operators and functions- Conditional execution- Tuples, Lists and Dictionaries- Classes- Geometry- Notes

You might find this helpful to understand the Revit API and especially the use of the RevitPythonShell as well.

Talking about Rhino and Revit, here is a series of very interesting buildz posts on
[Revit interoperating with Rhino](http://buildz.blogspot.com/2011/01/joe-k-working-with-rhino-and-revit-part.html).