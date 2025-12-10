---
post_number: "0142"
title: "Imports in Families"
slug: "imports_in_families"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'levels', 'python', 'revit-api', 'views', 'windows']
source_file: "0142_imports_in_families.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0142_imports_in_families.html"
---

### Imports in Families

I will have left on vacation once again by the time this is posted.
In case you wonder why I am on vacation so much, the
[trip to Italy](http://thebuildingcoder.typepad.com/blog/2009/03/back-again.html)
in March and the
[wonderful experiences](http://thebuildingcoder.typepad.com/blog/2009/03/more-questions.html)
I had there were actually to use up last year's contingent.
Now I am getting started on this years', going off on a climbing and driving trip to southern France with my two sons Christopher and Cornelius, especially to the
[Provence](http://en.wikipedia.org/wiki/Provence)
and
[Avignon](http://en.wikipedia.org/wiki/Avignon)
to visit my mother, sister and nephew.
So I will be posting a bit less or not at all for the coming two weeks.

Before leaving, though, here is an interesting and practically oriented question by my colleague Martin Schmid of Autodesk.
I wrote a new command for The Building Coder samples to answer it, which may even be useful for some in everyday life.
Martin is an experienced AutoCAD MEP and Revit MEP expert and application engineer and helped me a lot with the preparation of the
[Revit MEP API class](http://au.autodesk.com/?nd=class&session_id=2658)
at Autodesk University.

**Question:**
How hard do you think it would be to put together a quick bit of code that would:

1. Iterate over all family definitions in a project.- For each family definition, identify if there are any import symbols, i.e., imported DWG, SAT, DXF and other non-native Revit geometry?

The reason I ask is that performance is degraded if very complex geometry is imported.
Some complaints about performance were caused by such complex imported geometry.

I have an example demonstrating slow graphics generation in a mechanical view with an offending family.
It takes more than 30 seconds to zoom in on the area with the bad family.
In the family editor, one can see that there is an imported DXF symbol at that location.
If you return to the project and delete the offending family instances, the zoom is nearly instantaneous.

Unfortunately, it is difficult to isolate these family instances.
I am hoping there is a simple way to check families using the API, instead of the rote manual inspection method I have relied on so far.

Do you think you could bang something together on this?
Ive got another large customer dataset to wade through, and was hoping for some help!

**Answer:**
This is not at all hard to do.
Here is a description of an algorithm that achieves what you need:

- Select all family instances.- For each instance, open the corresponding family definition.- Within the family definition, list all imported symbols.

We could also select the families themselves directly, without going over the instances first.
I chose not to do so in the code presented below, because then we have to deal with various built-in families that are not actually used and we are not interested in anyway.

Here is the approach I have taken in more detail.
I added an additional step to ensure every family is processed only once, even if multiple family instances refer to it.
To achieve this, I create a dictionary mapping the family name to the family, and insert the families used by the family instances only if not already present.
This way, I can later also process the families in alphabetical order, to make it easier to analyse the report produced.
In-place families cannot be opened for editing, so we need to skip those.
I am currently not aware of any way to examine their contents.
Here are the detailed steps:

- Select all family instances.- For each instance, determine its family.- Add an entry to the family dictionary, if not already present.- Sort the family dictionary keys.- Open each non-in-place family and search for ImportInstance elements.- List the total count and symbol names of all import instances found.

I added a new command class CmdImportsInFamilies to implement this functionality.
Here is the source code of its Execute method:

```python
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  List<Element> instances = new List<Element>();
  doc.get\_Elements( typeof( FamilyInstance ),
    instances );

  Dictionary<string, Family> families
    = new Dictionary<string, Family>();

  foreach( FamilyInstance i in instances )
  {
    Family family = i.Symbol.Family;
    if( !families.ContainsKey( family.Name ) )
    {
      families[family.Name] = family;
    }
  }

  List<string> keys = new List<string>(
    families.Keys );

  keys.Sort();

  foreach( string key in keys )
  {
    Family family = families[key];
    if( family.IsInPlace )
    {
      Debug.Print( "Family '{0}' is in-place.", key );
    }
    else
    {
      Document fdoc = doc.EditFamily( family );

      List<Element> imports = new List<Element>();
      fdoc.get\_Elements( typeof( ImportInstance ),
        imports );

      int n = imports.Count;

      Debug.Print(
        "Family '{0}' contains {1} import instance{2}{3}",
        key, n, Util.PluralSuffix( n ),
        Util.DotOrColon( n ) );

      if ( 0 < n )
      {
        foreach( ImportInstance i in imports )
        {
          Debug.Print( "  '{0}'", i.ObjectType.Name );
        }
      }
    }
  }
  return CmdResult.Failed;
}
```

As usual, we return Failed when not modifying the database in any way, so that Revit understands that no changes were made to the project and no dirty flags need to be set.

I expect that this approach may need some tweaking to achieve the exact functionality you require, but I hope it provides a useful starting point at least.

After some initial testing, Martin came up with two requests for improvement:

- One thing Im wondering about is that a family can have families nested in it. Have you considered that?
- In the attached project, the Duplex Receptacle family has a nested Annotation Symbol called Duplex Annotation.
  That Duplex Annotation has a DWG nested in it.
  How can one inspect the annotation symbol for imports?

The initial solution above does indeed not take into account nested families.

Regarding the annotation symbols, we can in fact access the families used by them by making use of the property sequence AnnotationSymbol > AsFamilyInstance > Symbol > Family and then proceeding in the same way as for standard family symbols.

In order to adapt the code to support these two enhancements, I implemented two separate helper methods:

- **GetFamilies** to retrieve all families used by the family instances and annotation symbols in the given document and return a dictionary mapping the family name to the corresponding family object.
- **ListImportsAndSearchForMore** to list all import instances in all the given families, retrieve nested families within them, and recursively search in them as well.

Here is the source code for GetFamilies, taking an input document argument and returning a dictionary as described:

```csharp
Dictionary<string, Family> GetFamilies( Document doc )
{
  Dictionary<string, Family> families
    = new Dictionary<string, Family>();

  List<Element> instances = new List<Element>();
  doc.get\_Elements( typeof( FamilyInstance ),
    instances );

  foreach ( FamilyInstance i in instances )
  {
    Family family = i.Symbol.Family;
    if ( !families.ContainsKey( family.Name ) )
    {
      families[family.Name] = family;
    }
  }

  List<Element> annotations = new List<Element>();
  doc.get\_Elements( typeof( AnnotationSymbol ),
    annotations );

  foreach ( AnnotationSymbol a in annotations )
  {
    Family family = a.AsFamilyInstance.Symbol.Family;
    if ( !families.ContainsKey( family.Name ) )
    {
      families[family.Name] = family;
    }
  }
  return families;
}
```

The two loops collect all family instances and annotation symbols and determine their families, respectively.

Here is the source code for ListImportsAndSearchForMore. Besides the input document argument and the dictionary of all families to list and explore further recursively, it also takes an argument indicating the recursion level, which is used to indent the output report appropriately according to the nesting level of the nested families:

```python
void ListImportsAndSearchForMore(
  int recursionLevel,
  Document doc,
  Dictionary<string, Family> families )
{
  string indent
    = new string( ' ', 2 \* recursionLevel );

  List<string> keys = new List<string>(
    families.Keys );

  keys.Sort();

  foreach ( string key in keys )
  {
    Family family = families[key];

    if ( family.IsInPlace )
    {
      Debug.Print( indent
        + "Family '{0}' is in-place.",
        key );
    }
    else
    {
      Document fdoc = doc.EditFamily( family );

      List<Element> imports = new List<Element>();
      fdoc.get\_Elements( typeof( ImportInstance ),
        imports );

      int n = imports.Count;

      Debug.Print( indent
        + "Family '{0}' contains {1} import instance{2}{3}",
        key, n, Util.PluralSuffix( n ),
        Util.DotOrColon( n ) );

      if ( 0 < n )
      {
        foreach ( ImportInstance i in imports )
        {
          Debug.Print( indent
            + "  '{0}'", i.ObjectType.Name );
        }
      }

      Dictionary<string, Family> nestedFamilies
        = GetFamilies( fdoc );

      ListImportsAndSearchForMore(
        recursionLevel + 1, fdoc, nestedFamilies );
    }
  }
}
```

Using these two helper functions, the source code for the mainline of the Execute method is reduced to very little:

```csharp
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  Dictionary<string, Family> families
    = GetFamilies( doc );

  ListImportsAndSearchForMore( 0, doc, families );

  return CmdResult.Failed;
}
```

Running this in a project containing one family with an import symbol and another family with nested families within it produces the following output in the debug output window displayed by Debug > Windows > Output:

```
Family 'Family1' contains 1 import instance:
  'xr.dwg'
Family 'myFamily3' contains 0 import instances.
  Family 'M_Rectangular Column' contains 0 import instances.
  Family 'myFamily1' contains 0 import instances.
    Family 'M_Rectangular Column' contains 0 import instances.
  Family 'myFamily2' contains 0 import instances.
    Family 'M_Rectangular Column' contains 0 import instances.
```

In another project containing an annotation symbol with a nested family with an import symbol, it produces the following:

```
Family 'Duplex Receptacle' contains 0 import instances.
  Family 'Duplex Annotation' contains 1 import instance:
    'star.dwg'
```

Here is
[version 1.1.0.33](zip/bc11033.zip)
of the complete Visual Studio solution with the new command.