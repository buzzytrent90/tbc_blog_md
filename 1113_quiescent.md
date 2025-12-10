---
post_number: "1113"
title: "Determining the Quiescent State"
slug: "quiescent"
author: "Jeremy Tammik"
tags: ['family', 'revit-api', 'transactions', 'walls']
source_file: "1113_quiescent.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1113_quiescent.html"
---

Closed
Submitted Version/s: \* R2016 Copernicus RTS
Resolution: Fixed
From: Katsuaki Takamizawa (Contingent)
Sent: Friday, February 28, 2014 3:38 AM
To: Revit API
Subject: Finding if Revit is busy
Hi,
The developer is using the external event to implement a modeless dialog box.
Is there any way to find if Revit is in a state busy so that the request won't be executed right away? The developer wants to disable the modeless dialog during the busy state. (so the users clearly see when the dialog box is usable)
I think that maybe events like “begin/end the idling state” would be need to achieve this.
Any idea?
Thank you,
Katsu
From: Arnost Lobel
Sent: Saturday, March 01, 2014 10:25 AM
To: Partha P. Sarkar; Katsuaki Takamizawa (Contingent); Revit API
Subject: RE: Finding if Revit is busy
There is no begin-end Idling state. There is just "Idling". External events get executed at the same (literary) time - that means that during each internal idling event Revit executes all delegates of API Idling event and also all external events that are signaled at the same time.
The recommendation for using external dialog in modeless dialogs is this:
The dialog owns acquires an instance of ExternalEvent
The dialog's stay enabled until the end user does something there. It is assumes (and likely in most situations) that if the end user is during something in the modeless dialog he or she is unlikely working in Revit at the same time.
After the user starts an action in the dialog, the dialog signals the external event and wait for its handler to be called back. At that time the dialog's control should get disabled and stay disabled until the handler gets executed.
When Revit calls the handler back, the handler does what it needs to (depending on the action requested by the dialog user).
After the handler completes it API calls, the dialog's controls can be enabled again
Back to step #2
Arnošt
From: Partha P. Sarkar
Sent: Friday, February 28, 2014 4:25 AM
To: Katsuaki Takamizawa (Contingent); Revit API
Subject: RE: Finding if Revit is busy
Hi Katsu-san,
Did you check this –
Application.IsQuiescent -> This property can be used check if the current application is quiescent. If any document in the current application is in editing mode or an In-place family is being edited, it returns false. Otherwise, true will be returned.
Thanks,
Partha
From: Arnost Lobel
Sent: Saturday, March 01, 2014 10:25 AM
To: Partha P. Sarkar; Katsuaki Takamizawa (Contingent); Revit API
Subject: RE: Finding if Revit is busy
By the way, Application.IsQuiescent is useless in Revit. It does not do anything. If an application can legally call it, it will return false. And if it returns true it means the caller calls it illegally and may not call any other API anyway. IsQuiscent is not thread-safe either.
Arnošt
From: Katsuaki Takamizawa (Contingent)
Sent: Sunday, March 02, 2014 8:48 PM
To: Arnost Lobel; John (Zhenjun) Feng; Partha P. Sarkar; Revit API
Subject: RE: Finding if Revit is busy
Thank you all for the explanations. J
The developer says his modeless dialog has [Apply] like button which modifies Revit models when pressed. He wants to enable the button only if the modification may be executed right away. This is just how the developer wants to implement the user interface behavior.
If the following scenario makes sense (and unless it imposes any issue or side effect), I would like to file a wishlist for “begin/end the idling state” event.
1. Disable [Apply] button.
2. The application receives “the begin idling state” event.
3. Enable [Apply] button.
4. The user then presses [Apply] button. Raise() is called, the even handler is called and modification is done.
5. The application receives “the end idling state” event. Go back to the step 1.
Thank you,
Katsu
From: Arnost Lobel
Sent: Monday, March 03, 2014 10:04 PM
To: Katsuaki Takamizawa (Contingent); John (Zhenjun) Feng; Partha P. Sarkar; Revit API
Subject: RE: Finding if Revit is busy
Katsu,
did you also get my other email in which I tried explaining why using Idling is not so great an idea?
Remember - there is no Begin or End for Idling. There is only Idling. Besides, using Idling would negate the benefits of using External Events.
There is unfortunately no way in Revit API to achieve exactly what the user wants to do - that is having the dialog controls enabled only and only if Revit is ready to execute external events immediately. Choosing the best approach depends on what the dialog actually wants to do with Revit. In typical scenarios, end users go to an eternal modeless dialog only when they do not do anything else In Revit at that time. That is why using External events works very well for that kind of applications. The dialog's controls are enabled all the time except for the time between raising an external event and handling it.
Thanks
Arnošt
From: Katsuaki Takamizawa (Contingent)
Sent: 2014年3月5日 9:33
To: Arnost Lobel; John (Zhenjun) Feng; Partha P. Sarkar; Revit API
Subject: RE: Finding if Revit is busy
Hi Arnošt,
Yes, I have read your other email (thank you for the detailed information). I now see that the begin and end of the idling state do not exist – other external event handers and idling delegates may be executed.
I will communicate with the developer with this information. J
Thank you again,
Katsu
From: Joe Ye
Sent: Tuesday, March 04, 2014 8:44 PM
To: Katsuaki Takamizawa (Contingent); Arnost Lobel; John (Zhenjun) Feng; Partha P. Sarkar; Revit API
Subject: RE: Finding if Revit is busy
Hi,
I tried to remove the idling event registration in the end of this idling event handler to make the idling event run only once. (register the idling event when needed).
It looks I can remove the idling registration within this idling's event handler.
So this can avoid the idling handler from being called repeatedly, and make the Revit's performance good.
Cheers,
From: Arnost Lobel
Subject: RE: Finding if Revit is busy
Joe
Yes, Idling handler can be unregistered from the handler, which is true for other events in Revit too.
However, I do not understand how using the Idling even can help at all. What is the benefit over using an External event. They both get raised at the same time (assuming the external event is signaled). I can only repeat from I stated before: there is currently no way for external applications to know when their UI should be disable and when it can be disabled based on current state of Revit. It may be possible to add some very specialized features to docking dialogs, but we would have to add a request for it.
Thank you
Arnošt
From: Miroslav Schonauer
Date: Wednesday, March 5, 2014 08:44
To: Joe Ye , "Katsuaki Takamizawa (Contingent)" , Arnost Lobel , "John (Zhenjun) Feng" , "Partha P. Sarkar" , Revit API
Subject: RE: Finding if Revit is busy
I vaguely remember once playing with these scenarios and found basically what Arnost clarified: there is NO way to reliably get “Idling Started” and “Idling Ended” events.
Joe – your scenario below will work to get only ONE Idling event that is “quasi-Idling-started”, but then you can't do anything after that single catch – the problem is that you cannot do the reverse, ie to \*start\* Idling again from Idling event, simply because it's not running and in order to know when to start you are back to square one (been there ;-)
My 2p,
Miro
-->

