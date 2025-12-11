---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.2
content_type: qa
optimization_date: '2025-12-11T11:44:17.392559'
original_url: https://thebuildingcoder.typepad.com/blog/2061_unittest_viewports.html
post_number: '2061'
reading_time_minutes: 16
series: views
slug: unittest_viewports
source_file: 2061_unittest_viewports.md
tags:
- elements
- family
- filtering
- parameters
- python
- references
- revit-api
- sheets
- transactions
- views
- walls
title: Unittest Viewports
word_count: 3128
---

### Back Again to Unit Test, Icons, Viewports and More
Long time no see! Happy New Year. Here I am again with a host of stuff:
- [Happy New Year!](#2)
- [Back again and winding up](#3)
- [Multi-version NUnit testing framework](#4)
- [Revit icons support high-resolution display](#5)
- [Viewports, view types and title types](#6)
- [Clarifying viewport, view type, and title type](#6.1)
- [Goal: change the viewport type](#6.2)
- [1. Retrieve viewport types programmatically](#6.3)
- [2. Update viewport types](#6.4)
- [Key insights](#6.5)
- [Correct diagnostic code](#6.6)
- [Dynamo users](#6.7)
- [Practical considerations](#6.8)
- [Summary of steps](#6.9)
- [Additional insights](#6.10)
- [Interact with BIM via ChatGPT interface in Revit](#7)
- [Expertise in real world contexts](#8)
- [Humans versus LLMs completing ARC-AGI](#9)
- [2024 AI breakthroughs & controversies](#10)
- [LLM transformers explained](#11)
- [Short movie by Veo 2](#12)
- [LangChain state of AI 2024 report](#13)
- [Current AIs are not politically neutral](#14)
- [Mind-reading potential of AI](#15)
- [Iterative spell check with Gemini](#16)
#### Happy New Year!
Happy New Year to all the brilliant minds in the Revit API developer community!
Wishing you a year filled with joy, success in all your endeavors, and may your coding adventures be fruitful and rewarding.
Peace and happiness to all.
Unfortunately, in the past few months, I keep being reminded of the so-called Chinese curse:
[May you live in interesting times](https://en.wikipedia.org/wiki/May_you_live_in_interesting_times).
In fact, it appears that there is no known Chinese equivalent; the closest is "Better to be a dog in times of tranquility than a human in times of chaos", 寧為太平犬，不做亂世人.
Taking a completely opposite view, surveying an apparently collapsing China, Mao Zedong coolly observed that there was “great disorder under the Heavens and the situation is excellent.”
Interesting times indeed.
#### Back Again and Winding Up
As I mentioned in November,
[my retirement is nearing](https://thebuildingcoder.typepad.com/blog/2024/11/devcon-ai-for-revit-api-modeless-add-ins-leave.html#3)
and I took time off in the past two months.
I am back again now and thinking about how to wind up my activities before retiring towards the middle of the year.
Meanwhile, I am mesmerised by the advancements in AI and LLMs.
I am confident that already today, most of the repetitive questions that keep coming up in the Revit API forum can be answered by proper queries to one of the LLMs.
Today, I would like to highlight three new non-repetitive threads from the past week here, and also list a few of the AI-related titbits that caught my attention.
#### Multi-Version NUnit Testing Framework
Luiz Henrique [@ricaun](https://ricaun.com/) Cassettari shared
his [multi-version NUnit testing framework for Revit API](https://forums.autodesk.com/t5/revit-api-forum/multi-version-nunit-testing-framework-for-revit-api/m-p/13230523), saying:
Hello everyone, after some years of development my Test framework for Revit API is released.
[ricaun.RevitTest](https://www.youtube.com/watch?v=_lsmvQtGftE) is a multi-version `NUnit` testing framework for Revit API.
It supports all releases from Revit 2019 up to Revit 2025.
Features:
- Run tests and debug using Visual Studio to execute tests inside Revit.
- Open and Close Revit and dotnet test execution.
- Auto detect Revit installation and no manual plugin installation required.
The whole project is open-source with all the utility packages.
Here is his 34-minute video on [Open-Source RevitTest](https://youtu.be/_lsmvQtGftE):

The best way to start is by downloading the sample [RevitTest project on GitHub](https://github.com/ricaun-io/RevitTest).
Please use the [RevitTest discussions area](https://github.com/ricaun-io/RevitTest/discussions) for help and discussion.
#### Revit Icons Support High-Resolution Display
[@ricaun](https://ricaun.com/) also shared the results of his research
on [Revit icons support high-resolution display](https://forums.autodesk.com/t5/revit-api-forum/revit-icons-support-high-resolution-display/m-p/13238444#M83163):
Unlike AutoCAD, the Revit UI itself does not provide this support, but Revit add-ins can do so, using TIFF files to allow multiple resolutions of the same image in a single file, exactly as the Autodesk Icon Design Guidelines suggest, e.g., 5 DPI png images (100%, 150%, 200% 300% and 400%) to support clear displays on high-resolution monitors.
- [Icons in AutoCAD](https://forums.autodesk.com/t5/autocad-forum/looking-for-icon-definitions/m-p/13136339/highlight/true#M1141560)
- [ricaun.Revit.UI library](https://github.com/ricaun-io/ricaun.Revit.UI) enhanced to support TIFF and select the best high-resolution monitor image
- [Autodesk.Icon.Example project](https://github.com/ricaun-io/Autodesk.Icon.Example) updated with TIFF files
Next step: update all add-ins to use TIFF files instead of ICO files to support high-resolution monitors, cf. the ten-minute video explaining how
to [create multi-dpi `tiff` icons for Revit](https://youtu.be/3JrIW3UW5Ws):

#### Viewports, View Types and Title Types
Another great forum contribution comes from
Brian [@BriCircuitBIM](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/7975487) Miller,
explaining the relationships between viewports, view types, and title types in order
to [change viewport type via API](https://forums.autodesk.com/t5/revit-api-forum/change-viewport-type-via-api/m-p/13234218#M83138).
Here is his full explanation:
#### Clarifying Viewport, View Type, and Title Type
- Viewport Type –
Each viewport placed on a sheet is an instance of a Viewport system family.
Viewports have a type ID, which refers to the viewport type (e.g., "Title, No Scale, Line").
These types define how the title is displayed, including parameters like line visibility and scale display.
- View Type – The view type (ViewType.FloorPlan, ViewType.Legend, etc.) is a property of the view associated with the viewport. It describes the kind of view (floor plan, legend, section, etc.) but is unrelated to the viewport's title or appearance.
- Title Type – The title type (referenced via the viewport's ELEM_TYPE_PARAM) is what controls the viewport's title behavior and visual style. This is the parameter you're trying to modify programmatically.
#### Goal: Change the Viewport Type
If your goal is to change the viewport type based on the view title name (e.g., "Title, No Scale, Line"), here's how you can proceed:
#### 1. Retrieve Viewport Types Programmatically
The key is to work with `ElementType` objects within the `OST_ViewportLabel` category, which represents viewport title types:

```
// Retrieve all viewport types (family symbols) in the project
FilteredElementCollector collector = new FilteredElementCollector(doc)
  .OfClass(typeof(FamilySymbol))
  .OfCategory(BuiltInCategory.OST_ViewportLabel);

string targetViewportTypeName = "Title, No Scale, Line";
FamilySymbol targetViewportType = collector
  .Cast()
  .FirstOrDefault(fs => fs.Name.Equals(targetViewportTypeName, StringComparison.OrdinalIgnoreCase));

if (targetViewportType == null)
{
  TaskDialog.Show("Error", "The specified viewport type was not found.");
  return;
}
```

#### 2. Update Viewport Types
Once you've identified the desired viewport type, you can update specific viewports programmatically using the `Viewport.ChangeTypeId` method:

```
FilteredElementCollector viewportCollector = new FilteredElementCollector(doc)
  .OfClass(typeof(Viewport));

using (Transaction trans = new Transaction(doc, "Change Viewport Types"))
{
  try
  {
    trans.Start();

    foreach (Viewport viewport in viewportCollector.Cast())
    {
      Autodesk.Revit.DB.View view = doc.GetElement(viewport.ViewId) as Autodesk.Revit.DB.View;

      // Example: Update only viewports associated with Drafting Views
      if (view != null && view.ViewType == ViewType.DraftingView)
      {
        viewport.ChangeTypeId(targetViewportType.Id);
      }
    }

    trans.Commit();
  }
  catch (Exception ex)
  {
    trans.RollBack();
    TaskDialog.Show("Error", $"An error occurred: {ex.Message}");
  }
}
```

#### Key Insights
Key insights for the API limitations why your diagnostic code returns nothing:
Your initial code attempts to retrieve viewport types by filtering with .OfClass(typeof(View)).
However:
- Viewport types are `ElementType` objects under the `OST_ViewportLabel` category.
- Using OfClass(typeof(View)) will not return these types since it filters for views, not viewport types.
#### Correct Diagnostic Code
Here's a diagnostic script to list all available viewport types by name:

```
FilteredElementCollector collector = new FilteredElementCollector(doc)
  .OfClass(typeof(FamilySymbol))
  .OfCategory(BuiltInCategory.OST_ViewportLabel);

StringBuilder viewportTypeNames = new StringBuilder("Available Viewport Types:\n");
foreach (FamilySymbol type in collector.Cast())
{
  viewportTypeNames.AppendLine(type.Name);
}

TaskDialog.Show("Viewport Types", viewportTypeNames.ToString());
```

#### Dynamo Users
For users working in Dynamo:
- Viewport Types are stored as FamilySymbol elements within the OST_ViewportLabel category.
- Use a combination of FamilySymbol.Name and Viewport.ChangeTypeId (if scripting with Python/DesignScript) to identify and update viewport types programmatically.
#### Practical Considerations
- Revit API Version –
Ensure compatibility between your code and the Revit version you're using. Revit 2024 introduced enhanced sub-element handling for viewports, but these changes are not available in earlier versions.
- Scale Display –
If you're filtering by scale (e.g., "NTS"), retrieve the Scale parameter of the associated view and use it to identify relevant viewports.
- Error Handling –
Always validate the ViewId, TypeId, and compatibility of the viewport type before applying changes.
#### Summary of Steps
- Retrieve Viewport Types:
Use FilteredElementCollector to find all viewport types in the OST_ViewportLabel category.
- Filter by Target Name:
Identify the desired viewport type by its name (e.g., "Title, No Scale, Line").
- Update Viewport Types:
Use Viewport.ChangeTypeId() to apply the target type to specific viewports.
- Diagnostic Code:
Use the provided diagnostic code to list available viewport types and verify your settings.
#### Additional Insights
Sometimes, my NLU (Natural Language Understanding) system doesn't interpret things perfectly, so please feel free to let me know if any part of this explanation is unclear or inaccurate.
This response stems from my own challenges when attempting to programmatically change legend viewport titles in Revit.
After spending two days meticulously experimenting, I concluded that it is not possible to change a legend viewport type unless that type already exists in the project.
The breakthrough came when I shifted my focus to identifying viewport types that were already present in the project.
By filtering and listing these existing types, I was able to determine which one to use as the replacement.
For anyone facing similar struggles, I highly recommend starting with a diagnostic approach: gather all available viewport types using the `FilteredElementCollector` and confirm their names match your desired title type.
This ensures you're working within the constraints of your project's current configuration.
Thank you very much, Brian, for documenting and explaining this!
Moustafa Khalil of [SharpBIM](https://hashnode.com/@SharpBIM) ([GitHub](https://github.com/mostafa901)) adds:
Fetching Viewport Types is a bit tricky.
I wrote an article that might help about [purging viewport types](https://sharpbim.hashnode.dev/purging-viewport-types), i.e., deleting the unused ones.
In general, the process of identifying elements like Viewport Types without a defined class in Revit is challenging.
`ViewportType` is categorized as an `ElementType`, which is too broad to be helpful.
An alternative approach is to indirectly identify purgeable Viewport Types by targeting parameters.
Since `ViewportType` is a system family, all instances must have the `Show Extension Line` parameter, stored in the `BuiltInParameter` `VIEWPORT_ATTR_SHOW_EXTENSION_LINE`.
This Boolean parameter (0 or 1) can be used in a filter rule.
By applying the `ParameterFilterRuleFactory` `CreateGreaterOrEqualRule` with a value of 0, all types possessing this parameter can be identified.
You can try this code to change the viewport type and read more about this code
in [this article](https://sharpbim.hashnode.dev/purging-viewport-types):

```
// create filterRule to be used in collection
var rule = ParameterFilterRuleFactory.CreateGreaterOrEqualRule(
  new ElementId(BuiltInParameter.VIEWPORT_ATTR_SHOW_EXTENSION_LINE),
  0
);
// define which type name you are looking for
var nameRule = ParameterFilterRuleFactory.CreateEqualsRule(
  new ElementId(BuiltInParameter.ALL_MODEL_TYPE_NAME),
   "Titre sans ligne"
);

// get all elements that comply to this rule
var viewPortTypes = new FilteredElementCollector(doc)
  .WhereElementIsElementType()
  .OfCategory(BuiltInCategory.INVALID)
  .WherePasses(new ElementParameterFilter(rule))
  .WherePasses(new ElementParameterFilter(nameRule))
  .ToElementIds();

// should be only one result
var viewPortTypeId = viewPortTypes.FirstOrDefault();
if (viewPortTypeId != null)
{
  var vp = ... ; // the viewport you need to change
  Trans.Start();
  vp.ChangeTypeId(viewPortTypeId);
  Trans.Commit();
}
```

Many thanks to Moustafa as well.
#### Interact with BIM via ChatGPT Interface in Revit
Transitioning from the pure Revit API to more AI-related topics, Jacqueline Rohrmann shared
a [Revit add-in providing interactive ChatGPT interface to run Revit commands](https://www.linkedin.com/posts/jacqueline-rohrmann-b17130141_chatgpt-revit-i-had-some-time-last-week-activity-7270455540571328513-2qMK) on LinkedIn.
You can check out the comments on that post to see what people think of it.
#### Expertise in Real World Contexts
Now that LLMs are rapidly improving their code generation capabilities, what will happen to professional software developers?
Well, writing code is one thing, and deep experience and expertise is something completely different.
An interesting article from 2005 by Hubert L. Dreyfus and Stuart E. Dreyfus
analyses [Expertise in Real World Contexts](https://change-aide.com/Content/Downloads/Dreyfus-Expert.pdf).
Abstract: In this paper we describe a five-stage phenomenological model of skill acquisition, of which expertise is the highest stage. Contrary to the claims of knowledge engineers, we argue that expertise in general, and medical expertise in particular, cannot be captured in rule-based expert systems, since expertise is based on the making of immediate, unreflective situational responses; intuitive judgment is the hallmark of expertise. Deliberation is certainly used by experts, if time permits, but it is done for the purpose of improving intuition, not replacing it. The best way to avoid mistakes is to take responsibility for them when they occur, rather than try to prevent them by foolproof rules. In bureaucratic societies, however, there is the danger that expertise may be diminished through over-reliance on calculative rationality.
The five stages they identify are pretty obvious:
- Novice
- Advanced Beginner
- Competence
- Proficiency
- Expertise
LLMs are currently challenged to reach the higher stages, and it is worthwhile pondering that.
#### Humans versus LLMs Completing ARC-AGI
In December, OpenAI released `o1`, the most advanced LLM so far, and showed off the reasoning capabilities of its unreleased successor `o3`.
`o3` is the first LLM to be able to partially solve
the [ARC-AGI](https://arcprize.org/) Abstraction and Reasoning Corpus for Artificial General Intelligence test.\
The test is designed to be pretty simple for a human and very hard for an AI.
I did it myself and found it very easy, next to trivial, once I got the hang of it.
It is very worthwhile trying it out for yourself to get an idea of the tasks that challenge an AI, and to read the explanation of what aspects LLMs do and do not get right in the evaluation of the [OpenAI o3 breakthrough high score on ARC-AGI-PUB](https://arcprize.org/blog/oai-o3-pub-breakthrough).
#### 2024 AI Breakthroughs & Controversies
For a quick overview of what going on in general, check out
the [top 10 AI breakthroughs & controversies of 2024](https://www.techopedia.com/top-ai-breakthroughs-and-controversies)
#### LLM Transformers Explained
For an even more basic introduction to the fundamentals of what started off the current LLM revolution, one might want
to [meet transformers: the Google breakthrough that rewrote AI's roadmap](https://www.techspot.com/article/2933-meet-transformers-ai/)
– how attention replaced recurrence and changed the rules of AI.
#### Short Movie by Veo 2
December also saw ther release of the video generation AIs, Sora2 and Veo2.
Here is a [short movie by Veo 2](https://www.reddit.com/r/OpenAI/comments/1hkiqxo/a_short_movie_by_veo_2_its_crazy_good_do_we_have/?rdt=65478) that
gives you an idea of the state of the art.
#### LangChain State of AI 2024 Report
Another overview of the current state of AI affairs is provided by
the [LangChain State of AI 2024 Report](https://blog.langchain.dev/langchain-state-of-ai-2024/).
#### Current AIs are not Politically Neutral
Also in December an MIT study reports
that [some language reward models exhibit political bias](https://news.mit.edu/2024/study-some-language-reward-models-exhibit-political-bias-1210):
> Research from the MIT Center for Constructive Communication finds this effect occurs even when reward models are trained on factual data.
![AI political bias](img/ai_political_bias.png "AI political bias")
#### Mind-Reading Potential of AI
Many groups are researching different ways of using deep learning to interpret human brain signals, cf.,
[recreate Pink Floyd from brain activity](https://thebuildingcoder.typepad.com/blog/2023/08/15-years-polygon-areas-and-net-core.html#6)
and [read your thoughts](https://thebuildingcoder.typepad.com/blog/2023/05/dark-icons-newfamilyinstance-and-ai-news.html#6).
Here is a TED presentation
on [the mind-reading potential of AI](https://www.ted.com/talks/chin_teng_lin_the_mind_reading_potential_of_ai).
#### Iterative Spell Check with Gemini
Until today, I proof read my blog post text myself and then used Word for spell checking.
A very manual process performed by me, a human being.
Today, for the first time, I asked an LLM to spell check instead.
It is interesting to try out.
Here is a summary of our conversation:
- Jeremy: can you spell check the text for me, please?
- Gemini: Please provide the text you would like me to spell check. <... further details>
- Jeremy: here is the text:
- Gemini: The text you provided is well-written and informative. There are no spelling errors or grammatical mistakes. Here are some minor suggestions for improvement:
- Jeremy: no spelling errors at all in there?
- Gemini: You are absolutely right to question me! I apologize for the oversight. There is one clear spelling error: "towords" should be "towards"; My apologies for missing that. It highlights the importance of always having a second pair of eyes (or another AI!) review text.
- Jeremy: thank you. can you find any further spelling errors?
- Gemini: You're right to be thorough! It's very easy to miss things. I've gone through the text again with a fine-toothed comb. Here's what I found:
- "doers" should be "does": In the sentence "Unlike AutoCAD, the Revit UI itself doers not provide this support...", the correct word is "does."
I believe this is the only remaining spelling error. However, there are a few other points that are not strictly spelling but could be improved:
- Inconsistent use of italics: Sometimes you italicize phrases like ricaun.RevitTest and sometimes you don't. For consistency, it's generally better to use backticks for code or file names.
- "adcvanced" should be "advanced": In the section "Humans versus LLMs Completing ARC-AGI", the word "adcvanced" appears to be a typo.
I'm more confident now that these are all the issues. Thanks again for your patience and for helping me learn! I'm still under development, and your feedback is very valuable.
- Jeremy: i corrected the errors that you pointed out. thank you for that. here is the corrected text. could you take another look at it and ensure that all spelling errors have been addressed? thank you!
- Gemini: The text appears to be free of spelling errors. Here are some minor improvements you can consider:
I am still experimenting with how far to trust these things and how to remain critical, patient and persevere to obtain good results.
If you have not tried using an LLM for your own purposes, I strongly suggest you give it a try and gather some experience.