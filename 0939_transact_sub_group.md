---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.950868'
original_url: https://thebuildingcoder.typepad.com/blog/0939_transact_sub_group.html
post_number: 0939
reading_time_minutes: 3
series: general
slug: transact_sub_group
source_file: 0939_transact_sub_group.htm
tags:
- python
- revit-api
- transactions
- views
title: Transactions, Sub-Transactions and Transaction Groups
word_count: 580
---

### Transactions, Sub-Transactions and Transaction Groups

Here is a recent query from a developer that led to a nice succinct overview of transactions, sub-transactions and transaction groups by Arnošt Löbel.

This makes a change from the recent rash of monster posts.
The last ten posts were all rather overwhelming in size, and hopefully impressive in content as well.
From Aril 16 until 29, I posted a total of 26228 words, 245504 bytes.

So let's keep this one short, sweet and to the point.

**Question:** When should I use a transaction group with transactions versus transaction with sub-transactions?

At first glance, it looks as if both can achieve the same result, but there must be some kind of difference, mustn't there?

**Answer:** This question is actually covered at large in the API documentation for the respective classes.
Of course, you could write a whole book about the detailed usage of transaction phases (i.e. Transaction, Transaction groups, Sub-Transaction), if you wanted to, but the documentation already covers all the really necessary information.

Here is a brief summary:

- A **transaction** is a phase that is needed to make modifications to a model. A transaction is undoable. Once committed, it appears on the undo/redo menu, where the end user can make it undone or later redone. Transactions cannot be nested, for only one client can modify the model at any given time.
- A **sub-transaction** marks a phase within an open transaction. It is to control (if a control is needed) a set of modifications done in a transaction, meaning that the modification enclosed in a sub-transaction can be rolled back as if they have not happened. In a sense, it allows an application to go back to a certain point (the start of the sub-transaction) in a sequence of model changes. Sub-transactions can be nested. A sub-transaction cannot be undone/redone once committed.
- A **transaction group** can control a set of transactions in the sense that even transaction that have already been successfully submitted can be rolled back if they were committed while inside an open transaction group. This give an application a way of undoing past transaction if some other transactions later (but still within the group) do not go as well as expected. For example, if three transactions are required (for whatever reason) to achieve something, and it is necessary to commit all three for the changes to make sense, the transactions could be controlled by a group. If the second or the third transaction fails, the whole group can be rolled back. Transaction groups can be nested, but they cannot be undone/redone, i.e. there is no trace of them in the undo/redo menu for the end user to see.

The
[temporary transaction trick touch-up](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html) describes
a scenario using a transaction group to enable a transaction to be committed, so the model is updated and the new state can be queried, and still enabling the whole action can be undone afterwards.

For more information on the transaction framework, please refer to Arnošt's Autodesk University presentation
[CP3426 Core Frameworks in the Revit API](http://thebuildingcoder.typepad.com/blog/2012/11/au-classes-on-python-ui-server-and-framework-apis.html#4).

**Response:** Thank you!
I use the temporary transaction trick quite a lot, so I'll definitely start using transaction groups now as well.