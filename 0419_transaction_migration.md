---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: code_example
optimization_date: '2025-12-11T11:44:13.918447'
original_url: https://thebuildingcoder.typepad.com/blog/0419_transaction_migration.html
post_number: 0419
reading_time_minutes: 2
series: transactions
slug: transaction_migration
source_file: 0419_transaction_migration.htm
tags:
- csharp
- elements
- references
- revit-api
- transactions
- walls
title: Transaction Migration Errors
word_count: 331
---

### Transaction Migration Errors

Joo Lee just submitted a
[comment](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html?cid=6a00e553e168978833013485f433e1970c#comment-6a00e553e168978833013485f433e1970c) pointing
out an error or two in the flat migration of the CmdWallFooting command, which demonstrates an approach to retrieve the
[host reference](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html) using
a wall footing:

**Question:** I tested your CmdWallFooting command in Revit Architecture 2011.
But it does not work.
The transaction is not started.
How can I get the transaction to work?

**Answer:** Thank you very much for pointing this out!

This command was flat migrated from Revit 2010 and never tested.
I tested it now as well and discovered two problems with it:

1. It did not have any active transaction open at all, since it was set to read-only transaction mode.- The Revit 2010 transaction was replaced by a Revit 2011 sub-transaction, which requires a call to Start before it can be used.

Fix 1: The read-only transaction mode causes the following error when trying to delete the wall: "Attempt to modify the model outside of transaction."

It can be fixed by specifying the transaction attribute like this:
```csharp
  [Transaction( TransactionMode.Automatic )]
```

Alterntively, you can specify manual transaction mode and start your own main transaction.
That requires a little bit more coding and gives you full and explicit control in return.

Fix 2: The sub-transaction can be started by calling its Start method. Then the code to temporarily delete the wall looks like this:
```csharp
  SubTransaction t = new SubTransaction( doc );
  ICollection<ElementId> delIds = null;
  try
  {
    t.Start();
    delIds = doc.Delete( wall );
    t.RollBack();
  }
  catch ( Exception ex )
  {
    message = "Deletion failed: " + ex.Message;
    t.RollBack();
    return Result.Failed;
  }
```

Here is
[version 2011.0.74.3](zip/bc_11_74_3.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the updated command.

Thank you very much again, Joo Lee, for picking this up and pointing it out!