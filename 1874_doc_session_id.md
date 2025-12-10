---
post_number: "1874"
title: "Doc Session Id"
slug: "doc_session_id"
author: "Jeremy Tammik"
tags: ['elements', 'python', 'references', 'revit-api', 'rooms', 'sheets', 'views']
source_file: "1874_doc_session_id.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1874_doc_session_id.html"
---

### Doc Session Id, API Context and External Events
Some new topics, and, as always, some recurring:
- [Document session id](#2)
- [Valid Revit API context and external events](#3)
- [Determining RVT file version for DA4R workitem](#4)
- [Revit API via HTTP](#5)
- [Parable of the polygons](#6)
#### Document Session Id
An interesting question that I have dabbled with in the past came up in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [document session id](https://forums.autodesk.com/t5/revit-api-forum/document-session-id/m-p/9844775).
My dabbling has to do with a permanent document identifier;
Tobias, asking the question, and Richard, providing a more satisfying and useful answer for this specific case, highlight the use of `Document.GetHashCode` to identify document instances in the current session only:
\*\*Question:\*\* Is there something like a session id for open documents?
I would like to uniquely identify a Autodesk.Revit.DB.Document object, for example in the ViewActivated Event. This is needed while more than one document can be open at the same time in Revit. Because I hold some stuff in memory, I need to know to which document it belongs.
The DocumentPath as Id is not an option, because newly created documents have a zero string (could be occur more than one as well in a session)
\*\*Answer 1:\*\* Every Revit project includes a singleton `ProjectInformation` database element.
Every database element has a unique identifier.
So, theoretically, you could use the `ProjectInformation` unique identifier to uniquely identify a project.
Unfortunately, if I create a project A, and then copy it for reuse in project B, the two ProjectInformation elements will have the same unique id, so this solution only works if you can guarantee that no such copy will ever be created,
cf. [project identifier](https://thebuildingcoder.typepad.com/blog/2017/12/project-identifier-and-fuzzy-comparison.html#2).
Here is some further analysis of the task
to [identify a project](https://the3dwebcoder.typepad.com/blog/2015/07/implementing-mongo-database-relationships.html#2).
To address this, I suggested a different and safer solution using an extensible storage schema and a named GUID storage for project identification instead
using [named guid storage for project identification](https://thebuildingcoder.typepad.com/blog/2016/04/named-guid-storage-for-project-identification.html).
\*\*Response:\*\* Indeed, I searched before without a satisfying result.
I came to the same conclusion as your solutions.
I was hoping there was something else :-)
Possibly use the combination of unique identifier and the file path, but this is not very elegant.
I think I will end up using the extensible storage solution.
Maybe a second schema, not the same as my plugin uses for storing its data.
\*\*Answer 2:\*\* Some documents don't contain a path (yet to be saved ones).
[Document.GetHashCode](https://www.revitapidocs.com/2020/006a71c2-4393-e036-9987-14467342a7d3.htm) has
served me well over the years:
> The hash code is the same for document instances that represent the same document currently opened in the Revit session.
The hash code is generated when a Revit file is opened or created in session.
If the same Revit file is opened later (in the same session or a different session), the hash code will not be the same.
Mainly, I use this in a static variable to see if the current document has switched via the `ViewActivatedEvent`.
\*\*Response:\*\* Ah, very nice. Thank you for that.
#### Valid Revit API Context and External Events
Developers keep attempting to access the Revit API from outside of Revit itself, e.g., in the questions
on [how to open different version Revit files](https://forums.autodesk.com/t5/revit-api-forum/how-to-open-different-version-revit-files/m-p/9861186)
and [how to raise an external event from a WPF app](https://stackoverflow.com/questions/64683308/how-do-i-raise-an-external-event-in-the-revit-api-from-a-wpf-app):
The Revit API cannot be used outside a valid Revit API context:
- [Use of the Revit API Requires a Valid Context](http://thebuildingcoder.typepad.com/blog/2015/12/external-event-and-10-year-forum-anniversary.html#2)
- [Revit API Context Summary](http://thebuildingcoder.typepad.com/blog/2015/08/revit-api-context-and-form-creation-errors.html#2)
You can work around this limitation, though:
the Revit API enables you to define an external event that can be triggered from outside a valid Revit API context.
The implementation and definition of the external event happens within a Revit add-in and requires a valid Revit API context.
Once defined, though, the external event can be raised from outside.
The event handler, again, must reside and execute within.
#### Determining RVT File Version for DA4R Workitem
That said, some operations can be performed on an RVT BIM from outside Revit, without use of the Revit API or such a context.
Another question that came up repeatedly in the past few weeks is how to detect the Revit file version before passing it to a DA4R or Forge Design Automation for Revit workflow. In DA4R, you can specify what version of the Revit engine to launch. Picking the appropriate one for the given RVT file avoids the time-consuming upgrade process:
\*\*Question:\*\* When I specify `Autodesk.Revit+2021` for the design automation AppBundle and Activity engine, the Revit 2020 version RVT file that I pass in is upgraded to 2021.
I would like to avoid the RVT file version upgrading.
Is it necessary to prepare a separate AppBundle and Activity for each Revit version, detect the Revit file version up front, and select which Activity to use?
If so, how can I determine the Revit file version?
I saw the article on how
to [check the version of a Revit file hosted on the cloud](https://forge.autodesk.com/blog/check-version-revit-file-hosted-cloud).
I would like to know more detailed info for the accurate detection.
It says that the RVT file contains the file `BasicFileInfo` with the following data snippets:
- 0-14 bytes = unknown
- 15-18 bytes = the length(int32) of the subsequent field value
- 19-(19+length\*2) bytes = the Revit file version (UTF16LE)
Is it true?
Can I use this data to determine the Revit file version before passing it to DA?
![BasicFileInfo for DA4R](img/basicfileinfo_for_da4r.png "BasicFileInfo for DA4R")
\*\*Answer:\*\* This question is answered in full in the following blog posts:
- [Check the version of a Revit file hosted on the cloud](https://forge.autodesk.com/blog/check-version-revit-file-hosted-cloud)
- [Basic File Info and RVT File Version](https://thebuildingcoder.typepad.com/blog/2013/01/basic-file-info-and-rvt-file-version.html)
- [Automatically Open Correct RVT File Version](https://thebuildingcoder.typepad.com/blog/2020/05/automatically-open-correct-rvt-file-version.html)
Furthermore, this recent discussion on StackOverflow addresses your exact requirements more precisely still:
- [Reliably Determine Revit Version of BIM 360 Project](https://stackoverflow.com/questions/63135095/reliably-determine-revit-version-of-bim-360-project)
\*\*Response:\*\* I found the implementation of Python in the blog article that you pointed out really helpful.
Thank you so much!
#### Revit API via HTTP
The implementation and use of external events can be perfected and simplified, as proven by Igor Serdyukov, aka Игорь Сердюков or WhiteSharq, and Kennan Chen:
- [External Communication and Async Await Event](https://thebuildingcoder.typepad.com/blog/2020/02/external-communication-and-async-await-event-wrapper.html)
- [Revit.Async](https://thebuildingcoder.typepad.com/blog/2020/03/another-async-await-rex-and-structural-analysis-sdk.html#3)
Gregor Vilkner of [Microdesk](https://www.microdesk.com) makes use of that in his exciting class at
the [AEC Tech Hackathon 2020](https://www.aectech.us) in October:
- [Jailbreak Revit: GraphQL and ServiceBus](https://youtu.be/7LnbP4n4RYM)

Says Greg: Got a shout out for you... 19:30 min mark...
#### Parable of the Polygons
For something completely different, here is a nice little demonstration of the subtle influence various individual preferences and prejudice can have on collective behaviour; check out
the [Parable of the Polygons](https://ncase.me/polygons), a segregation and diversity simulation that clearly proves certain points:
1. Small individual bias → Large collective bias.

When someone says a culture is shapist, they're not saying the individuals in it are shapist. They're not attacking you personally.
2. The past haunts the present.

Your bedroom floor doesn't stop being dirty just coz you stopped dropping food all over the carpet. Creating equality is like staying clean: it takes work. And it's always a work in progress.
3. Demand diversity near you.

If small biases created the mess we're in, small anti-biases might fix it. Look around you. Your friends, your colleagues, that conference you're attending. If you're all triangles, you're missing out on some amazing squares in your life – that's unfair to everyone. Reach out, beyond your immediate neighbours.