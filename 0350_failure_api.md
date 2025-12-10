---
post_number: "0350"
title: "Failure API"
slug: "failure_api"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'python', 'revit-api', 'rooms', 'transactions', 'views', 'walls']
source_file: "0350_failure_api.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0350_failure_api.html"
---

### Failure API

A very frequent question from developers in the past has been about suppressing various warning and error messages in Revit.
One of the many
[new Revit 2011 API features](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-is-coming.html) specifically
targeted at allowing an add-in to integrate very tightly with Revit and its user interface is the new Failure API, providing the ability for an add-in to react to and suppress failures as well as define its own new ones and post them.
This is the description in the What's New section of the Revit API help file RevitAPI.chm:

#### Failure API

There are two capabilities offered by the new failure API:

1. The ability to define and post failures from within API code when a user-visible problem has occurred.- The ability to respond to failures posted by Revit and by API code through code in your application.

This section offers a high level overview of both capabilities; more detail about the failures API is provided in the "Failure API" document in the Revit API help file.

As a part of exposing these new capabilities, all overloads accepting "PostedErrors" have been removed from the API.

#### Failure posting

If you are using the failure posting mechanism to report your problem, all you need to do is:

1. If you are creating a new failure not already existing in Revit, define the new failure and register it in the FailureDefinitionRegistry during the OnStartup() call of your ExternalApplication (new failures must be registered at Revit startup).- Find the failure definition id, either from the BuiltInFailures classes or from your pre-registered custom failure using the class related to FailureDefinition.- Post a failure to a document that has a problem - using the classes related to FailureMessage to set options and details related to the failure.

#### Failure handling

Normally posted failures are processed by Revit's standard failure resolution UI at the end of transaction. The user is presented information and options in the UI to deal with the failures.

However, if your operation (or set of operations) on the document requires some special treatment for certain errors (or even all possible errors), you can customize failure resolution. Custom failure resolution can be supplied:

- For a given transaction using the interface IFailuresPreprocessor.- For all possible errors using the FailuresProcessing event.

Finally, the API offers the ability to completely replace the standard failure processing user interface using the interface IFailuresProcessor.

#### ErrorHandling SDK Sample

The use of the new failure API is demonstrated very comprehensively by the ErrorHandling Revit SDK sample, which shows how to create a failure definition id, failure definition, failure message and how to resolve failures in failure processing steps.
It suppresses a warning message for overlapping walls.
For more detailed information on that sample implementation, please refer to its read-me document ReadMe\_ErrorHandling.rtf.

This sample is quite extensive and demonstrates many aspects of the failure API.
When looking at it, one of my colleagues came up with some questions requesting clarification of its expected behaviour:

1. The command creates two overlapping walls without showing a warning message, which is clearly intended.- After the command is executed, manually creating walls also get no warning. Is this also expected? If so, do we need an additional command in order to restore the original UI behaviour? Just wondering whether this is intentional or the sample only shows how to set.- The sample creates several different walls in various steps. What does the second wall between (0, 10, 0) and (20, 10, 0) demonstrate? Is it intended to show how to delete it after regeneration?- Also, I kind of expected to see an error dialogue with some information. Am I missing something?

Leo Lu, the author of the sample, responded to these questions:

