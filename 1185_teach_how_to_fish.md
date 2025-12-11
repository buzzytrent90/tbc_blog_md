---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.473376'
original_url: https://thebuildingcoder.typepad.com/blog/1185_teach_how_to_fish.html
post_number: '1185'
reading_time_minutes: 5
series: general
slug: teach_how_to_fish
source_file: 1185_teach_how_to_fish.htm
tags:
- references
- revit-api
- rooms
- schedules
- sheets
- views
title: Teaching a Man How To Fish and Schedule Creation
word_count: 1010
---

### Teaching a Man How To Fish and Schedule Creation

As you all know, we at ADN spend a significant amount of our time ensuring that API information is available and easy to find for all.

In order to find something, though, you often need to invest at least a minimum amount of effort in searching.

This can be practiced, of course, and I obtain a useful answer to almost every question I ever have on any subject whatsoever from a single Internet search.

Here are two examples of performing a search for an example of programmatic Revit schedule creation, both desktop and Internet based:

#### Two Ways to Search for a Schedule Creation Example

**Question:** I am importing an Excel sheet into a Revit schedule.

How can I add all the fields from the Excel sheet along with the data to the Revit schedule using the Revit API?

**Answer:** Please excuse me for asking, but this question comes up in my poor little head at almost every query I receive from you: Do you perform any searches for information yourself before submitting a question like this to ADN?

The reason I ask is that I imagine it would save you time, and us as well, of course. We are spending a significant amount of time ensuring that all important API information is publicly available and easily found by the Internet search engines.

Anyway, here is how I approached your question:

1. I looked at the Revit API help file RevitAPI.chm. I did not immediately see an answer.
2. I opened the SDK sample project SDKSamples2015.sln and searched globally for 'schedule'.

Here is the result of the latter, omitting all but the first hit in each project:

```
Find all "schedule", Subfolders, Keep modified files open, Find Results 1, Entire Solution, "*.cs"
  C:\a\lib\revit\2015\SDK\Samples\RoomSchedule\CS\Command.cs(29):namespace Revit.SDK.Samples.RoomSchedule
  C:\a\lib\revit\2015\SDK\Samples\AllViews\CS\AllViews.cs(185):                    if (null == objType || objType.Name.Equals("Schedule")
  C:\a\lib\revit\2015\SDK\Samples\PanelSchedule\CS\CSVTranslator.cs(30):namespace Revit.SDK.Samples.PanelSchedule.CS
  C:\a\lib\revit\2015\SDK\Samples\RoutingPreferenceTools\CS\RoutingPreferenceBuilder\CommandReadPreferences.cs(38):    /// A command to read a routing preference builder xml file and add pipe types, schedules, segments, and sizes, and routing preferences rules to the
  C:\a\lib\revit\2015\SDK\Samples\ScheduleCreation\CS\Command.cs(33):namespace Revit.SDK.Samples.ScheduleCreation.CS
  C:\a\lib\revit\2015\SDK\Samples\PostCommandWorkflow\CS\PostCommandRevisionMonitor.cs(154):        private void ReactToRevisionsAndSchedulesCommand(object sender, BeforeExecutedEventArgs args)
  C:\a\lib\revit\2015\SDK\Samples\ScheduleAutomaticFormatter\CS\Application.cs(37):namespace Revit.SDK.Samples.ScheduleAutomaticFormatter.CS
  C:\a\lib\revit\2015\SDK\Samples\ScheduleToHTML\CS\Application.cs(38):namespace Revit.SDK.Samples.ScheduleToHTML.CS
  C:\a\lib\revit\2015\SDK\Samples\DuplicateViews\CS\Application.cs(79):            pbd2.LongDescription = "Duplicate all duplicatable drafting views and schedules.";
  Matching lines: 323    Matching files: 35    Total files searched: 989
```

As you can see, nine SDK samples out of a total of 173 projects deal with schedules in one way or another:

- RoomSchedule
- AllViews
- PanelSchedule
- RoutingPreferenceTools
- ScheduleCreation
- PostCommandWorkflow
- ScheduleAutomaticFormatter
- ScheduleToHTML
- DuplicateViews

I will leave the rest of this exercise up to you.

As a hint (sorry, I cannot stop myself, I need to check): the entire summary in the readme file of the SDK sample that I recommend you take a look at reads like this:

"**Summary:**  This sample introduces how to create a view schedule and how to show its data on a sheet."

I even tested the sample successfully to make sure it really does what you need using RvtSamples > Views > ScheduleAPI.

The sample makes use of the Revit Schedule API, which was introduced in Revit 2013.

The Building Coder even defines an own category for it: Schedule.

That was the desktop oriented approach.

Another, more modern approach that also yields useful results can be encapsulated in one single line and one single click:

Simply google for
[revit api schedule creation](http://lmgtfy.com/?q=revit+api+schedule+creation).

You know the story about teaching a man how to fish?

I sincerely hope you do not mind me pointing it out to you so directly.

I even wrote about this topic once before, way back in the very beginnings of The Building Coder, more than five years ago, on
[creating a group and how to fish](http://thebuildingcoder.typepad.com/blog/2009/02/creating-a-group-and-how-to-fish.html).

As the Germans say: [Petri Heil](http://dictionary.reverso.net/german-english/Petri%20Heil)!

Piscator non solum piscatur.

#### Addendum

Unbelievably enough, this is still not the end of the story.

Look what came next:

**Question:** In my query I had clearly mentioned that I want to do the task using REVIT API 2013.

You have given me the SDK link for Revit API 2015.

**Answer:** I am aware that you are working with Revit 2013, and pointed that out explicitly in my answer to you.

Do you mean to tell me that you need help finding the ScheduleCreation sample in the Revit 2013 SDK?

Do you have the Revit 2013 SDK installed at all?

I would expect you to have installed this already.

Simply search the Revit SDK for the ScheduleCreation folder.

Better still, simply open it in the Visual Studio SDKSamples2013.sln solution.

If you have not yet installed the SDK, it is available from the Revit API
[getting started material](http://thebuildingcoder.typepad.com/blog/about-the-author.html#2).

Luckily for you, the Revit 2013 SDK is still available for download from the
[Revit Developer Centre](http://www.autodesk.com/developrevit).

Please excuse me for saying so, but the information I am providing here is very basic indeed, and I should really not be wasting either my time or yours spelling it out to you in such detail.

Once again, everything I have said above is immediately available from on single Google search, using the keywords
[revit 2013 sdk download](http://lmgtfy.com/?q=revit+2013+sdk+download).