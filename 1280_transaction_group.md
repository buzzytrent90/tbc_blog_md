---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.665838'
original_url: https://thebuildingcoder.typepad.com/blog/1280_transaction_group.html
post_number: '1280'
reading_time_minutes: 8
series: transactions
slug: transaction_group
source_file: 1280_transaction_group.htm
tags:
- csharp
- elements
- family
- parameters
- revit-api
- sheets
- transactions
- views
title: Using Transaction Groups
word_count: 1575
---

### Using Transaction Groups

We repeatedly discussed the optimal usage and error handling of transactions, mainly based on the expert advice of Arnošt Löbel, Sr. Principal Engineer of the Autodesk Revit R&D team:

- [Scope and Dispose of Transactions](http://thebuildingcoder.typepad.com/blog/2012/04/scope-and-dispose-transactions.html)
- [Using Using Automagically Disposes and Rolls Back](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html)
- [Handling Transaction Status and Errors](http://thebuildingcoder.typepad.com/blog/2014/11/handling-transaction-status-and-errors.html#4)

Let's complete this with his most recent advice on handling transaction groups to
[combine multiple transactions into one undo](http://forums.autodesk.com/t5/revit-api/combine-multiple-transactions-into-one-undo/m-p/5502149):

**Question:**
I am creating an application that must do the following steps:

1. Load a FamilySymbol from a file.
2. Get the FamilyManager from the Family document and access some parameters.
3. Place an instance of the FamilySymbol into the project.

In order to load the FamilySymbol into the project, I have to have a transaction running. Then, in order to open the Family document, the transaction has to be closed. In order to place an instance of the component, a transaction has to be running.

This causes the application to create two undo points – one for each transaction.

Is there a way to combine the two transactions into a single undo point so the entire action of the command can be undone?

I am using the Manual TransactionMode.

**Answer:**
I am glad you asked.
Yes, it can certainly be done.
It is what Transaction Groups are for.
The steps are simple:

1. Start a transaction group. Give it a name.
2. Do your first transaction.
3. Open the family document.
4. Load the family into the project
5. Close the transaction group by calling Assimilate method

The last operation will merge all transactions that have been committed within the group into just one transaction.
It will bear the name of the transaction group.

When working with transaction group, make sure you scope them by the 'using' block, just like you would with transactions.

I hope this helps. There is more info if needed in the RevitAPI.chm file.
Simply look up the TransactionGroup class.

**Response:**
Ok, let me make sure I'm doing this correctly when dealing with exceptions.

When the TransactionGroup is rolled back, all of the committed and uncommitted Transactions inside it are rolled back?

When a TransactionGroup is committed or assimilated, all of the Transactions inside of it must already be committed?

```csharp
  using( TransactionGroup transGroup = new TransactionGroup( document ) )
  {
    transGroup.Start( "Transaction Group" );

    using( Transaction firstTrans = new Transaction( document ) )
    {
      try
      {
        firstTrans.Start( "First Transaction" );

        // do some stuff

        firstTrans.Commit();
      }
      catch
      {
        transGroup.Rollback(); // <-- We do not have to roll back firstTrans?

        return Result.Failed;
      }
    }

    using( Transaction secondTrans = new Transaction( document ) )
    {
      try
      {
        secondTrans.Start( "Second Transaction" );

        // do some stuff

        secondTrans.Commit();
      }
      catch
      {
        transGroup.Rollback(); // <-- We do not have to roll back secondTrans?

        return Result.Failed;
      }
    }

    transGroup.Assimilate();

    return Result.Succeeded;
  }
```

**Answer:**
Regarding to your two questions:

Yes and No, actually.
Your statement is almost correct except one part, which I highlight here: "... all of the committed and **uncommitted** Transactions..."
The thing is that you cannot close (that is either Commit, RollBack, or Assimilate) a transaction group while there is an uncommitted transaction still open.
All transactions that start inside a transaction group must be either properly committed or rolled back before any of the aforementioned methods can be called upon the transaction group object.

Yes, basically what I have just stated above.
All transactions inside a group must be either committed or rolled back.
If you try to call any of the transaction-group closing methods while there is still a transaction open (in the same document), you will get an exception.

As for your code snippet, it is quite all right, except for two details, one more important than the other.
I’ll start with that one:

You do not have to Roll Back the open transaction in the catch blocks, because it will be rolled back automatically upon leaving its using block.
However, in your case it can in fact even be dangerous to call RollBack in the catch.
The problem is that you may actually (in theory) get an exception from the Start method too, and if you do, you would most certainly get another one when attempting to roll the transaction back.
That would be an exception thrown while exception handling, and that is never a good thing.
Thus, if you want to call RollBack as you do, you need to test whether the transaction has actually started.
(There is a method for that or you could test the status.)

This second problem is less dangerous, but could lead to unexpected results.
(Unexpected as your code goes, I mean.)
A programmer should anticipate that committing a transaction does not need to succeed.
It can fail and it is not all that uncommon, in fact.
There are several possible reasons for such an outcome, one of which is a failed regeneration. When that happens and your transaction actually fails, it is most likely that you would not like to continue with opening another transaction that might only work if the previous transaction succeeded.
Ignoring the results of transactions can lead to a spiral of errors and failures that might be rather challenging to untangle.

Lastly, a very minor thingy, that is not a problem, but could make your code simpler. You do not have to have instantiate a new transaction for each and every transaction. You can re-start an existing one instead; simply give it a proper name in the Start method.

Putting all the above comments into action, I’ve rewritten your simple snippet as follows:

```csharp
  using( TransactionGroup transGroup
    = new TransactionGroup( document ) )
  {
    using( Transaction trans
      = new Transaction( document ) )
    {
      try
      {
        transGroup.Start( "Action" );

        trans.Start( "First Transaction" );

        // do some stuff

        if( trans.Commit() != TransactionStatus.Committed )
        {
          return Result.Failed;
        }

        trans.Start( "Second Transaction" );

        // do some more stuff

        trans.Commit();

        if( trans.Commit() != TransactionStatus.Committed )
        {
          return Result.Failed;
        }

        transGroup.Assimilate();
      }
      catch
      {
        return Result.Failed;
      }
    }
    return Result.Succeeded;
  }
```

**Response:**
One more question:
In the same logic as checking that Transaction.Commit returns TransactionStatus.Committed, is it a good idea to check that Transaction.Start returns TransactionStatus.Started?
Is there a reason why it wouldn't?

**Answer:**
I have been asked that very question many times in the past.
My answer is that even though it is indeed possible in theory that the Start method returns a status other then Started, it is rather unlikely that it actually happens, if is it all possible in the public API. (Note: We, Revit programmers use the same classes internally and for us it is a tiny bit more likely that such a situation may happen.) In most situations that come to my mind the Start method would raise an exception if it is not possible (or permitted) to start a transaction, or any one of the three transaction-phase objects for that matter.

And since I am back on this particular case, there is one small detail I could have changed in the snippet I wrote above:
I could have explicitly roll back the transaction group in the two places where I return with a failure due to a transaction failing to commit.
Although this explicit closing of the group is not necessary (for it will be closed implicitly upon leaving its using block), I do tend to write my code that way.
The reason is more style related; I simply prefer to make it clear to anyone who comes across my code that I knew what I was doing (here, I am aware that the group must be rolled back.)

Thank you very much, Arnošt, for your on-going support and expert in-depth advice!

#### Finally Finished Eliminating The Building Coder Samples Obsolete API Usage

I finally completed the on-going effort to eliminate all Obsolete API Usage from The Building Coder samples.

After [the last step](http://thebuildingcoder.typepad.com/blog/2015/02/list-pipe-sizes-and-more-obsolete-api-usage-removal.html),
there were still [15 warnings](zip/bc_migr_2015_g.txt) left:

- ViewSheet.Views is obsolete: Use GetAllPlacedViews() instead.
- Autodesk.Revit.Creation.Document.NewPipe: Use Pipe.Create() instead.
- Definitions.Create(string, Autodesk.Revit.DB.ParameterType, bool) is obsolete: Use Create(Autodesk.Revit.DB.ExternalDefinitonCreationOptions) instead.
- Element.get\_Parameter(string) is obsolete: More than one parameter can have the same name on a given element. Use Element.Parameters to obtain a complete list of parameters on this Element, or Element.GetParameters(String) to get a list of all parameters by name, or Element.LookupParameter(String) to return the first available parameter with the given name (11 occurrences).

I cleaned them up systematically, step by step, and published
[release 2015.0.117.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.117.2),
which is completely clean and compiles with no warnings at all.

Look at this
[diff between 2015.0.117.1 and 2015.0.117.2](https://github.com/jeremytammik/the_building_coder_samples/compare/2015.0.117.1...2015.0.117.2) to
see exactly what I changed.

This is an important milestone to accomplish before tackling the migration to upcoming future versions of the API.

As always,
[The Building Coder sample GitHub repository](https://github.com/jeremytammik/the_building_coder_samples) provides
the most up-to-date version.