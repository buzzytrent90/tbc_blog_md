---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.164952'
original_url: https://thebuildingcoder.typepad.com/blog/1034_catch_except_edit_slab.html
post_number: '1034'
reading_time_minutes: 5
series: general
slug: catch_except_edit_slab
source_file: 1034_catch_except_edit_slab.htm
tags:
- csharp
- elements
- revit-api
- transactions
- windows
title: Handle Your Own Exceptions and Edit Slab Boundaries
word_count: 1027
---

### Handle Your Own Exceptions and Edit Slab Boundaries

Today, we continue yesterday's discussion on the need to
[handle your own exceptions](http://thebuildingcoder.typepad.com/blog/2013/10/handle-your-own-exceptions.html) and
mention Joe Ye's solution to
[edit an existing slab boundary](#3).

#### How to Handle Your Own Exceptions

Yesterday we explained why Revit cannot gracefully handle all your exceptions for you and the need to do so yourself in all parts of your Revit add-in, especially the modeless ones that Revit is not even aware of.

Better still, of course, is to avoid all modeless activity completely, if you can.

This led to the following further discussion:

**Question:** When I said "unsafe access" I meant accessing the Revit model from a context that Revit is unaware of – not in event handlers, dynamic updaters, etc.
This unsafe access could be either direct or indirect: directly, for example by creating a public member `public Document Document {get;set;}` in a modeless form; indirectly, e.g. by reading some elements from the model, mapping them to another "safe" elements using a lazy query, like `elements = elementsFromModel.Select(x => new SomeObject(x))` and accessing those from another thread.

The situation with unhandled exceptions is rather clear; I sent you yesterday's add-in just as an interesting example; to fix this particular case, it is enough to simply set Focusable="False" on the hyperlink element.
The most difficult step to solve it was discovering the action sequences that led to the crash.
The problem is known and old, as you can see from this discussion on
[InvalidOperationException: 'System.Windows.Documents.Hyperlink' is not a Visual or Visual3D](http://social.msdn.microsoft.com/Forums/vstudio/en-US/5982cafe-f75b-42b4-99dc-50d3a81b30b0/invalidoperationexception-systemwindowsdocumentshyperlink-is-not-a-visual-or-visual3d) in
the Visual Studio discussion forum.

But I don't know how to remain the modeless form and properly contain such errors: try-catch is useless, because the error throws in a different thread.

The only approach I found so far is to implement my own exception handler, e.g. like this:

```csharp
  private void CurrentDomainOnUnhandledException(
    object sender,
    UnhandledExceptionEventArgs e )
  {
    var exception = e.ExceptionObject as Exception;
    if( exception == null ) return;
    TaskDialog.Show( "exception",
      string.Format( "{0}\n{1}",
        exception.Message, exception.StackTrace ) );
  }
```

I can then subscribe to that in external command Execute method like this:

```csharp
  AppDomain.CurrentDomain.UnhandledException
    += CurrentDomainOnUnhandledException;
```

In this case, I can at least see the exception details before Revit crashes.

The question is whether there is any way at all to stop the exception from being passed on up to the main Revit process.

**Answer:** You could add this question as a comment to the blog post and also open a discussion thread on it in the Revit API forums, e.g. in the Autodesk and AUGI discussion groups, to see what solutions other developers are using.

In C++ I believe you can specify what exception handler to call, so maybe this whole thing can be handled correctly and completely using a mixed mode C++ application.

But, still, why do you have to use any modeless Revit add-ins at all?

If they are modeless, you might as well implement them completely outside of Revit, and only communicate with them via the Idling event:

- Remove all the modeless activity from your add-in.
- Implement a separate independent stand-alone executable that does all the modeless stuff.
- Connect the two in a similar way as you currently do.

That would solve your problem, wouldn't it?

Obviously this is only useful if the application is truly modeless.
If a modeless dialogue needs to talk to Revit for some reason, you would either have to integrate into your add-in after all, or devise some mechanism to communicate from the stand-alone application with your add-in to extract the required information.

I also discussed the matter further with the development team, and they say:

The only 100% safe approach is not to let the add-in code have unhandled exceptions in the Revit process.

See this MSDN discussion on
[using AppDomain isolation to detect add-in failures](http://blogs.msdn.com/b/clraddins/archive/2007/05/01/using-appdomain-isolation-to-detect-add-in-failures-jesse-kaplan.aspx).

Revit currently does not use different AppDomains for add-ins.
But even if it did, it wouldn't solve this problem:

Unhandled exceptions on add-in originated threads are harder because the host isn't on the stack and can't catch the exceptions.
Starting with the CLR v2.0, unhandled exceptions on child threads will cause the entire process to be torn down.
Thus it is impossible for a host to completely recover from this.
With a little work though, it can detect which AppDomain and – assuming it gives each add-in its own domain – add-in caused the problem and log the failure before exiting and even restarting.

If it's unhandled, it's unhandled down to the termination of the host process.
Thus, surround all possible code with try-catch, or introduce a separate process to do the add-in work to avoid this situation.
Or do as you suggested and log the unhandled exceptions.
Possibly, those details could be written to the journal to help understand what happened.

#### Editing an Existing Slab Boundary

After a lengthy Autodesk discussion group thread on
[editing an existing slab boundary](http://forums.autodesk.com/t5/Revit-API/Edit-existing-slab-boundary/td-p/4407975),
my colleague
[Joe Ye](http://adndevblog.typepad.com/aec/joe-ye.html)
now published a solution on the AEC DevBlog using the
[temporary transaction trick](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html):
[Change the boundary of floors/slabs](http://adndevblog.typepad.com/aec/2013/10/change-the-boundary-of-floorsslabs.html).

Joe temporarily deletes the slab, which deletes and returns the element ids of the associated boundary lines.

The transaction can be rolled back, so the slab and its boundary remain unchanged.

After that, the boundary lines can be modified in a new transaction, modifying the slab as well.

Many thanks to Joe for solving this!