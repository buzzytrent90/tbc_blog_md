---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: code_example
optimization_date: '2025-12-11T11:44:16.590166'
original_url: https://thebuildingcoder.typepad.com/blog/1715_list_ifc_zones_py.html
post_number: '1715'
reading_time_minutes: 6
series: general
slug: list_ifc_zones_py
source_file: 1715_list_ifc_zones_py.md
tags:
- elements
- filtering
- parameters
- python
- references
- revit-api
- rooms
- sheets
title: List Ifc Zones Py
word_count: 1128
---

### Retrieving Linked IfcZone Elements Using Python
I spent a lot of time last week and during the weekend playing around with
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) topic
of [finding all ducts that have been tapped into](https://forums.autodesk.com/t5/revit-api-forum/find-all-ducts-that-have-been-tapped-into/m-p/8485269).
It led to several new releases
of [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.145.4).
Since we have not reached a final conclusion yet, however, I'll postpone the discussion of that.
Instead, I present a little exploration of how to access zones defined in an IFC file in Revit.
If you are following this blog closely, you might guess that this is related to
the [room boundary CSV exporter project](https://thebuildingcoder.typepad.com/blog/2019/01/room-boundaries-to-csv-and-wpf-template.html) that
I recently discussed.
For this exploration, I installed and used RevitPythonShell.
Here are the detailed steps:
- [Importing IFC zones into Revit](#2)
- [Installing and using RevitPythonShell](#3)
- [Programmatically accessing IFC zones in Revit](#4)
#### Importing IFC Zones into Revit
I started out my explorations by chatting with our IFC expert Angel Velez:
[Q] Does Revit IFC import also support IFCZONE? Can we use IFCZONE to generate Revit Zone elements?
[A] Exporting zones from Revit to IFC is supported,
and [you have to set up the project properly](https://sourceforge.net/p/ifcexporter/wiki/Exporting%20Zones) for that.
BTW, happy to say that Googling for 'Revit IfcZone' got me to that link!
Import, however, ignores zones.
Link creates them, as a subset of generic models.
[Q] When you say, 'link creates them', does it mean: in a Revit project, link in an IFC file.
IFCZONE elements are now accessible and visible in Revit and we can query their properties and boundaries?
[A] Correct – for link at least. Import does nothing with them.
[Q] OK... I have now linked an IFC file into a blank Revit document.
How can I access the zone information from here?
Is there any way in the UI?
[A] You have to tab into the document until you choose the zone. It will overlap the rooms it contains.
![IFC zone tabbed to](img/ifc_zone_tabbed_to.png)
[Q] Yes, I see it now.
I see a generic element with the `IFCZONE` properties, e.g., `IfcName`, and `IfcExportsAs` set to `IfcZone`.
What would be the workflow to generate a mapping from room elements to `IFCZONE` elements using the Revit API?
[A] I believe the rooms have a property that has the name of the zone(s) they belong to.
Although the 'rooms' are also generic elements. The Revit IFC import does not create real rooms, just space volumes. Converting to rooms is a request.
Thank you very much, Angel, for all the help!
The IFC zones are imported as `DirectShape` elements and assigned to the `Generic Model` category.
Their IFC properties are stored in shared parameters, like this:
![IFC zone properties](img/ifc_zone_properties.png)
As far as I can tell, I just need to look at the `IfcName` and `IfcExportAs` properties.
#### Installing and Using RevitPythonShell
Based on that discussion, I linked in an IFC file containing zones representing apartments into a blank Revit project document.
I started out exploring the model using [RevitLookup](https://github.com/jeremytammik/RevitLookup).
Since the objects of interest resided in the linked IFC file, however, I soon had to take recourse
to [another, more flexible, advanced and intimate database exploration tool](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html).
I visited the [RevitPythonShell home page](https://github.com/architecture-building-systems/revitpythonshell) and
downloaded the installer for Revit 2019.
It is a one-click install.
You can even do it while Revit is up and running – it will be auto-loaded into the running session.
Then, go to the `Add-In` tab and click the icon. That displays the console. Then copy-paste the code into that.
You can also load and run scripts in other ways – I've never tried that, though.
#### Programmatically Accessing IFC Zones in Revit
The RevitPythonShell enabled me to interact with the linked database, explore its elements and develop the following script step by step:

```
import clr
import math
clr.AddReference('RevitAPI')
clr.AddReference('RevitAPIUI')
from Autodesk.Revit.DB import *
from Autodesk.Revit.DB.Architecture import *
from Autodesk.Revit.DB.Analysis import *
uidoc = __revit__.ActiveUIDocument
doc = __revit__.ActiveUIDocument.Document
app = doc.Application
docs = app.Documents

n = docs.Size

print n, 'open documents:'

for d in docs:
  s = d.PathName
  print s
  if s.endswith('.ifc.RVT'): ifcdoc = d

print 'Linked-in IFC document:'
print ifcdoc.PathName

collector = FilteredElementCollector(ifcdoc).OfClass(clr.GetClrType(DirectShape)).OfCategory(BuiltInCategory.OST_GenericModel)

print collector.GetElementCount(), 'generic model direct shape elements'

def get_param(e,s):
  "Return string parameter value for given parameter name"
  ps = e.GetParameters(s)
  n = ps.Count
  assert(2 > n)
  if 0 < n: return ps[0].AsString()
  else: return None

def is_zone(e):
  "Predicate returning True is e is an IfcZone"
  export_as = get_param(e,'IfcExportAs')
  return export_as and export_as == 'IfcZone'

def zone_name(e):
  "Return IfcName of IfcZone element or None"
  if is_zone(e):
    return get_param(e,'IfcName')

zone_names = []

for e in collector:
  if is_zone(e):
    zone_names.append(get_param(e,'IfcName'))

zone_names.sort()

n = len(zone_names)

print n, 'zones:', zone_names
```

I am also linking in this script source here as a separate text
file [get_ifc_zone_properties.py](zip/get_ifc_zone_properties.py).
It lists all the open documents, namely two, the blank hosting project and the imported IFC file.
The imported IFC file has generated a placeholder RVT.
That is the document that I need to dig deeper into.
Using a filtered element collector, I retrieve all `DirectShape` elements belonging to the `Generic Model` category.
A couple of helper functions extract a named parameter from an element and implement a Boolean predicate to determine whether an element represents an IFC zone.
With those in hand, I can iterate over all the 1012 direct shapes, access the 25 zones, and save their `IfcName` properties to a list.
Here is the result of running the script:

```
2 open documents:
C:\...\test\010-123xx3-arc-bat01-apt01_2_2018-12-27_1507_ifc_link_host.rvt
C:\...\010-123xx3-arc-bat01-apt01_2_2018-12-27_1507.ifc.RVT

Linked-in IFC document:
C:\...\010-123xx3-arc-bat01-apt01_2_2018-12-27_1507.ifc.RVT

1012 generic model direct shape elements

25 zones: ['APT0101', 'APT0102', 'APT0103', 'APT0104',
  'APT0105', 'APT0106', 'APT0201', 'APT0202', 'APT0203',
  ...
  'APT0402', 'APT0403', 'APT0404', 'APT0405', 'APT0406',
  u'Par d\xe9faut:127272']
```

I very much enjoyed this little excursion into IFC matters and playing around interactively with the Revit API and Python.
I hope you enjoyed this short summary of my experiences.