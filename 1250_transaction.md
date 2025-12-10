---
post_number: "1250"
title: "Handling Transaction Status and Errors"
slug: "transaction"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'levels', 'python', 'revit-api', 'transactions', 'views']
source_file: "1250_transaction.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1250_transaction.html"
---

### Handling Transaction Status and Errors

Today, let's discuss how to [handle transaction status and errors](#4), quickly, before Autodesk University and my stint of DevDays conferences begins on Monday.

I also briefly touch on [programmatic file upload to A360](#3), [my arrival in Las Vegas and current reading](#2).

By the way, this is a special post for me, being the number 1250.
In the olden days, every new century of posts was important... now it is five quarters of a millennium.
So I have something to celebrate today, in a minor way, before entering conference mode tomorrow.

#### Arrival in Las Vegas and The Sense of an Ending

I spent today acclimatising to US Pacific time, lolling around on the spotted
[Red Rocks](https://en.wikipedia.org/wiki/Red_Rock_Canyon_National_Conservation_Area) outside
Las Vegas, reading the beautiful book
[The Sense of an Ending](https://en.wikipedia.org/wiki/The_Sense_of_an_Ending) by
[Julian Barnes](https://en.wikipedia.org/wiki/Julian_Barnes).

![Spotted rock at Red Rock](img/red_rock_spotted.png)

#### Automatically File Upload to A360

Unfortunately, what the heading suggests is currently not possible:

**Question:** I want to implement a Revit add-in that automatically uploads a file, e.g. the Revit project or some other document, to the user's Autodesk 360 storage cloud.

I haven't found anything about this even though I've searched all over the Internet.

Is there any API available to achieve this? Or maybe a sample? Or am I out of luck?

**Answer:** Unfortunately, A360 currently does not provide any public API, so driving it programmatically is not officially supported.

Alternatively, you might want to consider using BIM360 and Glue instead:

- [The BIM 360 Glue SDK](http://b4.autodesk.com/api/doc/index.shtml)
- [The BIM 360 Glue viewer and REST API](http://thebuildingcoder.typepad.com/blog/2012/12/the-bim-360-glue-viewer-and-rest-api.html)
- [BIM 360 Glue REST API authentication using Python](http://thebuildingcoder.typepad.com/blog/2012/12/bim-360-glue-rest-api-authentication-using-python.html)
- [The 360 View blog](http://thebuildingcoder.typepad.com/blog/2014/02/the-360-view.html)

Finally, of course, for a sharing based on viewing, exploring the model metadata, and anything else you would like to implement, you could use the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46).

#### All You Ever Wanted to Know About Handling Transaction Status and Errors and Were Afraid to Ask

Back to the Revit API and a deeper look than ever before at a very fundamental issue of importance to every single add-in making any kind of modification to the model, answered by Revit API interaction expert Arnošt Löbel:

**Question:** I'm working on a Revit add-in that performs a lot of work on a model with out any interaction from the user. Then after the work is complete the add-in will generate a report of the changes that have been made or problems it ran into.

With other add-ins I would always just contain transactions inside using blocks, and wrap that in a try catch block. However, with this add-in I'm going to end up with a rather complicated nesting of transaction groups, transactions, and sub transactions.

If something goes wrong during this process, I would like to be able to report back to the user exactly what went wrong, and what state the model is currently in. Thus, I'm trying to understand the various ways the transaction structure can fail, and how to check for these failures.

It seems like there are a number of places where these actions could potentially fail. Each Start and Commit method returns the actual status of the transaction, which as I understand may be something other then Started or Committed.

Assume I have single sub transaction in a transaction that's also in a transaction group. Each of these transaction levels needs to be started and committed. So that's six different places that could potentially return an incorrect status. Plus this doubles to 12 if you consider that each of these methods can also throw an InvalidOperationException.

Here is some
[sample code](https://gist.github.com/aireq/cc9faa268568349acabe) that
attempts to check for all these failure points.
This seems a little excessive though:

```csharp
namespace Sample
{
  class RevitTransactionStructure : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      var doc = commandData.Application.ActiveUIDocument.Document;

      using( var transactionGroup = new TransactionGroup(
        doc, "Transaction Group" ) )
      {
        try
        {
          var transactionGroup\_StartStatus = transactionGroup.Start();

          if( transactionGroup\_StartStatus != TransactionStatus.Started )
          {
            message = "Transaction Group did not start. Status = "
              + transactionGroup\_StartStatus.ToString();
            return Result.Failed;
          }

        }
        catch( InvalidOperationException )
        {
          message = "Transaction group threw a InvalidOperationException while starting";
          return Result.Failed;
        }

        using( var transaction = new Transaction(
          doc, "Transaction" ) )
        {
          try
          {
            var transaction\_StartStatus = transaction.Start();

            if( transaction\_StartStatus != TransactionStatus.Started )
            {
              message = "Transaction did not start. Status = "
                + transaction\_StartStatus.ToString();

              return Result.Failed;
            }
          }
          catch( InvalidOperationException )
          {
            message = "Transaction threw a InvalidOperationException while starting";
            return Result.Failed;
          }

          using( var subTransaction = new SubTransaction( doc ) )
          {
            try
            {
              var subTransaction\_StartStatus = subTransaction.Start();

              if( subTransaction\_StartStatus != TransactionStatus.Started )
              {
                message = "Sub-transaction did not start. Status = "
                  + subTransaction\_StartStatus.ToString();

                return Result.Failed;
              }
            }
            catch( InvalidOperationException )
            {

              message = "Sub transaction threw a InvalidOperationException while starting";
              return Result.Failed;
            }

            ///////////////////////////////////////////////////////
            ///////////DO SOMETHING TO THE MODEL///////////////////
            ///////////////////////////////////////////////////////

            try
            {
              var subTransaction\_CommitStatus = subTransaction.Commit();

              if( subTransaction\_CommitStatus != TransactionStatus.Committed )
              {
                message = "Sub-transaction did not commit. Status = "
                  + subTransaction\_CommitStatus.ToString();

                return Result.Failed;
              }
            }
            catch( InvalidOperationException )
            {

              message = "Sub transaction threw a InvalidOperationException while being committed";
              return Result.Failed;
            }
          }

          try
          {
            var transaction\_CommitStatus = transaction.Commit();

            if( transaction\_CommitStatus != TransactionStatus.Committed )
            {
              message = "Transaction did not commit. Status = "
                + transaction\_CommitStatus.ToString();

              return Result.Failed;
            }
          }
          catch( InvalidOperationException )
          {

            message = "Transaction threw a InvalidOperationException while being committed";
            return Result.Failed;
          }
        }

        try
        {
          var transactionGroup\_CommitStatus = transactionGroup.Commit();

          if( transactionGroup\_CommitStatus != TransactionStatus.Committed )
          {
            message = "Transaction Group did not commit. Status = "
              + transactionGroup\_CommitStatus.ToString();

            return Result.Failed;
          }
        }
        catch( InvalidOperationException )
        {
          message = "Transaction group threw a InvalidOperationException while being committed";
          return Result.Failed;
        }
      }
      return Result.Succeeded;
    }
  }
}
```

Thus I'm a little confused as to when I should be worrying about checking the return status of these methods, or the InvalidOperationException.

Is there a recommended or correct code structure for dealing with errors that may occur while starting or committing transaction groups, transactions, and sub transactions?

**Answer:** Yes, as far as I know, there definitely is a recommended way of handling transactions and transaction groups:
[encapsulate them in a .NET 'using' statement](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html).

This approach has several advantages, e.g. it is very simple, it does not worry about all those potential return codes, and it was recommended by one of our foremost Revit API interaction experts, Arnošt Löbel.

I looked at your sample code on the gist.

I do not think you need to keep track of all the transaction details.

Just log the things that work and are completed.

For instance, add a log entry for each successful transaction start and commit, and whatever operations your add-in performed in that step.

Finally, as always, test, test, test and test.

Implement a
[unit testing framework](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.16) to
automate your testing, so it can be repeated regularly and effortlessly.

Every time you find a potential problem of any kind, add that to the unit test as a new test instance.

For instance, simulate various kinds of failures and ensure that your transaction handling and logging mechanisms manage those correctly and report the expected results.

**Response:** Thanks! That blog post helps a lot. As I understand it containing a transaction in a using blog ensures that the transaction is not left open. The using block will also automatically dispose, and thus roll back the transaction before exiting the block. So I don't need to worry about manually rolling back a transaction if something fails.

However, using blocks do not catch exceptions, so I would need a try catch block around the using to catch and log any exceptions thrown. Keep in mind my code is running without user interaction so there will not be anyone to see error dialog. Any errors that occur I will just need to log. So consider the following code.

```csharp
  try
  {
    using (var trans = new Transaction(doc, "Transaction Name"))
    {
      trans.Start();

      ///////////DO SOMETHING TO THE MODEL///////////////////

      trans.Commit();
    }
  }
  catch (Exception exp)
  {
    logger.Error( "Exception occurred during transaction", exp );
  }
```

What if trans.Start fails and returns something other then Started? I'm assuming an exception would be thrown later when the code tries to modify the document outside the transaction or the code attempts to commit a transaction that was not started. In this case my code would still catch the error.

However, could trans.Commit returns something other then Committed? If so then it would not be throwing an exception, and thus the code above would not log any error. In this case I would need to check the return value of Commit and log an error if it returns something other then Committed.

```csharp
  try
  {
    using( var trans = new Transaction( doc, "Transaction Name" ) )
    {
      trans.Start();

      ///////////////////////////////////////////////////////
      ///////////DO SOMETHING TO THE MODEL///////////////////
      ///////////////////////////////////////////////////////

      var commitStatus = trans.Commit();

      if( commitStatus != TransactionStatus.Committed )
      {
        logger.Warn( "Transaction " + trans.GetName()
          + " did not commit. Status = "
          + commitStatus.ToString() );
      }
    }
  }
  catch( Exception exp )
  {
    logger.Error( "Exception occured durring transaction", exp );
  }
```

**Answer:** Thank you for drilling further down into this.

I passed your query on to our Revit API interaction expert Arnošt Löbel, and he says:

Thank you for the interesting topic. I understand workflows involving transactions can be confusing and even frustrating. It is not a simple workflow, however, thus certain level of complexity ought to be expected, at least when everything is to be done 'properly'. I will try to answer your questions as well as I can.

#### 1. Returning Status

Although all the actionable methods of transaction phases (transactions, sub-transaction, and transaction groups) do return status, it is not always necessary to test them and I must admit I do not always do it either (and still feel my code is quite robust). I can share some ground rules (of mine :-).

- The Start method rarely fails and I myself practically never test it in my own managed code. In fact it may be possible that when called from the API those methods never return anything but Started. If they could not, they would throw instead (certain rare exceptional situations exists.) It needs to be understood that we (Revit developers) use these very same classes and methods, and we have some additional workflows available to us during which the Start method may return other status. I do not have a direct proof for my claim, but I believe API users do not need to worry about the status returning from Start methods.
- Sub-transaction practically never return status other than the one expected for each respective method. It means that if the workflow is legal (i.e. the sub-transaction is within a modifiable document), the Start, Commit, and RollBack method will return what they should. I do not think I bother with testing the return values. I may have some asserts messages there instead to catch bugs, but if any of the methods fail on a sub-transaction my code will rather deal with an exception that will be thrown sooner or later. However, I would not catch exceptions specifically from sub-transaction methods. I mean do not immediately catch them (as in the user's sample below). The user should understand that if a sub-transaction fail with an exception it is very likely the users own fault (a bug) and should be addressed.
- Commit, RollBack and Assimilate methods called on a transaction group have a very high probability of succeeding, providing there are called at appropriate times and circumstances (if they are not, they will throw.) I may be not as careless with transaction groups as I am with sub-transactions, but I definitely do not go crazy about testing the results of neither the finishing methods. Again, I would probably be inclined to use an Assert instead.
- Transaction is certainly the class for which testing the results of methods programmers need to be rather cautious. It is unlikely that RollBack would fail (meaning: it should return RolledBack), but it is reasonably likely for a Commit to return other status than Committed. The programmer is definitely encouraged to check the status, particularly if the transaction is not the end of the model-modifying action. Programmers (both external and internal) should test the status and take appropriate actions. For example, if the status is RolledBack, than perhaps something else needs to be undone too. Or if the status is Pending (not too common in the API, though possible), the programmer must not do anything to the model as long as the transaction is in the 'pending' state.

#### 2. Transaction.Commit

This is specific to the Transaction objects only and to the Commit method specifically. It needs to be understood that returning status other than Committed from the Commit method cannot always be seen as a failure (per se) of the commitment process. It is perfectly expected and legal to receive the RolledBack status instead, because it might be a result of the end-user's actions (or other applications' updaters or failure handlers.) Rolling back instead of committing is simply Revit's way to inform the transaction owner that there were problems with the model, thus the changes could not be accepted. Those problems might have been caused by the transaction's owner as well as by other applications; it is not always possible to determine the exact cause. However, as I said, it is not an unexpected situation and it does not always need to be reported as an error. (Unless, of course, it is in unit testing when the testing application 'knows' the transaction should commit.)

Handling this case may vary depending on the complexity of the action. If it is a simple action (e.g., opening a transaction, making few modifications, and closing the transaction) I would not even worry about reporting that Commit did not return Committed. In most cases it should be already apparent if not obvious to the end user. (There would be a mini-warning, or failure handling dialog). In cases of rather complex actions with multiple transactions the programmer needs to be more cautious, because one change of the model may depend on the success of some preceding action. For example, if I have two complex changes to the model and each of the changes is in an isolated transaction, I may need to know whether or not the first transaction committed before I attempt to start the second transaction. Not because the second transaction would not start, but rather because the second transaction (and the change within) may no longer make sense. For example, I may do:

```csharp
  using(TransactionGroup tgroup = new TransactionGroup(document))
  {
    tgroup.Start();
    using(Transaction trans = new Transaction(document))
    {
      trans.Start("Change 1");

      MakeMyChanges(document);

      if(trans.Commit() != TransactionStatus.Commited)
      {
         tgroup.RollBack();
         return err\_code; // or throw an exception
      }

      trans.Start("Change 2");

      MakeMyOtherChanges(document);

      if(trans.Commit() != TransactionStatus.Commited)
      {
         tgroup.RollBack()
         return err\_code; // or throw an exception
      }
    }
    tgroup.Commit();
  }
```

Note: the code above assumes that the Pending status may not be returned from the Commit method. Otherwise a more complicated implementation would be needed. (As I said earlier, transaction Pending is not very common workflow in external applications. It is a subject for its own dedicated discussion.)

#### 3. Catching Exceptions

Unlike in the user's example, I rarely if ever catch exceptions specifically from methods of transaction phases. By which I mean: As long as I can I write my code in a way that makes me believe my calling those methods should not cause exceptions. Thus my exception-handling would be on a higher level somewhere up the code chain. Perhaps it depends in each programmer's style, but I am a believer in a defensive and cautious programming – I want to get an exception only as a real surprise; if I can avoid it I write the code so exceptions are not thrown at me. For example, if it is unclear whether a document is currently modifiable or not, I test it before I attempt to start a sub-transaction. (And if it is not I'd throw my own exception back at the caller who called my sub-transaction using method.)

So, yes, my recommendation for catching exceptions would be much like the user's second and third sample (the one with the Logger in the Catch.)

As a response to the user's code: It is a ground rule for the API programmers that if they receive an Invalid Operation exception (or Invalid Argument exception), they may consider debugging their code and fix the problem so Revit would not throw the exception. These two exception kinds is Revit's response to programmers not using the API properly. We in the Revit API group certainly strive to provide such an API that allows programmers to test before attempting to do something invalid with the API. That is, I believe, also the case with the transaction API. We document the circumstances in which the methods may fail and we provide methods to test whether or not such circumstances are currently met.

#### 4. Using 'using' Blocks to Roll Transaction Phases Back Automatically

I personally do not write code that depends on the automatic roll-back.
I explicitly roll back all phases that I mean to roll back.
I use the automatic rollback for exception handling only – meaning that I do not catch exceptions just to roll back a transaction phase because I know the forced disposal will do it for me. Again, this may be a personal taste and style, but I like it that way. I do not like code that I need to guess what it does.

Hopefully it is not too confusing. My intention is to make user to realize that there is no application-creation a cookie cutter. Small applications need different approach than more complex or enterprise solution would require. There is always a very fine line determining what level of error checking that one particular application should employ. The goal is to keep it simple, but not too simple. A good error handling makes bug finding and problem solving manageable.

Thank you very much, Arnošt, for the detailed answer.

Coming up next: DevDays and AU.

Stay tuned.