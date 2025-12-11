---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.517347'
original_url: https://thebuildingcoder.typepad.com/blog/0750_dispose_transact.html
post_number: '0750'
reading_time_minutes: 3
series: general
slug: dispose_transact
source_file: 0750_dispose_transact.htm
tags:
- csharp
- revit-api
- transactions
- views
- walls
title: Scope and Dispose of Transactions
word_count: 681
---

### Scope and Dispose of Transactions

Arnošt Löbel added an important comment on the sample code that I published for
[refreshing a view](http://thebuildingcoder.typepad.com/blog/2012/04/devcamp-and-refresh-display-for-a-kinetic-facade.html):

It is for the best if all transaction objects are always properly scoped, e.g. within a C# 'using' block.
This includes transactions, sub-transactions, transaction groups, and other scope-like objects such as StairsEditScope.
Not doing so can yield some pretty weird situations when exceptions come into play.

In your example, proper scoping can be implemented by adding 'using' statements, which automatically dispose of the objects when they leave the scope, like this:
```csharp
  using( TransactionGroup group
    = new TransactionGroup( doc ) )
  {
    group.Start( "Muda a fachada" );

    using( Transaction tran
      = new Transaction( doc ) )
    {
      tran.Start( "Step 1 " );
      Calcula\_Padrao( doc, Paneis, 0.10 );
      tran.Commit();

      uidoc.RefreshActiveView();

      Thread.Sleep( 2000 );

      tran.Start( "Step 2 " );
      Calcula\_Padrao( doc, Paneis, 0.4 );
      tran.Commit();

      uidoc.RefreshActiveView();

      Thread.Sleep( 2000 );

      tran.Start( "Step 3 " );
      Calcula\_Padrao( doc, Paneis, 0.05 );
      tran.Commit();

      uidoc.RefreshActiveView();

      Thread.Sleep( 2000 );

      tran.Start( "Step 4 " );
      Calcula\_Padrao( doc, Paneis, 0.4 );
      tran.Commit();

      uidoc.RefreshActiveView();
    }
    group.Assimilate();
  }
```

This is the safest way to guarantee that scoped objects are destroyed in the proper order.
In this case, the transaction must be destroyed (and committed or rolled back) before the group is finished.
If not, an exception will be thrown, and since it would be thrown during garbage collection, the consequences are hard to predict.

Here is
[FachadaCinetica2.zip](zip/FachadaCinetica2.zip) containing
an updated version of the kinetic facade sample command with proper scoping and including the complete Revit 2013 source code, Visual Studio solution and add-in manifest.

Oh, and by the way, the final result of the kinetic facade sample up and running can be viewed on Prof. Jos Luis Menegotto's
[Kinetics Architecture](https://sites.google.com/a/poli.ufrj.br/dc_menegotto/programacao-autolisp/ArqCin) page
for both AutoCAD and Revit.

**Question:** How is it if I have a normal external command and just start and commit a transaction within the command Execute method?

Does that require me to add a 'using' block around it as well, or does it get cleaned up properly by itself automatically?

**Answer:** Basically, the same rules apply.
I recommend always scoping inside 'using' blocks.

Again, it makes no difference if no unhandled exceptions are thrown during your command, but if they are, it might trigger some harsh treatment from the API firewall:

- If you only have one transaction, which is still open: it will be rolled back and render your command unsuccessful (with a message to the user). If the GC destroys it before the API framework finds that it was left open (unlikely, but possible), it will still be rolled back, but there may be an extra error message in the journal.- If you have more than one transaction in your command and an unhandled exception is thrown while your last transaction is still open, Revit will undo ALL your transactions; your command will be a complete failure.- Things may get complicated with sub-transactions and a transaction group. Those may be nested, and Revit requires that the phases are closed in reverse order to how they were opened, meaning the inner-most phases must be closed first, while the outer-most (most likely a transaction group) must be closed last. Things can go wrong if the GC starts collecting those unfinished phases and finalises them out of order, which is perfectly possible. Revit will throw an exception when that happens, which is not going to be good if this is already happening in an exception. The framework always tries to resolve the situation as best it can, but I have heard of some unhappy endings (if I may say so).

In addition, the API may start providing more of these scope-like objects in the future, and it is better get into habit of scoping them properly in your C# and VB code, something provided automatically and for free in C++.