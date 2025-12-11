---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.615778'
original_url: https://thebuildingcoder.typepad.com/blog/0246_room_occupancy.html
post_number: '0246'
reading_time_minutes: 6
series: general
slug: room_occupancy
source_file: 0246_room_occupancy.htm
tags:
- csharp
- elements
- family
- filtering
- parameters
- revit-api
- rooms
- selection
title: Room Occupancy
word_count: 1113
---

### Room Occupancy

Shifali asked a
[question](http://thebuildingcoder.typepad.com/blog/2009/04/new-2010-events.html#comment-6a00e553e1689788330120a6ab278e970c) on
reading and writing the room occupancy:

**Question:** We get the document from ExternalCommandData object's Application and the project information in document.
We need to store the construction type of the current document or project file.
Moreover, we get the Room elements of the document but we need to store the occupancy type of each room for applying the International Building Code for compliance reports of the project.

We also need access to the BuildingConstruction and BuildingType in the document ProjectInformation.gbXMLSettings.

**Answer:** I started by looking at the room occupancy issue, and found that this is actually very simple and straightforward and has been covered in numerous previous posts dealing with the
[exploration of element parameters](http://thebuildingcoder.typepad.com/blog/2008/11/exploring-element-parameters.html).
Still, since this is a very typical and recurring issue, I will happily revisit it.

I started up Revit and created a new model with a room or two.
Left clicking one of these, and selecting Element Properties... I see the parameter that I presume you are looking for listed as Occupancy under the Identity Data heading.
So that is where to find this data in the user interface.
I entered an arbitrary string value there in order to start exploring how to access the same data programmatically.

Next, I start up
[RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html)
([for 2010](http://thebuildingcoder.typepad.com/blog/2009/10/rvtmgddbg-for-revit-2010.html))
and use that to look at the room element parameters through the API via Add-Ins > RvtMgdDbg > Snoop Current Selection... > Parameters > Occupancy.
It shows the same string I just entered, so I seem to have found the required data.
Since I would like to access it language independently, I need to determine the built-in parameter enumeration value for this parameter, if one exists.
If I don't care about that, I can use the language dependent parameter name "Occupancy" to identify the data.
The built-in parameter can be determined by using the Built-in Enums Snoop or Built-in Enums Map buttons in RvtMgdDbg, which show me that the built-in parameter I am looking for is ROOM\_OCCUPANCY.

I can also look at the element parameters and see the associated built-in parameters using the Revit API introduction labs
[built-in parameter checker](http://thebuildingcoder.typepad.com/blog/2009/04/deeper-parameter-exploration.html).

Both of these tools show me that the parameter is string valued and read-write, so there should be no problem modifying it.

I implemented a new Building Coder sample command CmdSetRoomOccupancy to demonstrate making use of this to read and write the parameter and do nothing else.
For experienced Revit developers, this is a rather trivial command, since all it does is read and write the value of a single element parameter.
Still, it should be useful to point people to in the future who raise this frequently asked question.
The command retrieves either the currently selected rooms or all rooms in the model if nothing has been preselected using my GetSelectedElementsOrAll utility method.
For each room, its occupancy parameter is read and incremented.
This is achieved using the BumpStringSuffix helper method.

BumpStringSuffix increments the numerical suffix of a given string.
If the string already ends in a sequence of digits representing a number, it returns a string with the number incremented by one.
Otherwise, it returns the original string with a suffix "1" appended.
In case the original string is null, "1" is returned:
```csharp
static char[] \_digits = null;
static string BumpStringSuffix( string s )
{
  if( null == s || 0 == s.Length )
  {
    return "1";
  }
  if( null == \_digits )
  {
    \_digits = new char[] {
      '0', '1', '2', '3', '4',
      '5', '6', '7', '8', '9'
    };
  }
  int n = s.Length;
  string t = s.TrimEnd( \_digits );
  if( t.Length == n )
  {
    t += "1";
  }
  else
  {
    n = t.Length;
    n = int.Parse( s.Substring( n ) );
    ++n;
    t += n.ToString();
  }
  return t;
}
```

Here is the method BumpOccupancy which makes use of BumpStringSuffix to increment the room occupancy parameter on a given room element.
It reads the value of the element ROOM\_OCCUPANCY parameter.
If it ends in a number, it increments that number, otherwise it appends "1":
```csharp
static void BumpOccupancy( Element e )
{
  Parameter p = e.get\_Parameter(
    BuiltInParameter.ROOM\_OCCUPANCY );

  if( null == p )
  {
    Debug.Print(
      "{0} has no room occupancy parameter.",
      Util.ElementDescription( e ) );
  }
  else
  {
    string occupancy = p.AsString();

    string newOccupancy = BumpStringSuffix(
      occupancy );

    p.Set( newOccupancy );
  }
}
```

Now we can apply the BumpOccupancy method to all or the currently selected rooms in the model.
Here is the entire code of our external command Execute method which does so:
```csharp
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  List<Element> rooms = new List<Element>();
  if( !Util.GetSelectedElementsOrAll(
    rooms, doc, typeof( Room ) ) )
  {
    Selection sel = doc.Selection;
    message = ( 0 < sel.Elements.Size )
      ? "Please select some room elements."
      : "No room elements found.";
    return CmdResult.Failed;
  }
  foreach( Room room in rooms )
  {
    BumpOccupancy( room );
  }
  return CmdResult.Succeeded;
}
```

Here is
[version 1.1.0.54](zip/bc11054.zip)
of the complete Building Coder sample source code and Visual Studio solution including the new command.

So, I would hope that this amply answers the question on accessing and modifying the room occupancy, and also serves as a model for further simple parameter setting explorations.

Turning to look at the BuildingConstruction and BuildingType values in the ProjectInformation gbXMLSettings, I once again resort to RvtMgdDbg.
I select Add-Ins > RvtMgdDbg > Snoop Db... > ProjectInfo > Project Information 69280 > gbXML Settings ... and crash.
Well, first a message was displayed saying that this is only available in Revit MEP.

Ok, I restarted in Revit MEP and navigated through the same path down to the gbXML settings, which I can enter and explore this time.
I see the building construction and building type properties.
The former is an object, the latter displays a value kOffice.

Actually, I also see en entry for the gbXMLParamElem in the root directory of the RvtMgdDbg database snoop:

![Snoop gbXMLParamElem](img/gbXMLParamElem.png)

The element id shows me that this is the same object as the one stored in the project information.

So obviously there is no problem accessing this data either.
gbXMLParamElem is one of the classes defined by the Revit API and can be accessed using the standard Revit
[element filters](http://thebuildingcoder.typepad.com/blog/2009/03/filter-for-a-family.html),
for instance.
The desired properties are both members of this class:

- BuildingConstruction returns the Project Information Building Construction object.- BuildingType gets or sets the Project Information Building Type.