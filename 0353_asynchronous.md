---
post_number: "0353"
title: "Asynchronous API Calls and Idling"
slug: "asynchronous"
author: "Jeremy Tammik"
tags: ['levels', 'parameters', 'revit-api', 'transactions', 'views', 'walls']
source_file: "0353_asynchronous.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0353_asynchronous.html"
---

### Asynchronous API Calls and Idling

I discussed the
[Idling event](http://thebuildingcoder.typepad.com/blog/2010/04/idling-event.html) yesterday,
and since received a note from the venerable Revit API guru Guy Robinson who finds this new event just as momentous and exciting as I do and therefore promoted it to **the**
[Revit 2011 API standout feature](http://redbolts.com/blog/post/2010/04/19/Revit2011-API-standout-feature-e28093-being-Idle.aspx).
If Guy likes it, it's the real thing, for sure.

Since this once again dives full into the topic of
[controlling Revit from an external application](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html) or a
[modeless dialogue](http://thebuildingcoder.typepad.com/blog/2009/12/modeless-dialogues-in-revit.html),
here is a very apt comprehensive summary of the dangers and possibilities of working asynchronously with the Revit API by Arnošt Löbel, Senior Software Engineer of the Revit development team:

#### The danger of invoking Revit API asynchronously (and the correct way of doing so)

In the past couple of months I've got a number of requests for explaining somehow "odd" and "strange" behaviour of the API. Sometimes it was a transaction that refused to start for no apparent reason, another time it was an API call that failed, rather surprisingly, for it previously worked just fine under "apparently" the same conditions (except for the exact time.) On a few occasions Revit would record errors and/or warnings into the journal. One or two reported Revit crashing. Naturally, some of the problems led us to finding issues in Revit, most of which I am glad to say we have fixed. A bunch of the reported problems were of a different nature though – they were not caused by bugs; they were caused by not using the API "properly". I am not sure about how many improper ways of using the API exist, but I know about one that is definitely quite dangerous.
In this note, I am going to describe that one case and explain why it should be avoided.

The fact: Revit does not expect external applications calling the API from other than the main Revit thread. Although it is possible to access the API from other threads, and sometimes such calls may even succeed, it is not a recommended technique and the outcomes of such calls can be virtually unpredictable and often fatal (to Revit :-)).

To give you an example of what I mean, the following is one of the unsupported API access workflows:

1. Revit calls an external command by invoking the command's Execute method.- The command collects data from Revit API, initiates a modeless dialog, and returns control back to Revit.- The modeless dialog waits for other data (could be the user's input), and then calls the API.

The obvious reason for the above scenario is not holding down Revit while the external application is performing the task. Though this workflow is thoughtful and actually correct, it is not allowed in Revit because Revit does not have a multi-thread-ready API. Revit uses multi-threading internally to speed up certain processes, but Revit does not expect external applications to be executed outside of predefined workflows. Revit does safeguard the API in various ways, but it only has the guards the main entry points, not at each and every API method. When an external application bypasses the entry points (like in the case outlined above), a lot of things that should happen will not happen (or vice-versa):

- Transactions will not be protected properly.- Transaction mode and Regeneration mode cannot be set accordingly.- Revit will not be able to tell what is the active application's ID.- Revit cannot tell what depth-level the API is operating on, therefore events may behave unexpectedly.

Most of the above would either cause exceptions that would not happen otherwise, or miss on exceptions that should happen. Both cases could be equally dangerous.

When I mentioned that Revit expects external applications to use one of the predefined workflows, I meant:

- The OnStartup and OnShutdown method of an external application.- The Execute method of an external command.- Execution of an event handler.- Execution of a VSTA macro.- Execution of methods of an instance of the IUpdater interface (such as the Execute method).- Executions of method of a call-back Interface (such as ITransactionFinalizer::OnCommitted).

When Revit calls these methods in external applications it expects that whatever the method would do, it will do it while the execution was in that method. Once the method returns back to Revit, the calls is considered completed and the API closed for external calls (though it is not technically closed).

Now, when an external application steps outside of the predefined workflows, one of many things can happen:

- Nothing bad at all – the call succeeds and all is fine (this is really rare ;-)).- Calls appear all right and return successfully, though the gathered data are out of date or out of sync.- Calls fail, exceptions are thrown, Revit stops what it was doing at that moment (and this is actually a good scenario ;-)).- Revit crashes, document must be closed.- Revit model is corrupted by two independent processes executing at the same time trying to modify the same or related data. In a better scenario, Revit crashes before the corrupted model is saved. In the worst scenario, the data is saved, which makes the document unrecoverable later.

Consider for a moment one of the mildest scenarios when the asynchronous call (AC for short) "just" reads data from the model:

1. AC calls to get one parameter of a wall.- Another application waiting for the Idling event was called and changed the type of the wall.- AC calls to get another parameter of the wall.- Third application's updater reacts to the change of the type of the wall, and makes the wall deeper.- AC calls to get another parameter of the wall.

Though those three AC calls might very well be next to each other in the client's code, they were technically executed at different times and the data acquired by each of the methods reflect the state of the model at those different times. Therefore, some or all of the gathered data might be wrong.

Naturally, I will expect some of you may ask if there is another way at all for using multiple threads (and/or modeless dialogs) in external applications and still be able to interact with Revit. The answer is Yes, there is. Revit does not mind if an external application uses more threads. Revit only requires that the application calls the API from the standard entry points only, such as commands and events, and from the thread in which the call was made. In scenarios that have been described to me so far, a quite simple workaround was possible. In most cases it meant utilizing the Idling event. I'll describe it in steps:

1. Application registers itself and its commands.- On one command, it kicks off a working thread (or a modeless dialog) and leaves it there working (or waiting).- If the kick-off succeeded, the command registers a handler for the Idling event.- The command than returns, Revit continues running.- Whenever possible and appropriate, Revit will raise the Idling event.- The application's handler gets the call.- Now, it all depends on how the application communicates with its working thread (or the modeless dialog):
               1. It could be that the thread periodically feeds data back to the application, so when the event is raised, the data is either there already or not. The event handlers then use the data and calls the API as needed.- Or it could be that the application queries the work thread somehow at the time of the event. If the thread is ready and waiting, the application gets data from it and uses it as it sees fit.- When the work thread finishes, it signals the application, so when the next Idling event is raised (or another event, such as DocumentClosed), the application can unregister from it.

Of course, this workflow is not quite as good as if the API was perfectly multi-threaded, but it is as good a workaround as it gets. Like I said, most if not all the scenarios I heard about so far could be accomplished this way.

Many thanks to Arnošt for this comprehensive overview!