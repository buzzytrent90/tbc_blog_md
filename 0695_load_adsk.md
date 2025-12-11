---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:14.413179'
original_url: https://thebuildingcoder.typepad.com/blog/0695_load_adsk.html
post_number: 0695
reading_time_minutes: 4
series: structural
slug: load_adsk
source_file: 0695_load_adsk.htm
tags:
- csharp
- doors
- family
- revit-api
- transactions
- views
- windows
- structural
title: Loading an Inventor ADSK Component
word_count: 864
---

### Loading an Inventor ADSK Component

We held the DevLab in Munich yesterday before rushing off to get to Milano in the evening.

I slept a little bit in the taxi and plane on the way here.
I was still dead tired on arrival at the hotel just before ten, but thought I absolutely must get out and have a teeny weeny little sniff of Italy anyway before going to bed.
I went to a bar right next door and committed the deadly foreigner sin of having a cappuccino, which it is almost a criminal offence to drink except at breakfast time.
Quite soon the very friendly couple at the neighbouring table, Agnès and Max, informed me that a free concert was going to start shortly next door.
Agnès and Max are both a photographers, just like Dima, whom I met in
[Jerusalem last Thursday](http://thebuildingcoder.typepad.com/blog/2011/12/rex-content-generator.html),
so Thursday seems to be my meet-a-photographer day right now.
Check out
[Agnès' site](http://www.penombre.it) and
[blog](http://agnesweber.blogspot.com) and
[Max' photos](http://www.msvphoto.altervista.org) and
[blog](http://msvphoto.wordpress.com).

![Penombre](img/penombre.jpg)

The concert was really great, by the Italian singer
[Folco](http://it.wikipedia.org/wiki/Folco_Orselli)
[Orselli](http://www.myspace.com/folcoorselli).
I liked it so much I bought his CD, asked for the very first autograph of my life, and ended up in bed rather late, or early, depending on your point of view :-)

#### Loading an Inventor Component ADSK File

One of the many interesting topics raised at the DevLab came from Martin Schmid, our MEP expert, working with both AutoCAD and Revit.
He brought an ADSK component file created in Inventor with the goal of loading it into Revit.

I tried one or two things to achieve this which
[failed](#2)
before finding a
[successful solution](#3),
so you might want to skip straight to that if you are not interested in the blather.

#### Failed Attempts

The first thing we tried was the LoadFamily method, which simply does not work at all and is not intended to do so either:
```csharp
  const string filename
    = "C:/a/doc/revit/blog/src/LoadAdsk/Part1.adsk";

  // Load ADSK using LoadFamily does not work;
  // rc is false, and f is null:

  t = new Transaction( doc );
  t.Start( "Load Adsk" );
  Family f = null;
  bool rc = doc.LoadFamily( filename, out f );
  t.Commit();
```

This does not cause any error, but 'rc' is false and 'f' remains null.

The second attempt was trying to open the component as a document using OpenDocumentFile:
```csharp
  Document adsk\_doc = app.OpenDocumentFile( filename );
```

This should achieve the same result as dragging and dropping the component onto the Revit window, which opens it in the family editor.
It does actually successfully open the ADSK in the family editor but it also displays an error saying "Attempted to read or write protected memory. This is often an indication that other memory is corrupt."

![Error message on opening ADSK file](img/adsk_openDocumentFile_error.png)

I am not yet sure whether this message is erroneous.
Maybe you can simply ignore it and automatically dismiss it through the API.

Anyway, after those two failed attempts, we went on to find a solution which really works:

#### Successful Workflow using OpenBuildingComponentDocument

Scott Reinemann pointed out that we should try the OpenBuildingComponentDocument method, and that works fine.

Here is the code that achieves exactly what we want:

- Open the ADSK component family file.- Load the component family into the project.- Retrieve a family symbol from it.- Place an instance of the symbol.

```csharp
  Document adsk\_doc
    = app.OpenBuildingComponentDocument( filename );

  Debug.Print(
    "Opened component document {0} with path '{1}'",
    adsk\_doc.Title, adsk\_doc.PathName );

  t = new Transaction( doc );
  t.Start( "Load ADSK component document" );
  Family f = adsk\_doc.LoadFamily( doc );
  t.Commit();

  Debug.Print(
    "Loaded component family {0}",
    f.Name );

  FamilySymbol symbol = null;

  foreach( FamilySymbol s in f.Symbols )
  {
    symbol = s;
    break;
  }

  t.Start( "Place ADSK component document" );
  FamilyInstance i = doc.Create.NewFamilyInstance(
    XYZ.Zero, symbol, StructuralType.NonStructural );
  t.Commit();

  Debug.Print(
    "Placed component family instance {0}",
    i.Name );
```

Here is the Part1 symbol defined by the component family appearing in the project browser after being loaded:
![Part1 family symbol in project browser](img/adsk_part1_project_browser.png)

This is the family instance placed in the project origin:

![Part1 family instance](img/adsk_part1_instance.png)

Right now the PathName property remains empty, but the Title property is set as expected, so the messages printed to the Visual Studio debug output window say:

```
Opened component document Part1 with path ''
Loaded component family Part1
Placed component family instance Part1
```

This implementation automatically places the family instance hardwired at the origin and does not mimic the placement of a family when using the user interface, which displays a preview of the symbol at the cursor.

To have that, you can make a call to
[PromptForFamilyInstancePlacement](http://thebuildingcoder.typepad.com/blog/2010/06/place-family-instance.html) instead.

Here is
[LoadAdsk.zip](zip/LoadAdsk.zip) containing
the source code and complete Visual Studio solution for this command as well as a sample component file Part1.adsk to play around with.