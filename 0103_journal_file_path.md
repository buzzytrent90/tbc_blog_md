---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: documentation
optimization_date: '2025-12-11T11:44:13.367069'
original_url: https://thebuildingcoder.typepad.com/blog/0103_journal_file_path.html
post_number: '0103'
reading_time_minutes: 2
series: general
slug: journal_file_path
source_file: 0103_journal_file_path.htm
tags:
- csharp
- references
- revit-api
- views
title: Getting the Journal File Path
word_count: 481
---

### Getting the Journal File Path

Here is an edited excerpt of an interesting and useful dialogue between Greg Wesner and Harry Mattison, both of Autodesk:

I am building an application that needs to access the journal file when Revit saves a document.

The path to the journal file can be found at Application.RecordingJournalFilename. But you only have access to this Application object when a command is executed and you get passed an ExternalCommandData reference in your command method.

If you just want to create an event handler for DocumentSavedEventHandler in an ExternalApplication, the only class you have access to is a ControlledApplication class. This class provides no method to determine the path to the journal file.

So I'm looking for any way to get the path to the journal file without having to make the user execute a command. Any ideas?

Are you planning to write some additional text into the journal file? I don't think that will work because the Revit process has the journal file in use while Revit is running.

In any case, journals are written to the "Journals" folder alongside the "Program" folder where Revit.exe is located. Using that rule you could get the path the journal file like this:

```csharp
Process p = Process.GetCurrentProcess();
string exePath = p.MainModule.FileName;
string journalPath = exePath.Replace(
"Program\\Revit.exe", "Journals" );
```

Yeah, I thought about that. To get the name of the file, I guess I would just find the file with the latest time stamp.

I'm only reading data from it. I'm logging certain warnings or errors that users hit.
I discovered that I have to be very specific when opening the journal file and only open it with read-only permission and the access set to share.
Otherwise, the open call will fail.
So I've already passed that hurdle.
But I did all my initial work within a command.
I had not anticipated that the application object in the OnStartup() method would be a different object.

Unfortunately, on Vista, if you launch without Admin privileges, or possibly Power User, Revit cannot write the file to the Journals folder under Program Files.
Therefore it uses the User Data folder instead.
In that case, you have to use different logic to find it, and determining its location is more complex.

It would be nice to be able to get the Application object (the same one you can access from an ExternalCommandData) outside of a command.

In Revit 2010 the problem is simpler, because you can get the Application from any Document, and a Document is provided by several common events like DocumentOpened, ViewActivating, etc.

Right now I'm working in 2009. But that sounds like what I need. The document gets passed in to the DocumentSavedEventHandler. I was looking for a way to get the Application from that and couldn't find it.