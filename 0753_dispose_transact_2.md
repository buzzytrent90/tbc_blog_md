---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.523454'
original_url: https://thebuildingcoder.typepad.com/blog/0753_dispose_transact_2.html
post_number: '0753'
reading_time_minutes: 2
series: general
slug: dispose_transact_2
source_file: 0753_dispose_transact_2.htm
tags:
- csharp
- revit-api
- transactions
title: Using Using Automagically Disposes and Rolls Back
word_count: 406
---

### Using Using Automagically Disposes and Rolls Back

I talked about
[scoping and disposing of transactions](http://thebuildingcoder.typepad.com/blog/2012/04/scope-and-dispose-transactions.html) last
week, and received prompt and interesting comments from two competent and experienced Revit add-in developers, Danny Polkinhorn and Guy Robinson (your name has to end in 'y' to be really good – tongue in cheek).

**Danny says:** Should we also use Try...Catch blocks as well?
```csharp
  using( Transaction trans
    = new Transaction( doc ) )
  {
    try
    {
      trans.Start();
      //code here
      trans.Commit();
    }
    catch( Exception ex )
    {
      trans.RollBack();
      //exception code here
    }
  }
```

Do we need to roll back the transaction or does the Using block take care of that?

**Guy replies:** I'm with Danny here, prefer to gracefully fail so use this format:
```csharp
  TransactionGroup group;
  try
  {
    using( group = new TransactionGroup( doc ) )
    {
      group.Start( "Some group" );
      // . . .
    }
  }
  catch( Exception )
  {
    if( group.HasStarted() && !group.HasEnded() )
    {
      group.RollBack();
    }
  }
```

As so often in the past, I leave the final verdict to Arnošt Löbel, who replies with a clear 'no' to the suggestion of adding an exception handler around the 'using' block, because, as implied by the title, it automatically disposes, rolls back, and even handles exceptions:

**Answer:** No, there is no need to explicitly roll back a transaction (or sub-transaction, transaction group) declared in a 'using' block. The destructor of the object will take of the rolling back when the object leaves the 'using' block.

However, it is still the responsibility of the programmer to commit the object properly.
For example, using what Guy wrote:
```csharp
  using( TransactionGroup group
    = new TransactionGroup( doc ) )
  {
    group.Start( "Some group" );

    // do something here

    // need to commit explicitly to
    // avoid implicit rolling back

    group.Commit();
  }
```

By the way: I always commit or rollback explicitly if it is under my control.
I only rely on the destructor for exceptions.
I believe it makes the code easier to comprehend.

The snippet above is equivalent to (but more elegant than) the following:
```csharp
  TransactionGroup group
    = new TransactionGroup( doc );

  try
  {
    group.Start( "Some group" );
    // do something here
    group.Commit();
  }
  catch( Exception )
  {
    // this needs to be tested to
    // avoid another potential exception

    if( group.HasStarted() )
    {
      group.RollBack();
    }

    // should re-throw if you do
    // not know how to handle

    throw;
  }
```

Adding a 'using' block around this try-catch (like Guy did) is completely unnecessary and worthless.

Again: If you always use 'using' for all scope object, you code will be more elegant, transparent and safer.