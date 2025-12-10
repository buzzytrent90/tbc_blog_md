---
post_number: "1518"
title: "Sched Param Guid"
slug: "sched_param_guid"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'parameters', 'python', 'revit-api', 'schedules', 'sheets', 'transactions', 'views', 'walls']
source_file: "1518_sched_param_guid.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1518_sched_param_guid.html"
---

### Schedule Parameter and Shared Parameter GUID
Lots of information on, from and about schedule parameters, and a new elegant solution to a long-standing challenge:
- [Direct access to shared parameter GUID](#2)
- [Getting parameter information from a schedule](#3)
#### Direct Access to Shared Parameter GUID
Alexander Ignatovich (Александр Игнатович) already shared several exciting solutions with us all in the past.
Today he faced a new problem.
As usual, he came up with an impressively direct solution to share.
The problem is rather old and known, and he found an elegant new way to solve it:
I wanted to get shared parameters GUIDs directly from project parameters.
Of course I found the solution demonstrated by
the [shared project parameter GUID reporter](http://thebuildingcoder.typepad.com/blog/2015/12/shared-project-parameter-guid-reporter.html), where you need to attach the parameter to the project information (for instance parameters) and wall (for type parameters) categories.
I also found another blog post on [parameter of Revit API 30 – project parameter information](http://spiderinnet.typepad.com/blog/2011/05/parameter-of-revit-api-30-project-parameter-information.html), but this solution did not work for me, because all definitions in `doc.ParameterBindings` are `InternalDefinition` objects in Revit 2017.
I investigated further and found that `InternalDefinition` has an `Id`. I retrieved the corresponding database element from the document by this `Id`. I saw that `SharedParameterElement` is returned for shared project parameters and `ParameterElement` for non-shared project parameters. `SharedParameterElement` has a `GuidValue` property, which is exactly what I need.
The code:
```csharp
[Transaction( TransactionMode.ReadOnly )]
public class CmdSharedParamGuids : IExternalCommand
{
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
var uiapp = commandData.Application;
var uidoc = uiapp.ActiveUIDocument;
var doc = uidoc.Document;
var bindingMap = doc.ParameterBindings;
var it = bindingMap.ForwardIterator();
it.Reset();
while( it.MoveNext() )
{
var definition = (InternalDefinition) it.Key;
var sharedParameterElement = doc.GetElement(
definition.Id ) as SharedParameterElement;
if( sharedParameterElement == null )
{
TaskDialog.Show( "non-shared parameter",
definition.Name );
}
else
{
TaskDialog.Show( "shared parameter",
$"{sharedParameterElement.GuidValue}"
+ "- {definition.Name}" );
}
}
return Result.Succeeded;
}
}
```
Thank you very much for this elegant solution, Alexander!
Sweet and simple!
I am sure it will make many people very happy!
I therefore added it to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
in [release 2017.0.132.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.132.0).
We can make use of this right away to enhance the answer to the question below as well.
#### Getting Parameter Information from a Schedule
From
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
on [getting parameter information from a schedule](http://forums.autodesk.com/t5/revit-api-forum/getting-parameter-information-from-a-schedule/m-p/6802850):
\*\*Question:\*\* The `ScheduleField` class has a property `ParameterId`, which is good.
Now I would like to know more about this parameter:
1. The name of the parameter.
2. If it is an instance or type parameter.
3. If it is a shared or built-in parameter.
4. The parameter Unit.
5. GUID of the parameter if it is shared.
I think I can get 1 and 2 working, but pretty much clueless about 3, 4 and 5.
\*\*Answer:\*\* Before anything else, I must ask you:
Have you installed [RevitLookup](https://github.com/jeremytammik/RevitLookup) and are you using it on a regular basis in your Revit database exploration?
With that tool, you can interactively snoop the database.
In this case, you could grab one of those ParameterId values, use the built-in Revit \*Select by Id\* command in the \*Manage\* tab to select the corresponding database element, regardless of whether it has a graphical representation or not, and interactively explore its nature, contents, parameters, relationships with other elements, etc.
That will probably answer your question.
It is statically compiled, however, so it mainly displays properties. It does not dynamically evaluate all methods available on all classes.
For that, you can use other,
even [more powerful and interactive database exploration tools](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html).
Also, my discussion
on [how to research to find a Revit API solution](http://thebuildingcoder.typepad.com/blog/2017/01/virtues-of-reproduction-research-mep-settings-ontology.html#3) might
come in handy for you at this point.
\*\*Response:\*\* I have installed RevitLookup for Revit 2015 (the version I'm developing as our development needs to be backward compatible with our current projects).
RevitLookup is a great tool that can save hundreds of hours of time on browsing the "watch" in the debug mode, which I have been using.
However, I still can't find the field parameter definitions (shared or built-in) for the `ScheduleField` class. I understand the "viewschedule" and "schedulefield" are different classes. I suspect this may have to do with Revit API's limitation.
By the way, it seems RevitLookup defines a ribbon panel in the 'Add-ins' tab – however, I can't see the 'Manage Tab'; am I missing something here?
\*\*Answer:\*\*
1. name of the parameter:
```csharp
Name = Field.GetName()
```
2. Type or Instance?
Probably only a definitive answer for a schedule of 1 System Family Category. Shared Parameters in User Created Families can be both Type and Instance (in different families).
3. Shared or BuiltIn?
- Field.ParameterId < -1 : BuiltInParameter – Field.ParameterId == BuiltInParameter value
- Field.ParameterId > 0 : SharedParameter
- Field.ParameterId = -1 : miscellaneous (calculated value, percentage)
Shared Parameter:
```csharp
SharedParameterElement shElem = doc.GetElement(
Field.ParameterId) as SharedParameterElement;
```
You can find the answer to 4 and 5 in `shElem.Definition` .
Built-in Parameter:
You need an element (Elem) to get access to a BuiltInParameter, so you need to have a non-empty schedule.
```csharp
Parameter par = Elem.get_Parameter(
(BuiltInParameter) Field.ParameterId.IntegerValue );
```
You can find the answer to 4 in `par.Definition`.
\*\*Response:\*\* Very nice answers, thanks a lot.
Regarding 2, I guess I have to use the `FamilyManager` to open the families to find out if the shared parameter is a 'Type' or 'Instance' one.
\*\*Answer:\*\* More on \*Select by Id\*:
[Select element by id](https://knowledge.autodesk.com/support/revit-products/learn-explore/caas/CloudHelp/cloudhelp/2017/ENU/Revit-Troubleshooting/files/GUID-2B1CC22C-CB1F-45DA-B57B-62C36013D9E0-htm.html) is
part of the standard end user interface:
- Help entry on [Select Elements by ID](http://help.autodesk.com/view/RVT/2017/ENU/?guid=GUID-2B1CC22C-CB1F-45DA-B57B-62C36013D9E0)
- 101-second YouTube video on [Selecting Elements Using the Element ID](https://www.youtube.com/watch?v=prv8nGrU56o):

As said, it comes in handy for using RevitLookup to snoop element data and properties, since it can be used on invisible elements that cannot be selected in any other way as well.
\*\*Response:\*\* I found the post you mentioned and tried; it works well that I can find the shared parameter visually on the Revit property panel even it is a 'hidden' element; that's quite cool. :)