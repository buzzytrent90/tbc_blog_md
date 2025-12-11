---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.483718'
original_url: https://thebuildingcoder.typepad.com/blog/1191_activate_famil_symbol.html
post_number: '1191'
reading_time_minutes: 5
series: elements
slug: activate_famil_symbol
source_file: 1191_activate_famil_symbol.htm
tags:
- csharp
- elements
- family
- geometry
- parameters
- revit-api
- selection
- transactions
- views
- walls
- windows
title: Activate Your Family Symbol Before Using It
word_count: 1094
---

### Activate Your Family Symbol Before Using It

Here is an interesting case with a simple solution that was rather hard to discover.

In summary, you need to ensure that all family symbols are activated before making use of them.

**Question:** I'm having a strange problem when replacing curtain walls by windows in Revit 2014.

When I run my add-in command, all the curtain walls are successfully replaced by the window family instances.

All except the first also cut the hosting wall in the proper manner.

The first one, however, does not.

When I manually use the Cut tool, the first window will also behave properly and cut the wall like the others do.

I don't understand why the first window doesn't cut the wall and all the following ones do.

Can you please tell me what I am doing wrong?

**Answer:** The first thing I notice looking at your add-in solution is that you have not
[set the 'Copy Local' flag to false](http://thebuildingcoder.typepad.com/blog/2011/08/set-copy-local-to-false.html) on
your Revit API assemblies.

Please always do that.
Otherwise, your debugger may not work properly.
It also causes your zip file to be 16 MB in size instead of 6 KB.

I tested the behaviour in both Revit 2014 and Revit 2015 and can reproduce exactly what you say.

**Response:** Thank you for your reply.
I really need this to work, so do you think it is possible to implement the following manual workaround in the API?

In the user interface, I can use this workaround:

1. Launch the 'Cut Geometry' command.
2. Select the wall.
3. Select the 'hidden' window.

This will cut the window opening in the wall the way I want it to.

Can this be achieved programmatically?
For instance, is there a way to call the 'Cut geometry' command in the API?
I read something about SolidSolidCutUtils, but couldn't find any way to use that in this case.

**Answer:** I don't know off-hand.
To determine whether the cut geometry command can be driven programmatically, you can check your journal file, retrieve the command id, and try it out as described in this discussion on how to
[programmatically launch an add-in command](http://thebuildingcoder.typepad.com/blog/2013/10/programmatic-custom-add-in-external-command-launch.html).

More information on using the PostCommand API is provided by the overview of
[What's New in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) and the
[Revit 2015 API news and DevDays Online recording](http://thebuildingcoder.typepad.com/blog/2014/04/revit-2015-api-news-devdays-online-recording.html).

**Response:** I tried the PostCommand method.
Does it provide any way to pass in parameters to the command being launched?

I found this:

```csharp
  app.PostCommand(
    RevitCommandId.LookupPostableCommandId(
      PostableCommand.CutGeometry ) );
```

It does indeed launch the CutGeometry command.
At that point, Revit waits for user input.
Specifically, it waits for the user to select two elements.

I tried to specify them programmatically through the current selection set like this:

```csharp
  Selection sel = app.ActiveUIDocument.Selection;
  sel.Elements.Add(element to be cut);
  sel.Elements.Add(element that makes the cut);
```

Revit doesn't react to that, however.

I also tried to use some kind of constructor option, and that failed as well:

```csharp
  PostableCommand.CutGeometry(
    host, cutting element);
```

So my next question: when using PostCommand, how can I trigger the pick element option or, alternatively, how can I tell the cut geometry tool which elements need to be cut?

**Answer:** Have you read the Revit API documentation and the blog posts describing
[PostCommand](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.3)?

They should actually be clear about this.

No, sorry, you cannot pass in any arguments to it.

It really does launch the standard interactive user interface command and prompts the user for input in the normal manner.

I'm sorry if that is not what you need.

**Response:** Too bad PostCommand cannot be parameter driven.
So in all, I still cannot progress.
The ugly workaround I am currently using is to copy the first window, delete it and place it again.
So, I am really looking forward to a better solution.
I cannot see any errors in my code.

**Answer:** I am glad to hear that you have a workaround, at least.
Congratulations on that.

While further exploring this issue, we discovered the following:

This is an interesting issue – I can reproduce it as described in 2015.
No matter the order of curtain walls deleted and replacement windows added, the first replacement window does not intersect/cut the wall.
I tested this vs. the provided code by using Insert(0) instead of Add() to build the list of curtain walls to remove.

If I delete the first wall in the UI first, the problem transitions to the next one when the code is run.

I experimented with separating the delete and create calls by a call to regenerate, and also into separate transactions, with no improvement.

Finally, we discovered the following explanation for this behaviour:

The reason is that a FamilySymbol does not regenerate when it is created.
The first time it regenerates is in commit transaction.
Therefore, when the first window is placed in the wall, the symbol of the window does not regenerate immediately, so some data is not prepared and no opening is generated.
A regeneration is needed after creating a family instance.

The way things stand right now in the Revit API, you need to activate a family symbol before using it.
This is the single missing step to get the code working.
Basically, the extra step is simply:

```csharp
  if( !symbol.IsActive )
  { symbol.Activate(); doc.Regenerate(); }
```

These statements need to be executed before calling NewFamilyInstance.

This is the proper technique for the developer to get things working in shipping releases.

It seems like using NewFamilyInstance with an un-activated symbol is bad.
Perhaps it will be prevented in future releases, since the results cause the model to be in an indeterminate state.

Here is a slightly larger code snippet demonstrating a simple real world use of this statement:

```csharp
  if( doc.LoadFamily( filenaam, out nested ) )
  {
    foreach( ElementId s in nested.GetFamilySymbolIds() )
    {
      symbol = doc.GetElement( s ) as FamilySymbol;
      if( !symbol.IsActive )
        symbol.Activate();
      break;
    }
  }
  else
  {
    trans.Commit();
    return Result.Failed;
  }
  trans.Commit();
```

**Response:** Thank you for the workaround!

It works fine and is a faster solution than my previous one.