### Determining the Quiescent State

I am still going through my emails after returning from the desert.
Here is a summary of an interesting discussion between Katsuaki Takamizawa, Arnošt Löbel, Partha Sarkar, Joe Ye and Miroslav Schonauer on how to determine the quiescent state of Revit for accessing the API from a modeless context, and some recommendations to simply avoid that entirely, since other more efficient and stream-lined interaction models end up being easier to implement and more reliable to use.

**Question:** I am using an external event to interact with the Revit API from a modeless dialogue.

Is there any way to find if Revit is in a busy state so that the request won't be executed right away?
I would like to disable the modeless dialogue during such a busy state, so the user can clearly see when the dialogue box is usable.

Are there any events like 'begin/end the idling state' that I can use to achieve this?

**Answer:** There is no begin-end Idling state.
There is just the Idling event itself.
External events are executed at the same (literary) time – that means that during each internal idling event Revit executes all delegates of API Idling event and also all external events that are signalled at the same time.

The recommendation for using external events in modeless dialogues is this:

1. The dialogue acquires an instance of ExternalEvent
2. It remains enabled until the end user interacts with it, assuming that while the end user is doing something in the modeless dialogue he or she is unlikely working in Revit at the same time.
3. After the user starts an action in the dialogue, it raises the external event and waits for its handler to be called back. At that time the dialogue's control should be disabled and remain so until the handler has been executed.
4. When Revit calls the handler back, the handler does what it needs to, depending on the action requested by the dialogue user.
5. After the handler completes its API calls, its controls are enabled again
6. Back to step #2

This is demonstrated by the
[ModelessDialog ModelessForm\_ExternalEvent SDK sample](http://thebuildingcoder.typepad.com/blog/2012/04/idling-enhancements-and-external-events.html).

Although the Application class provides an IsQuiescent property that one would assume could be used to check if the current application is quiescent, it is unfortunately useless in Revit. It does not do anything. If an application can legally call it, it will return false. If it returns true, it means the caller calls it illegally and may not make any other API calls anyway. IsQuiescent is not thread-safe either.

We already presented a more detailed discussion on the uselessness of the
[Application IsQuiescent property](http://thebuildingcoder.typepad.com/blog/2013/01/what-i-do-wall-layers-and-open-transactions.html#4) in
January 2013.

**Response:** Thank you for the explanations.

My modeless dialogue displays an 'Apply' button which modifies the Revit model when pressed.
I would like to enable the button only if the modification may be executed right away.

If the following scenario makes sense (and unless it imposes any issue or side effect), I would like to file a wish list item for a 'begin/end the idling state' event.

1. Disable the 'Apply' button.
2. The application receives the 'begin idling state' event.
3. Enable the 'Apply' button.
4. The user then presses the 'Apply' button. The external event Raise method is called, the event handler is called and Revit model modification performed.
5. The application receives the 'end idling state' event. Go back to the step 1.

**Answer:** As said, using Idling is probably not so great an idea for this.

There is no Begin or End event for Idling.
There is only the Idling event itself.
Besides, using Idling would negate the benefits of using External Events.

There is unfortunately no way in Revit API to achieve exactly what you want to do – that is, having the dialogue controls enabled only and only if Revit is ready to execute external events immediately.

Choosing the best approach depends on what the dialogue actually wants to do with Revit.

In typical scenarios, end users interact with an external modeless dialogue only when they do not do anything else In Revit at that time. That is why using External events works very well for that kind of applications. The dialogue's controls are enabled all the time except for the time between raising an external event and handling it.

You can subscribe to the Idling event and unsubscribe immediately in the event handler the first time it is called. This almost provides the functionality of a 'begin Idling' event. It also avoids the idling handler being called repeatedly, to avoid degrading system performance.

However, this still does not enable any reliable way to get 'Idling Started' and 'Idling Ended' events.

The scenario below will work to get only ***one*** Idling event that is 'quasi-Idling-started', but then you can't do anything further after that single catch – the problem is that you cannot do the reverse, i.e. cause Idling to ***restart*** again from the Idling event handler, simply because it's not running.

It therefore does not help at all in this situation, which is better addressed by using an external event.
Both the Idling and external events are raised at the same time (assuming the external event has been signalled).

Summary: there is currently no way for external applications to know when their UI should be disabled and when it can be enabled based on the current state of Revit, ***and*** simple and effective alternative approaches are available to eliminate the need for this functionality.

Many thanks to everybody involved for this illuminating discussion and clarification!