---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.9
content_type: code_example
optimization_date: '2025-12-11T11:44:13.574020'
original_url: https://thebuildingcoder.typepad.com/blog/0220_door_marks.html
post_number: '0220'
reading_time_minutes: 4
series: general
slug: door_marks
source_file: 0220_door_marks.htm
tags:
- doors
- elements
- family
- filtering
- parameters
- python
- revit-api
- selection
- transactions
title: Door Marks
word_count: 806
---

### Door Marks

Here is a question from Håvard Dagsvik of
[Cad Quality as](http://www.cad-q.com/no)
on door marks.
It can obviously be generalised to other model marks in the BIM as well.

**Question:** One of the best documentation features of Revit is the function assigning numeric values to the Mark parameters.
If you insert 5 doors they are marked 1-5.
Now if door 3 and 5 are deleted, then a new door will still start its mark on 6.
Even if the project is closed and reopened, the next door still gets Mark# 6.
That means there has to be a list inside Revit somewhere of the mark numbers that previously has been used, and Revit avoids reusing those numbers again.
Is there a way through the API to get to that list?
And somehow manipulate it to make Revit start counting on our choice of number?

**Answer:** Using the
[RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html) BIM
debugging and analysis tool or the Revit API introduction labs
[built-in parameter checker](http://thebuildingcoder.typepad.com/blog/2009/04/deeper-parameter-exploration.html) to
explore various element parameters, I see that the Mark parameter is accessible through the built-in parameters ALL\_MODEL\_MARK.
This is a read-write parameter:

```
ALL_MODEL_MARK  Mark  String/Text  read-write  4
```

You can therefore obtain a list of all door marks by filtering for all door instances and reading their ALL\_MODEL\_MARK parameter value.

I do not believe that you can manipulate Revit to influence the automatic mark counting setting used by the automatic assignment, but what you certainly can do is implement an external command which renumbers existing door instances by changing the value of their ALL\_MODEL\_MARK parameter value in any way you please.

To demonstrate reading and listing all existing door marks as well as modifying the marks of existing doors, I created a new Building Coder sample command CmdListMarks, which implements the following functionality:

- Select all door instances in the model.- Iterate over the selected door instances.- Create a dictionary whose keys are all door marks in the model and mapping each mark to a list of all doors with that mark.- Print out the sorted dictionary contents.- If any door elements have been selected prior to running the command, modify their door marks to a predefined hardcoded value "42".

Here is the output produced by running this command in a small model with four doors with the marks 1, 2 and 4, where 4 is duplicated on two doors.

```
4 doors found.
3 door marks found:
  1: 1 door
  2: 1 door
  4: 2 doors
```

If I select the two doors marked 4 and run the command again, their marks are modified to 42.
Running it yet again lists the new mark settings:

```
4 doors found.
3 door marks found:
  1: 1 door
  2: 1 door
  42: 2 doors
```

Here is the complete code of the new external command:
```python
static bool \_modify\_existing\_marks = true;
const string \_the\_answer = "42";

public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  Autodesk.Revit.Creation.Filter cf
    = app.Create.Filter;

  List<Element> doors
    = new List<Element>();

  Filter f1 = cf.NewTypeFilter(
    typeof( FamilyInstance ) );

  Filter f2 = cf.NewCategoryFilter(
    BuiltInCategory.OST\_Doors );

  Filter f = cf.NewLogicAndFilter( f1, f2 );

  int n = doc.get\_Elements( f, doors );

  Debug.Print( "{0} door{1} found.",
    n, Util.PluralSuffix( n ) );

  if( 0 < n )
  {
    Dictionary<string, List<Element>> marks
      = new Dictionary<string, List<Element>>();

    foreach( FamilyInstance door in doors )
    {
      string mark = door.get\_Parameter(
        BuiltInParameter.ALL\_MODEL\_MARK )
        .AsString();

      if( !marks.ContainsKey( mark ) )
      {
        marks.Add( mark, new List<Element>() );
      }
      marks[mark].Add( door );
    }

    List<string> keys = new List<string>(
      marks.Keys );

    keys.Sort();

    n = keys.Count;

    Debug.Print( "{0} door mark{1} found{2}",
      n, Util.PluralSuffix( n ),
      Util.DotOrColon( n ) );

    foreach( string mark in keys )
    {
      n = marks[mark].Count;

      Debug.Print( "  {0}: {1} door{2}",
        mark, n, Util.PluralSuffix( n ) );
    }
  }

  n = 0; // count how many elements are modified

  if( \_modify\_existing\_marks )
  {
    ElementSet els = doc.Selection.Elements;

    foreach( Element e in els )
    {
      if( e is FamilyInstance
        && null != e.Category
        && (int) BuiltInCategory.OST\_Doors
          == e.Category.Id.Value )
      {
        e.get\_Parameter(
          BuiltInParameter.ALL\_MODEL\_MARK )
          .Set( \_the\_answer );

        ++n;
      }
    }
  }

  // return Succeeded only if we wish to commit
  // the transaction to modify the database:

  return 0 < n
    ? CmdResult.Succeeded
    : CmdResult.Failed;
}
```

The command returns Succeeded only if the BIM has actually been modified, otherwise Failed.
This ensures that the document dirty flag stored in its
[IsModified property](http://thebuildingcoder.typepad.com/blog/2008/12/document-ismodified-property.html) is
not set unnecessarily.

Here is
[version 1.1.0.47](zip/bc11047.zip)
of the complete Building Coder sample source code and Visual Studio solution including the new command.