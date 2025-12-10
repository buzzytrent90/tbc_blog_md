---
post_number: "0531"
title: "Command and Conquer When Switching Views"
slug: "commands_and_views"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'levels', 'parameters', 'revit-api', 'schedules', 'transactions', 'views', 'windows']
source_file: "0531_commands_and_views.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0531_commands_and_views.html"
---

### Command and Conquer When Switching Views

Today, February 3, is the first day of the
[Chinese New Year](http://en.wikipedia.org/wiki/Chinese_New_Year) and the beginning of the spring festival, which lasts for fifteen days.
This is a year of the
[Rabbit zodiac sign](http://en.wikipedia.org/wiki/Rabbit_%28zodiac%29).
Here is a
[duilian](http://en.wikipedia.org/wiki/Chunlian),
a pair of lines of poetry, or actually a special form called
[chunlian](http://en.wikipedia.org/wiki/Chunlian), used
as a New Year's decoration that expresses happy and hopeful thoughts for the coming year sent us by Joe Ye from Beijing:

![Year of the rabbit chunlian](img/year_of_the_rabbit_1.png)

The meanings of these lines are:

- Top: Lucky Star is shining.- Left: Good family, good person, and good luck.- Right: Abundant happiness, abundant fortune, and prosperous day.

Continuing our exploration of ways to automate Revit, Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de) recently
presented his results of exploring the Revit ribbon internals using UISpy,
[driving Revit using UI Automation](http://thebuildingcoder.typepad.com/blog/2011/01/ribbon-spying-and-ui-automation.html),
[subscribing to UI Automation events](http://thebuildingcoder.typepad.com/blog/2011/01/subscribing-to-ui-automation-events.html), and
[automating Revit window switching](http://thebuildingcoder.typepad.com/blog/2011/01/further-ideas-for-using-ui-automation.html).
It is fascinating to be able to drive Revit pretty reliably from outside by simulating the user interaction with the ribbon and other user interface elements.

Here is the source code and Visual Studio project for the sample application
[DrivingRevitViaUIAutomation](zip/DrivingRevitViaUiAutomation.zip) that
Rudolf used to implement the following three functionalities from a
[stand-alone executable](http://thebuildingcoder.typepad.com/blog/2011/01/ribbon-spying-and-ui-automation.html#app):

- Open a Revit document.- Close the active document with or without saving it.- Switch the current ribbon tab.

![UI automation sample application with populated ribbon tabs](img/rh_ui_automation_sample_populated.png)

He continues with some more thoughts on setting active windows and views:

I suppose that setting the active view via GUI cannot be performed on command level, so it must be performed at application level.
Just imagine this situation:
First, you press your button to call your command.
Then, this command changes the view or the document.
If your command changes your document, your transaction (which is per document – also the undo stack is per document, of course) may become invalid.
In Revit, changing the view affects the availability of the 'ADD\_INS\_TAB'.

In addition, it makes sense to invalidate the command context after switching the active view, situations can occur where the new view is of ViewType.Schedule.
In this sort of view, neither can external commands be executed nor will the OnIdling event be fired.

So, how can we perform this, though?

I suggest an approach which invokes a command twice, in fact.

The idea is to provide an OnIdling event handler that invokes commands after setting the new active view or document.
For example, first you invoke your command.
In this command, you have two possibilities.
If the active view (and/or the active document) is right, do the work as usual.
If not, activate the OnIdling event and fill a global variable of type 'AutomationElement' with your corresponding command button.
Next time Revit idles, this button will be invoked, so your command will be performed in the right document and view, without violating the transaction mechanism.

After this, the global variable can be set to null and the OnIdling event handler can be deactivated again.

It would of course also be possible to let the event remain alive all the time while Revit is running, but why waste performance?

Another point:

Invoking commands can be performed in other ways than by pressing buttons.
Using the IExternalCommand interface just means that an Execute method with the appropriate signature must be implemented:
```csharp
public Result Execute(
  ExternalCommandData revit,
  ref String message,
  ElementSet elements )
{
  return Result.Succeeded;
}
```

You could just as well define a placeholder Execute method and call another method with different parameters from it, such as this:
```csharp
public Result Execute( UIApplication revit )
{
  // do some useful stuff here…

  return Result.Succeeded;
}
```

It could be invoked like this:
```csharp
void application\_Idling(
  object sender,
  IdlingEventArgs e )
{
  Application app = sender as Application;

  UIApplication uiApp = new UIApplication( app );

  Command\_Example cmdExample = new Command\_Example();

  cmdExample.Execute( uiApp ); // invoke command here
}
```

So, instead of a global 'AutomationElement' for the next button to be pressed, another type could be chosen, and no button needs to be pressed, which may be faster, by the way.
It may be necessary to set a pause between ending the first command and invoking the second one.
To be continued...

![Happy New Year of the Rabbit](img/year_of_the_rabbit_2.png)

Happy New Year of the Rabbit!

**Addendum:** Arnošt Löbel adds that an alternative and simpler solution would be to subscribe to the ViewActivated event instead of Idling.
This event is much better suited for this particular problem.
In addition, it is much less of a performance hit, because ViewActivated is raised only when a view is actually activated, while Idling can be raised several times per second.
Even if the handler does nothing at all, simply the fact that the code needs to go from native to managed and back to native again means a significant slowdown.