1. Creating two overlapping walls without a warning message is only a small part of the sample.
   It also demonstrates how to create failure definition id, failure definitions, and failure messages.
   These parts are basic.
   It demonstrates that there are different ways to handle the warnings and errors: in the FailuresPreprocessor, FailuresProcessing event or FailuresProcessor.
   Please also refer to the detailed notes in the sample's readme file.
   These parts are more advanced and tricky:
   - FailuresPreprocessor is being set for one transaction and used only during finishing of this one transaction.- Once the FailuresProcessing event is subscribed to in line 195 of the code, it will be raised whenever it is needed.
       You can unsubscribe from it with a -= operation so that the event won't be raised any more.- FailuresProcessor can be set up via a method of the Revit add-in.
         If a new FailuresProcessor is set, any previously set one is completely removed.
         Once you set a FailuresProcessor, it will remain alive for the whole Revit session.- The behaviour is not expected if the sample only wants to solve the overlapping walls problem.
     Then a FailuresPreprocessor for a single transaction can solve this problem – when the external command is over, you'll get back to original UI behaviour immediately.
     The sample also uses FailuresProcessing and FailuresProcessor which remains alive after the external command to handle other warnings or errors.
     When you draw some overlapping walls in the UI after the external command has returned, the warnings will be handled by the failures processor.
     The solution is simple – just remove the codes which use FailuresProcessing and FailuresProcessor, then the sample won't affect the normal UI behaviour any longer.- The overlapping walls problem is just an example we chose to demonstrate how to handle a warning or error in the FailuresPreprocessor.
       Of course one can do other things that may cause other expected warnings or errors that can also be handled in the FailuresPreprocessor similarly to the sample.
       Use FailuresPreprocessor rather than FailuresProcessing or FailuresProcessor in this condition so that the codes will not affect the Revit UI behaviour.- Yes, that's an option.
         Or you can choose to suppress some specified warnings you do not care in some projects.
         Or you may want to quietly roll back all transactions if any error arises.
         That is all completely up to you.

Here is another example of using the failure API which is less extensive and therefore much smaller and simpler:

#### Minimal Room Warning Swallower Failure Processing Sample

Harry Mattison of the Revit development team provided this small and sweet sample of using the failure API to suppress a specific warning message.
It includes the RoomWarningSwallower IFailuresPreprocessor implementation used as sample code for the IFailuresPreprocessor interface in the Revit API help file.

The command creates an unbounded room and suppresses the warning that would otherwise be given saying "Room is not in a properly enclosed region".

The duration for this implementation is only for the transaction in the external command, so after the command is executed manually placed unbounded rooms do result in the warning again a usual.

However, as mentioned above, it is also possible with the new failure API to suppress warnings for the entire Revit session.

Here is the implementation of the warning swallower class implementing the IFailuresPreprocessor interface:
```python
public class RoomWarningSwallower : IFailuresPreprocessor
{
  public FailureProcessingResult PreprocessFailures(
    FailuresAccessor a )
  {
    // inside event handler, get all warnings

    IList<FailureMessageAccessor> failures
      = a.GetFailureMessages();

    foreach( FailureMessageAccessor f in failures )
    {
      // check failure definition ids
      // against ones to dismiss:

      FailureDefinitionId id
        = f.GetFailureDefinitionId();

      if( BuiltInFailures.RoomFailures.RoomNotEnclosed
        == id )
      {
        a.DeleteWarning( f );
      }
    }
    return FailureProcessingResult.Continue;
  }
}
```

This is the mainline of the external command Execute method, which performs the following steps:

- Determine an arbitrary level to use to place the room on.- Start a transaction, since we are using manual transaction mode.- Set up the room warning swallower failures pre-processor.- Create the unbounded room.

```csharp
Document doc = commandData.Application
  .ActiveUIDocument.Document;

FilteredElementCollector collector
  = new FilteredElementCollector( doc );

collector.OfClass( typeof( Level ) );
Level level = collector.FirstElement() as Level;

Transaction t = new Transaction( doc );

t.Start( "Create unbounded room" );

FailureHandlingOptions failOpt
  = t.GetFailureHandlingOptions();

failOpt.SetFailuresPreprocessor(
  new RoomWarningSwallower() );

t.SetFailureHandlingOptions( failOpt );

doc.Create.NewRoom( level, new UV( 0, 0 ) );

t.Commit();

return Result.Succeeded;
```

Many thanks to Harry for this nice little sample and to Leo for the extensive explanation of the ErrorHandling behaviour!

I have included the former as a new module and external command named CmdPreprocessFailure in The Building Coder samples.
Here is
[version 2011.0.0.64](zip/bc_11_64.zip)
of the complete Visual Studio solution including the new command.