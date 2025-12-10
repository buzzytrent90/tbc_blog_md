---
post_number: "1889"
title: "Param Filter Type"
slug: "param_filter_type"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'parameters', 'revit-api', 'sheets', 'views', 'walls', 'windows']
source_file: "1889_param_filter_type.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1889_param_filter_type.html"
---

### Parameter Filter Checking Element Type and OCR
Getting ready for a rainy weekend, here is an important insight in using a filtered element collector with a parameter filter, a handy open source OCR tool and a few productivity tips:
- [Parameter filter also checks element type](#2)
- [Parameter property also checks element type](#2.1)
- [Capture2Text, a handy OCR tool](#3)
- [Productivity tips](#4)
#### Parameter Filter Also Checks Element Type
We made an interesting discovery in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [string parameter filtering retrieving false data](https://forums.autodesk.com/t5/revit-api-forum/string-parameter-filtering-is-retrieving-false-data/m-p/10034687):
If the internal `ParameterValueProvider` class does not find the requested parameter from the element itself, will also try to read it from the element's type.
This undocumented behaviour can lead to some confusion and is difficult to fix perfectly in an efficient manner.
\*\*Question:\*\* I am trying to filter walls based on their "Fire Rating" parameter using the `FilterStringContains` evaluator.
However, this filter retrieves both walls satisfying the rule plus other walls that don't even contain that parameter!
This seems like an error to me.
Here is a [sample project with a macro](zip/TestFilterStringContains.rvt) providing a reproducible case using the following code:
```csharp
BuiltInParameter bip = BuiltInParameter.FIRE_RATING;
ElementId pid = new ElementId( bip );
ParameterValueProvider provider
= new ParameterValueProvider( pid );
FilterStringRuleEvaluator evaluator
= new FilterStringContains();
FilterStringRule rule = new FilterStringRule(
provider, evaluator, "/", false );
ElementParameterFilter filter
= new ElementParameterFilter( rule );
var myWalls = new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_Walls )
.WherePasses( filter );
List false_positive_ids
= new List();
foreach( Element wall in myWalls )
{
Parameter param = wall.get_Parameter( bip );
if( null == param )
{
false_positive_ids.Add( wall.Id );
}
}
string s = string.Join( ", ",
false_positive_ids.Select(
id => id.IntegerValue.ToString() ) );
TaskDialog dlg = new TaskDialog( "False Positives" );
dlg.MainInstruction = "False filtered walls ids: ";
dlg.MainContent = s;
dlg.Show();
```
This displays a message like this in the sample model, listing walls that are not even equipped with the target parameter, let alone have a value for it:
![Parameter filter returns false positives](img/TestFilterStringContains_result_message.png "Parameter filter returns false positives")
What can I do to ignore walls that don't contain the parameter?
\*\*Answer:\*\* This was logged as \*REVIT-172990 – Parameter filter with FilterStringContains returns false positive\* for the development team to analyse.
You might be able to achieve the desired result by combining the FilterStringContains evaluator with other rules and filters.
For instance, you may be able to add something like 'is not empty' in addition to 'contains'.
The development team analysed the issue and found the reason for this behaviour.
If the internal method `ParameterValueProvider::getParameterValue()` cannot get the parameter from the element itself, will also try to get it from the element's type:
```csharp
if (m_elemOrSymbol == EOS_Symbol
|| err != ERR_SUCCESS || !oParameterValue)
{
ElementId typeId = pElement->getTypeId();
if (validElementId(typeId))
{
const Element \*pTypeElement
= pElement->getDocument()->getElement(typeId);
if (pTypeElement != NULL)
{
oParameterValue = pTypeElement->getParameterValue(
m_parameter);
err = getErrFromParameterValue(oParameterValue);
if (err != ERR_SUCCESS)
{
return nullptr;
}
}
}
}
```
Therefore, yes, you cannot find the parameter in the walls.
However, you can find the parameters in their wall type.
That's why you can still get those walls.
This behaviour has been in effect for a long time, so the development team is not thinking about changing it.
However, this information needs to be made clear in the documentation and knowledge base.
A new issue \*REVIT-173253 – Parameter filter with FilterStringContains returns false positive\* has been raised for the documentation team to document this behaviour.
\*\*Response:\*\* I am going to accept your reply as an accepted answer but it is not as expected.
- I think this behaviour is not acceptable logically, even though it was in effect for a long time.
At least it should return the wall type itself rather than returning all walls under that type.
- Can it be modified to have a Boolean to search through the element's type or not?
Or, like the filtered element collector, e.g., `WhereElementIsElementType` and `WhereElementIsNotElementType`?
\*\*Answer:\*\* I agree that this behaviour defeats part of the purpose of the parameter filter.
If I were you, I would choose the simple solution of filtering for parameters that are guaranteed not to be present on the type.
\*\*Answer 2:\*\* I always preferred the functionality as it is now. There are various solutions out there for finding elements that rely on not having to separately look through the parameters on the type or know that a parameter is on a type or an instance.
Imagine you want to find all instances that have a certain type parameter, you would first have to filter for all types, filter those for the parameter match, then filter again for all instances that are of one of those types. This is already a common approach for a lot of cases where parameter filters can’t be used.
I thought it just saved a lot of work really, I see no need for a change. If you want to know the parameter is on the instance then put the results through a further filter. You can supply a list of ids to a further element filter at any time. If you want to know parameter is on the type, filter first for types. In general, I don’t see much use of Logical AND/OR Filters sometimes I wonder if their existence is known of? People seem to have other methods but again use: LogicalAndFilter with
- ParameterElementFilter and

ElementIsElementTypeFilter(Inverted = True)
ElementParameterFilter is a slow filter, so you should first use another quick filter before it.
It works like in the UI when you set up visibility filters, i.e., there is no distinction between type and instance parameters there. I imagine this is the internal system that this filter class leverages.
Later: When I consider further there is a bit of a hole in functionality there.
Since you are forced to get a list of elements and then rule out those that have a type parameter match not an instance parameter match, i.e., there are four cases:
Parameter with matching name and value is on: Type, Instance, Both, Neither.
We can only find with filter that it is on: Either or Neither.
So, the question is, how would we find Elements with a matching parameter on the Instance only?
After filtering with ElementParameterFilter, we would have to manually check via Linq perhaps.
We only care that it is or isn't on the instance, so that could simplify things.
\*\*Answer 3:\*\* Exactly what I meant by saying, defeats part of the purpose.
Thank you for fleshing it out in more detail.
Linq is super slow compared to the built-in parameter filter, and I see no other option to achieve the in-between distinctions.
Many thanks to Ameer and Richard for this very fruitful and illuminating discussion!
#### Parameter Property Also Checks Element Type
Our learning is never-ending.
I response to the above,
Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen added:
> I noticed the same behaviour for the Parameter properties of an Element.
Getting a parameter on (family) instances by GUID or Definition also returns the type-parameter:
![Parameter property by GUID and Definition](img/param_by_guid_includes_type.png "Parameter property by GUID and Definition")
> Please update the documentation for these properties as well.
#### Capture2Text, a Handy OCR Tool
I happened upon a very handy OCR tool that looks pretty good:
[Capture2Text](http://capture2text.sourceforge.net) is
a free open source Windows desktop OCR application, licensed under the terms of the GNU General Public License.
It is implemented as a hotkey app, making it very minimalistic and efficient to use:
Click the Windows button + Q, mouse the source pixel rectangle to capture, and the resulting text is placed in the clipboard.
It can't get much more minimal or efficient than that, can it?
Besides that, Capture2Text supports:
- Almost a hundred target languages
- Multiple simultaneous target languages
- Immediate translation via Google translate
- Text-to-speech voice
![Capture2Text](img/Capture2Text_conceptual_illustration.png "Capture2Text")
#### Productivity Tips
For more efficiency in personal and profession life in general, here are a couple of tips by
Endy Austin
on [how to be productive, feel less overwhelmed, and get things done](https://www.freecodecamp.org/news/how-to-get-things-done-lessons-in-productivity).