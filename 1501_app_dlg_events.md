---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.7
content_type: qa
optimization_date: '2025-12-11T11:44:16.170620'
original_url: https://thebuildingcoder.typepad.com/blog/1501_app_dlg_events.html
post_number: '1501'
reading_time_minutes: 7
series: general
slug: app_dlg_events
source_file: 1501_app_dlg_events.md
tags:
- elements
- parameters
- revit-api
- sheets
title: App Dlg Events
word_count: 1371
---

### Using Other Events to Execute Add-In Code
The Revit API is very simple.
It is entirely event driven.
Any and every use of the Revit API requires a valid API context.
The most common and obvious way to get into a valid Revit API context – and the most commonly used event – is the one to launch an external command, which calls the `IExternalCommand` `Execute` handler method.
Other important ones to be aware of are `ApplicationInitialized` and `DialogBoxShowing`, unconnected with any external command at all, as you can see below, and the method described by David Echols to [load and execute assembly code on the fly](#2) in an `Idling` event handler by subscribing to that event in the application `OnStartup` method.
Here is an interesting discussion exploring some options from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
searching for an [event to run an IExternalCommand](http://forums.autodesk.com/t5/revit-api-forum/event-to-run-a-iexternalcommand/m-p/6692479):
\*\*Question:\*\* Wondering if you can assist with an issue I'm having that I just can't seem to solve.
Would be nice just to know if what I am trying to do is possible or not.
I have been playing around with the events within an `ExternalApplication` and I have got these to work fine (i.e. using the `DocumentCreated` event to flash up a "HelloWorld" task dialog when a document is created)
I have also written a few basic external commands with buttons on the addin ribbon that work fine.
My issue is: how do I get an event to fire off an external command instead of having to use a button?
I am basically trying to get the event to trigger off the command and make some changes to the document and so I need the current document which I don't believe I can access within the external application.
I have read all of the event samples in the SDK but still can't work it out and I'm sure it should be really simple.
\*\*Answer by Matt:\*\* I'm pretty sure that's what [PostCommand](http://www.revitapidocs.com/2017/b0df464d-1733-ea9e-ac40-399fa9c9a037.htm) is for.
\*\*Answer by Erik:\*\* Matt's response is correct and valid for all buttons in Revit (almost), but if you created the external command, you can just create an instance of the class and run the `Execute` method. That's even simpler.
\*\*Response:\*\* I am trying to do what you suggest, "just create an instance of the class and run the `Execute` method", but that's where I am getting stuck.
In my external app I am trying to execute the command using
```csharp
Command cmd = new Command();
cmd.Execute();
```
(Editor's note on Typepad crash: I was forced to replace the character 'c' in the last line above by the HTML escape code `c` before I was able to post this.)
However, that understandably gives me the error

```
Error CS7036 There is no argument given that corresponds to the required formal parameter 'commandData' of 'Command.Execute(ExternalCommandData, ref string, ElementSet)'
```

Could you please expand on how I would use the `Execute` method from an external application?
\*\*Answer by Gonçalo:\*\* Mike, you need a handler for the event, and the `IExternaCommand.Execute` method does not have the same parameters.
You can fake the parameters (commanddata, etc.) using reflection and an adapter; but that seems awkward.
My suggestion is just to implement a method to handle the event you are after. After all, the sender is usually the application.
\*\*Answer by Matt:\*\* I think Erik was suggesting that if you wrote the external command, you could call the same function that you call in the external command.
`PostCommand` would be the best fit, if you can apply it. Why don't you tell us exactly what you're trying to do?
\*\*Answer by Erik:\*\* Maybe I read your message a bit fast. I missed the fact that you were in an external application context, I automatically thought you were in `IExternalCommand`; then you could just pass the `ExternalCommandData` object forward.
But to solve your problem you could just refactor your code a bit to get an overload for the `Execute` method that takes a `UIApplication` object. The `Execute` method in the `IExternalCommand` interface seems a bit obsolete anyway.
Also, another thing to bear in mind here is that you should subscribe to the `ApplicationInitialized` event if you are to do anything with documents.
Here is an example code snippet:
```csharp
public class Command : IExternalCommand
{
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
return Execute( commandData.Application );
}
public Result Execute( UIApplication uiapp )
{
//Do all sorts of shiny stuff with your command.
return Result.Succeeded;
}
}
public class RevitStartup : IExternalApplication
{
public Result OnShutdown(
UIControlledApplication application )
{
return Result.Succeeded;
}
public Result OnStartup(
UIControlledApplication application )
{
application.ControlledApplication
.ApplicationInitialized
+= ControlledApplication_ApplicationInitialized;
return Result.Succeeded;
}
private void ControlledApplication_ApplicationInitialized(
object sender,
ApplicationInitializedEventArgs e )
{
var command = new Command();
//I never remember if the sender is Application or UIApplication
if( sender is UIApplication )
command.Execute( sender as UIApplication );
else
command.Execute( new UIApplication( sender as Application ) );
}
}
```
This will also work even if you don't register a button.
And to finish off, like Mike said: tell us why you want to achieve and maybe we can find an even easier way of doing it =)
\*\*Response:\*\* Many, many thanks for all your replies.
Although none of them answered the question they did help me realise that what I was trying to do was the wrong way to approach things and that I actually didn't need the external command.
My reason for thinking I needed the external command was trying to get the `Document` but thanks to Erik's post I found I could get the `UIApplication` from the sender and that this then let me access the document. So now I am able to run all of my code in the external application!
Again, many thanks for all your assistance.
To finalise, my end result was to register a dialog event on startup and shutdown and then:
```csharp
public void handle_DialogBoxShowing(
object sender,
DialogBoxShowingEventArgs args )
{
if( args.DialogId == "Dialog_Revit_NewProject" )
{
UIApplication uiapp = sender as UIApplication;
Application app = uiapp.Application;
// etc etc etc
```
Many thanks to Mike, Matt, Erik and Gonçalo for raising and helping to solve this!
![Apples](img/131_apples_500x375.jpg)
#### Addendum – Load and Execute Code on the Fly
In the [comment below](http://thebuildingcoder.typepad.com/blog/2016/11/using-other-events-to-execute-add-in-code.html#comment-3039130901),
David Echols explains how he can load and execute assembly code on the fly by subscribing to the `Idling` event in the application `OnStartup` method:
> I have used the following code in the last four versions of Revit for overnight batch processing of PDF and DWG exports and sheet lists across multiple workstations. This is not the most elegant way to do this, but it does show how I load the assembly on the fly and execute the command that does the processing.
```csharp
// The event is added in IExternalApplication.OnStartup.
// Note: The handler is unloaded on the first line in
// the handler. The event is started again in the
// DocumentClosed event handler when the current
// document is closed.
public static void RemoteJobHandler(
object sender,
IdlingEventArgs e )
{
GlobalSettings.RevitUiControlledApp.Idling
-= IdlingHandler.RemoteJobHandler;
LogManager.LogMessage(
"IdlingHandler.RemoteJobHandler - Enter handler",
LogSeverityType.Debug );
Job job = new Job( GlobalSettings.RemoteJobXmlData );
string dllLocation = job.DllFileName;
string className = job.CommandName;
try
{
// do a bunch of stuff here to setup
ModelPath modelPath = ModelPathUtils
.ConvertUserVisiblePathToModelPath(
item.RevitFileName );
uiDocument = GlobalSettings.RevitUiApp
.OpenAndActivateDocument( modelPath,
openOptions, false );
Type type = LoadDllWithReflection( dllLocation, className );
IRemoteCommand command = GetCommandFromType( type );
command.RunRemotely( uiDocument.Document, item );
}
catch( Exception ex )
{
// handle/log errors
}
finally
{
// cleanup
}
}
public static Type LoadDllWithReflection(
string dllLocation,
string className )
{
try
{
Assembly assembly = Assembly.LoadFrom( dllLocation );
Type type = assembly.GetType( className );
return type;
}
catch( ReflectionTypeLoadException ex )
{
var exceptions = ex.LoaderExceptions;
foreach( var failedType in ex.Types )
{
if( failedType != null )
Debug.WriteLine( failedType.FullName );
}
foreach( Exception loadException in exceptions )
{
Debug.WriteLine( loadException.ToString() );
}
}
return null;
}
public static IRemoteCommand GetCommandFromType( Type type )
{
IRemoteCommand command = Activator.CreateInstance( type )
as IRemoteCommand;
if( command == null )
th row new Exception( "Could not get Remote Command" );
return command;
}
```