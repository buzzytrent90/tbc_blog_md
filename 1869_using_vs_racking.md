---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:16.929060'
original_url: https://thebuildingcoder.typepad.com/blog/1869_using_vs_racking.html
post_number: '1869'
reading_time_minutes: 5
series: general
slug: using_vs_racking
source_file: 1869_using_vs_racking.md
tags:
- csharp
- family
- revit-api
- sheets
- transactions
- views
- windows
title: Using Vs Racking
word_count: 951
---

### Avoid Brain Racking by Using Using
You should always keep things simple (I think).
The opposite can lead to racking your brain.
In one case at least, that can easily be avoided by using a `using` statement:
- [Avoid brain racking by using `using`](#2)
- [On the VS operation unspecified error](#3)
- [Native sons](#4)
#### Avoid Brain Racking by Using Using
Here is a quick note on the importance
of [using `using` to automagically dispose and roll back](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html) transactions
and transaction groups, and the grief and brain racking that can easily ensue from failing to do so:
\*\*Question:\*\* For certain users, my add-in is hard crashing Revit to the desktop.
No exceptions, just everything closes.
I’ve got the journal files and I wanted to get some help seeing if there is anything to learn from them.
\*\*Answer:\*\* Sorry to hear about the hard crashes. And on certain users' systems, to boot.
Is the problem reproducible?
Does it only happen on certain specific machines?
Do you have any idea at all where the problem might be?
Does it send error reports to Autodesk?
What context can cause it?
Do you think it is due to your add-in, or Revit, or both?
\*\*Response:\*\* Thanks for the follow up.
I was able to get a hold of a customer’s computer that was seeing the error and track it down.
I had a `TransactionGroup` that was started but never completed – I did not call `Assimilate` or `Rollback`.
Adding the missing rollback fixed the error.
I did a review of all my other code and confirmed I closed all my other transaction groups and transactions after I opened them.
Relying on the destructor to call rollback obviously is a bad idea and was not intentional.
The relevant lines from the journal file look like this:

```
' 7:< REGEN_DOC_CONTEXT_INFO: Changing wrong atom in regeneration
' 7:< REGEN_DOC_CONTEXT_INFO: Document is changed in regeneration while it is not supposed to
```

The transaction wasn’t closed and Revit tried to regenerate.
\*\*Answer:\*\* Congratulations on solving the issue!
Thank you very much for your research and useful explanation.
Were your transaction groups originally encapsulated in a `using` statement?
The Building Coder recommends doing so and claims that will obviate the need to call `Assimilate` or `Rollback`:
[Using Using Automagically Disposes and Rolls Back](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html).
I would be a rather shocked if that would turn out to be false.
So, I hope your transaction group causing the problem was not enclosed in a `using` statement.
If that is the case, I would strongly recommend ALWAYS enclosing transactions and transaction groups in `using` statements.
That would (I still hope) reliably avoid this problem forever.
Can't wait to hear your response on this...
\*\*Response:\*\* Brilliant!
No, I was not using `using` in my code.
Makes perfect sense to do so to prevent errors like mine.
I’m going to work on adding that to my code. There are 170 places I use a transaction, so it’s going to be a little work.
Your blog is a great resource. I use it quite regularly when trying to figure things out.
I don’t have any interesting code snippet to share.
It’s basically a `TransactionGroup` with an `Assimilate` call in an `IF` statement and a forgotten `Rollback` in the `ELSE`.
![Brain rack](img/brain-rack.png "Brain rack")
#### On the VS Operation Unspecified Error
A small note on the Visual Studio 'Operation could not be Completed, Unspecified Error':
You may have notices building Revit add-ins within Visual Studio getting strange build failures and a message along the lines of \*21>Error: The operation could not be completed. Unspecified error\*.
This seems to be an issue where the Visual Studio IDE gets itself into a strange state and the project in question has not been loaded properly for some reason.
I found it related to switching project configurations before the build or even switching solutions.
It could also have to do with do a merge in git outside of the IDE that causes projects to change and reload.
Anyway, without being clear the exact steps, when I do see the error, I found one of the following clears things up:
- Close and reopen the solution in question.
- Fully exit and restart Visual Studio.
Could it have something to do with switching header files, or switching from debug to release?
It may, but I have seen it just simple closing one solution and loading another.
I also got it in a standalone C# solution with some C++ projects.
It usually appears while trying to compile the C++ projects, which are a interop dependency for the C# projects – one theory is that somehow the C# binary locks the C++ one and doesn’t let msbuild query it for… god knows what, since interop is done at runtime afaik (never seen this error on rebuild, only build ).
And yes, only VS restart cures it; fortunately, we rarely touch the C++ projects, so I just ignored this error so far.
Jeremy adds: I never touch C++ code, only C# Revit add-ins, and I see this message now and then as well.
I just restart VS or even reboot Windows (under Parallels on Mac) and all is well.
#### Native Sons
John Candela pays tribute to Native Americans for Monday,
[Indigenous People's Day](https://en.wikipedia.org/wiki/Indigenous_Peoples%27_Day), with
his track [Native Sons](https://soundcloud.com/jdcandela/native-sons):

[JC](https://soundcloud.com/jdcandela "JC") · [Native Sons](https://soundcloud.com/jdcandela/native-sons "Native Sons")