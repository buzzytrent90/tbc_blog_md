---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.744980'
original_url: https://thebuildingcoder.typepad.com/blog/0324_process_start.html
post_number: '0324'
reading_time_minutes: 2
series: general
slug: process_start
source_file: 0324_process_start.htm
tags:
- csharp
- family
- revit-api
- walls
title: Using Process.Start to Open a Project or Family
word_count: 375
---

### Using Process.Start to Open a Project or Family

Here is a question that comes up from time to time on how to open and display a project or family file in the Revit user interface.
I mentioned a workaround in the answer to a
[comment by Nadim](http://thebuildingcoder.typepad.com/blog/2009/09/detail-lines.html?cid=6a00e553e1689788330120a6cf0e3e970b#comment-6a00e553e1689788330120a6cf0e3e970b
),
and now I think it is time to promote this little information nugget to a proper post to make it easier to find.

**Question:** I need to open and display an existing project or family in the Revit user interface.
I see that the NewFamilyDocument and NewProjectDocument methods create new documents but do not open them.
I tried to use the OpenDocumentFile method taking a document path name argument to open an existing project, but it does not display it in the user interface either, it only returns an in-memory version of the document.
I also implemented an event handler and verified that the OnDocumentOpen event is not triggered by it.
Is it possible to open and display a document in the Revit user interface, and if so, how, please?

**Answer:** Yes, correct, NewFamilyDocument, NewProjectDocument and OpenDocumentFile only create new or open existing documents for API manipulation.
They do not display them in the user interface, only open them in the background in-memory, without the Revit user interface even being aware of it, which is why the event is not triggered either.
There is currently no API method at all to make a document current in the Revit user interface.
There is a workaround, though, that several people have used successfully:
you can use Process.Start as described in the comments on
[debugging an add-in](http://thebuildingcoder.typepad.com/blog/2008/09/debugging-a-rev.html?cid=6a00e553e16897883301156fc1fed8970b#comment-6a00e553e16897883301156fc1fed8970b).
Here is a little console application demonstrating such a call:
```csharp
class Program
{
  static void Main( string[] args )
  {
    System.Diagnostics.Process.Start(
      "C:/tmp/wall.rvt" );
  }
}
```

This uses the existing instance of Revit if it is already running.
It also works fine to start up Revit if you replace the project file "C:/tmp/wall.rvt" by the Revit application executable, although in that case obviously no document is opened.