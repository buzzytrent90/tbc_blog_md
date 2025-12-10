---
post_number: "0752"
title: "Failure Rollback"
slug: "failure_rollback"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'levels', 'revit-api', 'schedules', 'transactions', 'windows']
source_file: "0752_failure_rollback.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0752_failure_rollback.html"
---

### Failure Rollback

I am in full blast preparing and updating material for the
[Revit API training in Munich](http://thebuildingcoder.typepad.com/blog/2012/04/migrating-vsta-macros-to-sharpdevelop.html#1) starting
the day after tomorrow, as well as the upcoming
[AEC DevCamp conference](http://thebuildingcoder.typepad.com/blog/2012/04/devcamp-and-refresh-display-for-a-kinetic-facade.html#1) and
another event that I have not even mentioned here yet:

#### Revit 2013 API Webcast

We will be holding a webcast on the Revit 2013 API on May 17, Ascension Day
([register](http://usa.autodesk.com/adsk/servlet/item?id=10417086&siteID=123112&cname=Revit%20API,%20Webcast,%20May%2017%202012,%20201226)).
A list of all the ADN DevTech training opportunities is provided by the
[Training Course Schedule](http://www.adskconsulting.com/adn/cs/api_course_sched.php),
entering "Revit API" as the course topic.
I'll write something about the webcast topics anon.

Another important piece of news from the ADN team is the launch of a new series of blogs:

#### ADN DevBlogs

The Autodesk Developer Network ADN team started the launch of a series of new blogs, marking a significant shift from material that's only available to ADN members, like technical solutions known as DevNotes on the ADN extranet, to creating material that's publicly available to everyone.
The first two are up and running now:

- [AutoCAD DevBlog](http://adndevblog.typepad.com/autocad)- [Infrastructure Modeling DevBlog](http://adndevblog.typepad.com/infrastructure)

More ADN DevBlogs for AEC, Manufacturing and Media & Entertainment will coming soon.
For more information, please refer to
[Stephen Preston's welcome post](http://adndevblog.typepad.com/autocad/2012/02/welcome.html).

Back to the nuts and bolts of the Revit API, here is a case brought up and eventually solved by Mario Guttman of
[Perkins+Will](http://www.perkinswill.com):

#### Roll Back Failure Suppressing all Messages, both Warnings and Errors

**Question:** I am trying to implement a failure handler that will silently respond to all warnings.
I am happy to roll back the transaction or delete the elements but I can't seem to get rid of the dialogue box with failures other than warnings.

**Answer:** I figured it out.

In case anyone is interested, I needed an additional statement in the FailureHandler class:
```csharp
return FailureProcessingResult
.ProceedWithRollBack;
```

I also needed another statement with the transaction start:
```csharp
failureHandlingOptions
.SetClearAfterRollback( true );
```

Here is a more complete description and solution:

I'm running a program that creates a lot of new Revit elements.
The process may fail in the middle for some reason.
I don't want the user to have to respond to a dialogue each time; I just want the program to keep going with the next element.

Specifically, if it is a warning, I want to ignore it, and if it is any other kind of error, I want to cancel creating that element and continue with the next one.

I do want to record the errors so I can inform the user at the end about what problems were encountered.

I looked at your posting on the
[Failure API](http://thebuildingcoder.typepad.com/blog/2010/04/failure-api.html ) about
how to handle failure messages, and also the Revit 2012 API Developer Guide and sample code, but didn't quite find what I was looking for.

I finally figured it out and thought you might like to share this:

To begin with, you need to create a FailureHandler class; mine looks like this:
```csharp
public class FailureHandler : IFailuresPreprocessor
{
  public string ErrorMessage { set; get; }
  public string ErrorSeverity { set; get; }

  public FailureHandler()
  {
    ErrorMessage = "";
    ErrorSeverity = "";
  }

  public FailureProcessingResult PreprocessFailures(
    FailuresAccessor failuresAccessor )
  {
    IList<FailureMessageAccessor> failureMessages
      = failuresAccessor.GetFailureMessages();

    foreach( FailureMessageAccessor
      failureMessageAccessor in failureMessages )
    {
      // We're just deleting all of the warning level
      // failures and rolling back any others

      FailureDefinitionId id = failureMessageAccessor
        .GetFailureDefinitionId();

      try
      {
        ErrorMessage = failureMessageAccessor
          .GetDescriptionText();
      }
      catch
      {
        ErrorMessage = "Unknown Error";
      }

      try
      {
        FailureSeverity failureSeverity
          = failureMessageAccessor.GetSeverity();

        ErrorSeverity = failureSeverity.ToString();

        if( failureSeverity == FailureSeverity.Warning )
        {
          failuresAccessor.DeleteWarning(
            failureMessageAccessor );
        }
        else
        {
          return FailureProcessingResult
            .ProceedWithRollBack;
        }
      }
      catch
      {
      }
    }
    return FailureProcessingResult.Continue;
  }
}
```

Then, in the body of the main program you need something like:
```csharp
  Transaction transaction = new Transaction( doc );

  FailureHandlingOptions failureHandlingOptions
    = transaction.GetFailureHandlingOptions();

  FailureHandler failureHandler
    = new FailureHandler();

  failureHandlingOptions.SetFailuresPreprocessor(
    failureHandler );

  failureHandlingOptions.SetClearAfterRollback(
    true );

  transaction.SetFailureHandlingOptions(
    failureHandlingOptions );

  transaction.Start( "Transaction Name" );

  // Do something here that causes the error

  transaction.Commit();

  // The following is just illustrative.
  // In reality we would collect the errors
  // to show later.

  if( failureHandler.ErrorMessage != "" )
  {
    System.Windows.Forms.MessageBox.Show(
      failureHandler.ErrorSeverity + " || "
      + failureHandler.ErrorMessage );
  }
```

Many thanks to Mario for all his research and friendly sharing!