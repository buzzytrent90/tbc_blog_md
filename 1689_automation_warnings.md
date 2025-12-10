---
post_number: "1689"
title: "Automation Warnings"
slug: "automation_warnings"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'revit-api', 'rooms', 'sheets', 'transactions', 'views', 'walls']
source_file: "1689_automation_warnings.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1689_automation_warnings.html"
---

### Swallowing StairsAutomation Warnings
Here at the Forge Accelerator in Rome, I am starting to take some a first look at
the [Forge](https://autodesk-forge.github.io)
[Design Automation API](https://forge.autodesk.com/en/docs/design-automation/v2/overview) for Revit.
It is not yet available or documented, except to a closely restricted private beta that I am not a member of, so I cannot go into any details.
For more information on its current status, please refer to
[Mikako Harada's discussion of Design Automation for Revit](https://fieldofviewblog.wordpress.com/revit).
However, you can prepare for the day when it comes by handling your add-in warnings properly.
To make use of it, you obviously need to know the Revit API, and it becomes very easy indeed if you also have some experience with Forge apps.
Revit API code can be run in a Forge app by using the `IExternalDBApplication` interface, already listed in
the [Revit API documentation](https://apidocs.co/apps/revit/2019/97318be3-45c4-d93b-ee7b-174fa80ab951.htm).
This interface supports addition of DB-level external applications to Revit, to subscribe to DB-level events and updaters.
DB-level applications cannot create or modify UI.
Therefore, if your add-in pops up any warnings, it cannot be converted to a Forge Design Automation for Revit app – or, worse still, it will simply silently terminate as soon as it misbehaves.
Therefore, today, let's take a look at suppressing warnings caused by a typical Revit add-in.
As an example, we'll choose the StairsAutomation Revit SDK sample.
It generates five different types of stairs:
![StairsAutomation result](img/StairsAutomation_result.png)
Two of them generate Revit warning messages:
- Stair #3 generates [8 warnings about overlapping handrail model line elements](zip/StairsAutomation_warnings_stair_3_8.html).
- Stair #4 generates [1 warning about a missing riser](zip/StairsAutomation_warnings_stair_4_1.html).
Happily, Revit warnings can easily be handled automatically making use of
the [Failure API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32).
Specifically, we presented
a [generic warning swallower](http://thebuildingcoder.typepad.com/blog/2016/09/warning-swallower-and-roomedit3d-viewer-extension.html#2) that
can handle just about any warning message that crops up.
For the StairsAutomation sample, nothing much is required.
The code generating the stairs obviously runs inside a `Transaction`, and that, in turn, is enclosed in a `StairsEditScope`.
The call to `Commit` the stair editing scope is called with a custom failures preprocessor instance:
```csharp
editScope.Commit(
new StairsEditScopeFailuresPreprocessor() );
```
In the original sample, the failures preprocessor does next to nothing:
```csharp
class StairsEditScopeFailuresPreprocessor
: IFailuresPreprocessor
{
public FailureProcessingResult PreprocessFailures(
FailuresAccessor a )
{
return FailureProcessingResult.Continue;
}
}
```
I simply added the following lines of code to it, to delete all warnings before returning:
```csharp
IList failures
= a.GetFailureMessages();
foreach( FailureMessageAccessor f in failures )
{
FailureSeverity fseverity = a.GetSeverity();
if( fseverity == FailureSeverity.Warning )
{
a.DeleteWarning( f );
}
}
```
Now, all five stair variations are created without any warning messages being displayed.
Of course, in your own more complex add-ins, you may need to handle other failures beside simple warnings that can be ignored.
For the most general case, you can make use of
the [generic warning swallower](http://thebuildingcoder.typepad.com/blog/2016/09/warning-swallower-and-roomedit3d-viewer-extension.html#2) mentioned
above.
To document the steps I took to achieve this and track all the changes I made, I extracted the sample to an
own [StairsAutomation GitHub repository](https://github.com/jeremytammik/StairsAutomation).
It ended up being so simple that I need actually not have bothered, though...
Looking forward to making further explorations and digging deeper into this area anon.