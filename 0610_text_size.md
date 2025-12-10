---
post_number: "0610"
title: "Text Size"
slug: "text_size"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'references', 'revit-api', 'selection', 'transactions']
source_file: "0610_text_size.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0610_text_size.html"
---

### Text Size

I am back again from the
[Autodesk European Football Tournament 2011](http://through-the-interface.typepad.com/through_the_interface/2011/07/recovering-from-autodesks-european-football-tournament-2011.html) in Verona, Italy,
vividly described in more detail by Kean on his blog.
We were both part of the combined Global Services (to which we of ADN belong) and Worldwide Marketing team:

![Autodesk GS&M Team 2011](file:////j/photo/jeremy/2011/2011-07-02_verona_football/gs_and_m_team.jpg)

Back to the Revit API, we already looked at some aspects of text formatting such as
[textnote alignment](http://thebuildingcoder.typepad.com/blog/2010/04/revitlookup-and-textnote-alignment.html) and
[rotation](http://thebuildingcoder.typepad.com/blog/2010/06/textnote-rotation.html).
Here is a still more immediate and simpler issue:

**Question:** I created some text using the NewTextNote method.
Now I would like to change its size or font like a normal user might do using Edit type > Text size.
How can that be achieved programmatically?

**Answer:** The Text Size parameter can be set as follows:
```csharp
  Reference r = uidoc.Selection.PickObject(
    ObjectType.Element );

  TextElement text = doc.GetElement( r )
    as TextElement;

  TextElementType textType = text.Symbol;

  Parameter textSize = textType.get\_Parameter(
    "Text Size" );

  Transaction transaction
    = new Transaction( doc );

  transaction.Start( "change text size" );

  // units are in feet, so this is 1/8"

  double newSize = ( 1.0 / 8.0 ) \* ( 1.0 / 12.0 );

  textSize.Set( newSize );

  transaction.Commit();
```

Obviously, it would be better still to use the built-in parameter enumeration value to identify the text size parameter, rather than the language dependent user display parameter name.