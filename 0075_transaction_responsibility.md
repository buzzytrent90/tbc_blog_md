---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.6
content_type: documentation
optimization_date: '2025-12-11T11:44:13.320386'
original_url: https://thebuildingcoder.typepad.com/blog/0075_transaction_responsibility.html
post_number: '0075'
reading_time_minutes: 1
series: transactions
slug: transaction_responsibility
source_file: 0075_transaction_responsibility.htm
tags:
- csharp
- parameters
- revit-api
- transactions
title: Transaction Responsibility
word_count: 203
---

### Transaction Responsibility

The last post on
[transactions](http://thebuildingcoder.typepad.com/blog/2009/01/transactions.html)
explained the need to open a transaction of our own before trying to modify a document during an event.
That is actually not all.
We also need to remember that starting a transaction is not permitted at all times, and also that starting a new transaction or ending one may fail.
This leaves us with a certain amount of responsibility.
Therefore, we should add some error handling to ensure that failures are handled gracefully.
The sample should be amended like this:

```csharp
void application\_OnDocumentNewed( Document doc )
{
  // we cannot modify the document
  // unless a transaction is started

  if( doc.BeginTransaction() )
  {
    // once a new transaction is started
    // we are responsible for ending or
    // aborting it, so we have to put
    // everything in a try-catch block

    try
    {
      CreateUserDefinedParameters( doc );

      // we are responsible for ending
      // the transaction we started

      doc.EndTransaction();
    }
    catch( Exception )
    {
      // if we cannot finish what we wanted
      // we should probably abort the whole thing

      doc.AbortTransaction();
      throw; // re-throw the exception
    }
  }
}
```

Thank you to Arnost Lobel and Harry Mattison for providing this feedback!

I have arrived in Perugia by now ... rather slow progress, but getting there.