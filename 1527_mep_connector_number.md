---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.223280'
original_url: https://thebuildingcoder.typepad.com/blog/1527_mep_connector_number.html
post_number: '1527'
reading_time_minutes: 4
series: mep
slug: mep_connector_number
source_file: 1527_mep_connector_number.md
tags:
- elements
- family
- parameters
- references
- revit-api
- selection
- sheets
- windows
- mep
title: Mep Connector Number
word_count: 890
---

### Revit MEP Connector Number
I answered a question in a forum I never participated in previously,
the [Revit MEP forum](http://forums.autodesk.com/t5/revit-mep-forum/bd-p/188),
prompted by Robert Klempau's direct mention in the thread
on [connector numbers of mechanical equipment](http://forums.autodesk.com/t5/revit-mep-forum/connector-numbers-of-mechanical-equipment/m-p/6870978).
Since it is rather technical and even includes a snippet of Revit source code, let me reiterate it here for better readability and future reference:
\*\*Question:\*\* I created a mechanical exhaust fan with four connectors for testing something else.
Interestingly, when I was creating a system, Revit asked me which connector I wanted to use. In the list of connectors, it looks like each one has its own connector number (connector 4 to 7).
Can someone tell me where is this parameter saved in the Revit family, and am I able to change that?
![MEP connector number](img/mep_connector_nr_1.png)
\*\*Answer 1\*\* Do you have any other type of connector in the family?
Usually they are numbered after the placement sequence.
To include a text, use the description parameter of the connector properties; that will make your life easier in this selection window.
\*\*Answer 2:\*\*
I created a video showing what to do when adding connectors of the same system classification to your family.
That you see only connector 4 till 7 and not 1 till 4 can have several reasons.
- There are other connectors in your model, like duct, conduit, cable tray or electrical connectors in your family.
- There are connectors in your family, but you removed some of them.
In your case I see 4 Duct connectors and 1 electrical connector.
Don't worry about the connector numbers.
As suggested, add the description to the connector so you know exactly which connector is which.
![MEP connector number](img/mep_connector_nr_2.png)
Here is a video providing the full details:

\*\*Response:\*\* You are right. There is an electrical connector. Besides that, initially there were two duct connectors and I removed them, then I added four duct connectors. That explains why my duct connectors in the dialog start from "4".
However, if two initial duct connectors had been removed, why does the current duct connectors start numbering from "2"? Does it mean these two initial connectors are still in my family? Do I need to find them out and delete them?
\*\*Answer 1:\*\* When you add a connector to your family, Revit will give it a number.
That connector always keeps that number.
When you remove the connector, Revit will remove also that number.
Revit will not use the deleted number again.
Every new connector gets the highest connector number in your family +1.
So, a removed connector will stay unused.
I used the RevitLookup tool to take a deeper look in the Revit family and there you see that there are only the connectors that are present in the family.
![Snooping MEP connector](img/mep_connector_nr_3.png)
My conclusion: When removing a connector, it will be removed permanently and it will not reuse that connector number again.
Note: The Snoop DB command is provided by the RevitLookup add-in I use to look directly into the Revit Database. Please see:
- [Revit 2016: click here](http://forums.autodesk.com/t5/revit-api-forum/revitlookup-for-revit-2016-is-here/m-p/5600976)
- [Revit 2017: click here](http://forums.autodesk.com/t5/revit-api-forum/revitlookup-for-revit-2017-is-here/m-p/6279580/highlight/true#M15603)
\*\*Answer 2:\*\* Checking the Revit source code, there is one key function to get the 'Connector number' in the edit family environment:
```csharp
int ConnectorElem::defaultIndex()
{
// ...
for( elemIter.initIter(); !elemIter.isDone();
elemIter.increment() )
{
pElem = getDocument()->getElement(
elemIter.getElementId() );
if( !pElem )
{
continue;
}
if( IS_A( ConnectorElem, pElem ) )
{
pConnectorElem = downcast  (
pElem );
nIndex = pConnectorElem->getIndex();
if( nIndex > nMaxIndex )
{
nMaxIndex = nIndex;
}
}
}
// We want to start the indices at 1 if there are none.
if( nMaxIndex <= 0 )
{
nMaxIndex = 1;
}
else
{
nMaxIndex++;
}
return nMaxIndex;
}
```
The index that you refer to as connector number is serialized with the `ConnectorElem`, so it remains unchanged once the connector exists.
Based on the above implementation, we can draw the following conclusions:
- The index starts from 1. In some old family, you might also find an index 0, caused by old code.
- The newly created connector index is the highest index + 1.
- The index might not be continuous. If you remove an 'inner' connector whose index is in \[1, maxIndex\), maxIndex is kept, so the index will be skipped.
- The index might be reused, if user removes all indices in \[index, maxIndex\].
So, to answer your questions:
- When you add a connector to your family, Revit will give it a number → Yes.
- That connector always keeps that number → Yes.
- When you remove the connector, Revit will remove also that number → Yes.
- Revit will no use the deleted number again → No: If you remove the maxIndex, the maxIndex will be reused.
- Every new connector gets the highest connector number in your family +1 → Yes, so a removed connector will stay unused.
- I used RevitLookup to take a deeper look in the Revit family and there you see that there are only the connectors that are present in the family → Index is a serialized parameter on the connector element, but it is not visible to the user.