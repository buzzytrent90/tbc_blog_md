---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:16.388516'
original_url: https://thebuildingcoder.typepad.com/blog/1610_analyticalmodelsupport.html
post_number: '1610'
reading_time_minutes: 3
series: structural
slug: analyticalmodelsupport
source_file: 1610_analyticalmodelsupport.md
tags:
- elements
- geometry
- python
- references
- revit-api
- sheets
- walls
- structural
title: Analyticalmodelsupport
word_count: 515
---

### How to Use GetAnalyticalModelSupports
I highlight yet another thread from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
on [how to use GetAnalyticalModelSupports](https://forums.autodesk.com/t5/revit-api-forum/getanalyticalmodelsupports-how-to-use-this/m-p/7503547),
answered by Dragos Turmac, Autodesk Software Engineer, and with additional usage hints by J. Peele, who also raised the question in the first place:
\*\*Question:\*\* I'm trying to retrieve connected framing elements through the API.
My end goal is to select a beam and retrieve the element id of the elements its ends are framing into.
I have been able to do this by checking location intersection of all other beams, but this scales by `n^2` based on the number of beams for the check.
I read the article
on [finding connected structural elements](http://thebuildingcoder.typepad.com/blog/2011/01/finding-connected-structural-elements.html),
but I'm confused on the actual implementation of it.
Has anybody come across a working example showing how to
call [`GetAnalyticalModelSupports`](http://www.revitapidocs.com/2018.1/15f01976-9e34-8850-6fa8-79c77a7ed3a4.htm)?
I'm not sure what objects have this property since they all seem to inherit it. Or am I missing an import?
I'm working in Python. Here is the code so far:

```
  import clr
  clr.AddReference('ProtoGeometry')
  from Autodesk.DesignScript.Geometry import *

  # Import RevitAPI
  clr.AddReference('RevitAPI')
  import Autodesk
  from Autodesk.Revit.DB import *

  # Import ToDSType(bool) extension method
  clr.AddReference('RevitNodes')
  import Revit
  clr.ImportExtensions(Revit.Elements)

  from Autodesk.Revit.DB.Structure import *

  # The inputs to this node will be stored as a list in the IN variables.
  dataEnteringNode = IN

  if isinstance(IN[0], list):
    elements = [UnwrapElement(i) for i in IN[0]]
  else:
    elements = [UnwrapElement(IN[0])]

  AnalyticalEl = [element.GetAnalyticalModel() for element in elements]

  # Assign your output to the OUT variable.
  OUT = AnalyticalEl;
```

Thanks so much for any help you can give.
\*\*Answer:\*\* Does this snippet work for you and answer your question?
```csharp
AnalyticalModel model
= aWallFoundation.GetAnalyticalModel()
as AnalyticalModel;
if( null == model )
continue;
IList supports
= model.GetAnalyticalModelSupports();
List ids
= new List();
foreach( AnalyticalModelSupport m in supports )
{
ids.Add( m.GetSupportingElement() );
}
```
\*\*Response:\*\* Finally got it working yesterday.
This code does the trick, though the problem was a conceptual oversight on my part.
I was imagining the supports would be similar to what pressing tab on a beam element does; however, it seems the supports are actually the members involved in the gravity load path.
More importantly, the analytical supports will be null if Revit has 'Automatic Member Supports' turned off (as it should be, Revit is unusably slow on large projects with it on).
I had to run a manual support check before anything would show up.
Hope this helps anyone having similar issue to what I ran into. Also, if there is any way to programmatically get the connected beams similar to how tab behaviour works, I would still be interested to learn how.
Many thanks to Dragos and J. for raising and solving this issue!
![Structural framing supports](img/rst_framing_supports.png)