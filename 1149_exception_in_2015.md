---
post_number: "1149"
title: "Multithreading Throws Exceptions in Revit 2015"
slug: "exception_in_2015"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'references', 'revit-api', 'transactions', 'views', 'windows']
source_file: "1149_exception_in_2015.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1149_exception_in_2015.html"
---

### Multithreading Throws Exceptions in Revit 2015

I have received several cases lately dealing with add-ins that have been working fine in previous versions of Revit and now throw various exceptions after migration to Revit 2015.

Similar issues were also raised in the Revit API discussion forum, e.g. exploring a
[Revit 2015 exception with a CustomExporter](http://forums.autodesk.com/t5/Revit-API/Revit-2015-exception-with-Autodesk-Revit-DB-CustomExporter/m-p/4981202).

In all of the cases so far, these problems were due to attempts to access the Revit API from a multi-threaded context.

As the Revit API documentation clearly states, multi-treaded use of the Revit API is not supported and never has been.

This is spelled out in those very words in the
[Revit API developers guide](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-F0A122E0-E556-4D0D-9D0F-7E72A9315A42).

If you search it for the single word "thread", the first hit displayed is the section on
[Deployment Options](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-411A9D2F-9748-46AC-9C2F-EA2D90DABCFD):

#### Deployment Options

The Autodesk Revit API supports in-process DLLs only. This means that your API application will be compiled as a DLL loaded into the Autodesk Revit process. The Autodesk Revit API supports single threaded access only. This means that your API...

Let's look at two recent cases dealing with this:

**Question 1:** I am migrating my Revit add-in from Revit 2014 project to Revit 2015.

In it, I loop through the parameters of an element, and check the parameter StorageType.
If it's Double, e.g., I call AsDouble to retrieve the parameter value.

This code worked fine in Revit 2013 and 2014.
In 2015, it throws an exception error and exits Revit.

Many other properties on the parameter also generate the same exception, e.g. HasValue, IsReadOnly and so on.

If I run in Debug mode, an AccessViolationException is thrown:

![AccessViolationException](img/ky_accessviolationexception.png)

Running my add-in program directly from Revit in release mode causes an unrecoverable error:

![Unrecoverable error](img/ky_unrecoverable_error.png)

After that, Revit closes:

![Error report](img/ky_error_report.png)

My add-in calls a method that runs in its own background worker process.

As said, this code worked fine for several years, e.g. in Revit 2012 and 2013, and the problem only appears in Revit 2015.

**Question 2:** Having previously developed a custom add-in for Revit 2013 and 2014, I am now testing a forthcoming release for Revit 2015 compatibility. After changing references to the 2015 versions of RevitAPI.dll and RevitAPIUI.dll and the target framework to .NET 4.5, I attempted to run the program via the 2015 Add-in Manager. It loaded without any issues, but when it attempted to execute code pertaining to a background worker class (which updates a UI progress bar) Revit crashed immediately.

Are add-ins no longer able to execute background worker classes on a separate thread from the main Revit UI thread?
If so, this seems like a significant change.

Essentially, I am trying to get a progress bar to update while performing time-consuming calculations based on elements in the Revit model (no transactions, just collecting and reading parameter values).

**Answer:** You are perfectly correct in your assessment of the problem:

Revit add-ins are not allowed to access the Revit API from separate threads, only from the main Revit UI thread.

However, this is not a significant change, and not even a change at all.

This has always been the case.

For better protection of the add-ins developers and their users, it is now more strictly enforced.

Please look at the
[Revit API developers guide](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-F0A122E0-E556-4D0D-9D0F-7E72A9315A42) and
its section on
[Deployment Options](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-411A9D2F-9748-46AC-9C2F-EA2D90DABCFD) quoted
above.

If you were previously accessing the Revit API from a separate thread, you and your users were at great risk.

If all you are doing in the separate thread is display a progress bar, with no Revit API access at all, there should be no problem.

Revit API access is however limited to the main thread.

This very topic was explicitly discussed on The Building Coder three years ago, in a post with a title that says it all:
[no multithreading in Revit](http://thebuildingcoder.typepad.com/blog/2011/06/no-multithreading-in-revit.html).

Regarding the task at hand, you will have to collect all the required Revit element data in one go beforehand, from the main execution thread.

Once you have it, you can happily launch your processing and progress bar in a separate thread, as long as you make no more Revit API calls in it.

**Response:** Thanks for your feedback. The challenge I was facing is that it is precisely the Revit API calls (which consist of filtered element collectors and get parameter values) that consume the vast majority of operation time, i.e. ca. 30 seconds for a moderately sized Revit model. Hence, I chose to introduce a background worker that would enable me to update a progress bar during the operation, so that our users would know if there was time to go get a coffee.

I found your progress bar example in the
[ADN Revit MEP sample AdnRme](http://thebuildingcoder.typepad.com/blog/2012/05/the-adn-mep-sample-adnrme-for-revit-mep-2013.html) quite
helpful insofar as you were able to implement UI updates without resorting to multithreading via a call to the System.Windows.Forms.Application.DoEvents method. I tried moving my earlier code outside the background worker and replacing each instance of

```csharp
bw.ReportProgress(progress, notification);
```

with

```csharp
reportProgress(progress, notification);
```

where the associated functions are defined as follows:

```csharp
bw.ProgressChanged += new ProgressChangedEventHandler(
delegate(object o, ProgressChangedEventArgs args)
{
bProg.Value = args.ProgressPercentage;
labelPerc.Text = args.ProgressPercentage + "%";
labelLoading.Text = args.UserState.ToString();
});
void reportProgress(int progressvalue, string notification)
{
bProg.Value = progressvalue;
labelPerc.Text = progressvalue + "%";
labelLoading.Text = notification;
System.Windows.Forms.Application.DoEvents();
}
```

And the good news is... with this change my application seems to work perfectly well in Revit 2015!

The only noticeable difference is that I loose UI control within my form while the time-consuming operations are taking place (except for that brief moment when the UI refresh takes place and users can access events like click, resize, drag, et. al.). This isn't a problem for me, as I don't need users to do any UI work while my application is performing these operations. Are you aware of any potential pitfalls of this DoEvents call? If not, I'd say this is a great workaround!

So please be aware that multi-threaded usage might be causing unexpected new exceptions in your code that seemed to work fine in previous versions.

I hope you can appreciate and be grateful that you are now forced to rectify the previous illegal API usage.

Sorry for the additional work this might be causing.

It is for your own good :-)