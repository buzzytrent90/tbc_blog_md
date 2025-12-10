---
post_number: "0736"
title: "Melbourne Day One"
slug: "melbourne_day_1"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'python', 'references', 'revit-api', 'rooms', 'selection', 'transactions', 'walls']
source_file: "0736_melbourne_day_1.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0736_melbourne_day_1.html"
---

### Melbourne Day One

These days right now represent a unique time in our life history.
Venus and Jupiter are in a
[stunning conjunction](http://earthsky.org/astronomy-essentials/visible-planets-tonight-mars-jupiter-venus-saturn-mercury),
closer together right now than they will ever be again for the next two hundred years.
If you have a chance, don't forget to look up at the sky in the evenings!

![Venus Jupiter conjunction](img/venus_jupiter.jpg)

In another unique and far-away event, at least from my normal habitat, we completed the first day of the Revit API Training here in Melbourne.

Talking about being far away from my everyday habitat, I went for a walk with my hosts Kim, Rob, Erika and Lewis and their friends Geoff, Vivienne and Alice up the Anakie Gorge in the
[Brisbane Ranges National Park](http://en.wikipedia.org/wiki/Brisbane_Ranges_National_Park)
last Saturday, enjoying the wonderful Australian flora:

![Anakie Gorge](file:////j/photo/jeremy/2012/2012-03-17_anakie_gorge_melbourne/dsc03406.jpg)

Back in the city and the Autodesk training room in
[Queen's Road](http://en.wikipedia.org/wiki/Queen_Street,_Melbourne) we
are a nice mix of participants, many with significant Revit product and some Revit API experience, others with zero of both, but decades of professional application development behind them and a wish to make use of it within the Revit API.

On the first day we went through the basics of the Revit API, all of which are also covered by the
materials provided by the
[Revit Developer Center](http://www.autodesk.com/developrevit) and
described in more detail in the
[hands-on training preparation](http://thebuildingcoder.typepad.com/blog/2012/01/preparing-for-a-hands-on-revit-api-training.html) suggestions.

#### Command Instantiation and Element Picking

One little sample command that we ended up developing together simply demonstrates interactive element selection and changing the name of an element type.
In the case of a wall, the attempt to change the name of the wall itself throws an exception, because the wall instance is actually just reflecting its type name.
For other instances it throws no exception but has no effect either.
Changing the code to modify the name of the element type instead of the instance works in both these cases.

While playing around with this, one interesting and previously unanswered question that we stumbled across was on the instantiation of the external command implementation class.

In AutoCAD.NET, a separate instance of a command implementation class is created for each document, and then reused for future command invocations in that document.
How is this handled in Revit?

The code required to explore and answer this question is very simple: just implement a public constructor for the class which counts and prints out the number of invocations.
```python
  [Transaction( TransactionMode.Manual )]

  public class Command : IExternalCommand
  {
    static int \_instance\_count = 0;

    public Command()
    {
      Debug.Print( "{0} instances.",
        ++\_instance\_count );
    }

    // . . .
  }
```

As it turns out, a new instance of the command class is created every time the command is launched.

Oops.
Looking in a bit more depth, we discovered that this is in fact clearly stated in the developer guide, which says: "When a command is selected, a command object is created and its Execute() method is called. Once this method returns back to Revit, the command object is destroyed."

This confirms what every careful programmer would do subconsciously anyway: keep your command class implementation as light as possible, since it will be re-instantiated for each call of your command.

If you need to store any large amounts of data, then do so either in static variables, or, more cleanly, in separate singleton classes elsewhere.

The code we ended up with to select an element and rename its type looks like this:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  Element e;

  try
  {
    Reference r = uidoc.Selection.PickObject(
      ObjectType.Element,
      "Please pick an element." );

    e = doc.get\_Element( r.ElementId );
  }
  catch( RvtOperationCanceledException )
  {
    return Result.Cancelled;
  }

  using( Transaction tx = new Transaction( doc ) )
  {
    tx.Start( "Rename Element" );

    ElementId id = e.GetTypeId();

    Element type = doc.get\_Element( id );

    if( null != type )
    {
      type.Name = "Melbourne " + type.Name;
    }

    tx.Commit();
  }
  return Result.Succeeded;
}
```

We used this to discuss a number of basic aspects of add-in creation:

- Referencing the Revit API assemblies and
  [setting the copy local flag](http://thebuildingcoder.typepad.com/blog/2011/08/set-copy-local-to-false.html).- Implementing the basic application skeleton code.- Using attributes to define the journaling, regeneration and transaction options.
      Journaling we might return to tomorrow, regeneration is trivial, since there is only one single option nowadays, and transaction is important to understand: automatic, manual or read-only, of which I generally recommend using only the latter two.- Creating the
        [add-in manifest and GUID](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html) and
        [other add-in manifest features](http://thebuildingcoder.typepad.com/blog/2010/08/network-access-to-add-in-manifest-and-icons.html).- Using the
          [Revit add-in wizard](http://thebuildingcoder.typepad.com/blog/2011/10/product-and-add-in-wizard-updates.html) to
          handle all that automatically.- Selecting an element using PickObject and handling the exception thrown by user cancellation.- Transaction management and element modification.- Instances versus types.

#### Filtered Element Collector and Using Parameter Filter for Non-Empty String

In a second step, we had a look at a filtered element collector to access the Revit database contents.

We decided that parameter filters are of special interest, and explored how to filter for an empty and a non-empty string value.

Here is the code that we ended up with:
```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  BuiltInParameter bip
    = BuiltInParameter.ALL\_MODEL\_MARK;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  // Filter for an empty string:

  //FilterStringRuleEvaluator evaluator
  //  = new FilterStringEquals();

  // Filter for an non-empty string:

  FilterStringRuleEvaluator evaluator
    = new FilterStringGreater();

  FilterStringRule rule = new FilterStringRule(
    provider, evaluator, "", false );

  bool inverted = false;

  ElementParameterFilter filter
    = new ElementParameterFilter( rule, inverted );

  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .WherePasses( filter );

  foreach( Element e in col )
  {
    Parameter p = e.get\_Parameter( bip );

    Debug.Print( "'{0}': '{1}'",
      e.Name,
      (null==p? "null" : p.AsString() ) );
  }

  return Result.Succeeded;
}
```

In its current, final state, it uses a parameter string filter to retrieve and list all elements with a non-empty Mark parameter value.

To do so, we search for any string values greater than the empty string "".

We also tried using a null value instead of the empty string, but that throws a rather inelegant exception in the FilterStringRule constructor saying

- ArgumentNullException:
  "The input argument \"ruleString\" of function
  `anonymous-namespace'::FilterStringRule\_constructor
  or one item in the collection is null at line 1193
  of file n:\\build\\2012\_ship\_inst\_20110916\_2132
  \\source\\api\\revitapi\\gensrc\\APIFilterRule.cpp.
  \r\nParameter name: ruleString"

We also tested searching for all empty string values using a FilterStringEquals evaluator, and that worked fine as well.

#### Evernote and a Revit Product and Family Tutorial

During our explorations, we underlined the importance of families and the family API.
I mentioned the Autodesk
[Revit 2010 Families Guide](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=13080413&linkID=9243097)
[several](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html)
[times](http://thebuildingcoder.typepad.com/blog/2011/06/boolean-operations-and-instancevoidcututils.html)
[in the](http://thebuildingcoder.typepad.com/blog/2011/10/families-guide.html)
[past](http://thebuildingcoder.typepad.com/blog/2012/02/bim-versus-free-geometry-and-product-training.html).
It is free and covers the basics well together with Google, especially some important Revit MEP content best practices.

For more background, especially on Revit MEP, the
[Learning Autodesk Revit MEP 2012](http://cad-notes.com/2011/12/learning-autodesk-revit-mep-2012-training-video-is-available) video
training by Simon Whitbread,
Don Bokmiller and Joel Londenberg is recommended.

One neat little non-Revit-API topic that popped up was the handy and free little
[Evernote](http://www.evernote.com) utility
for storing and sharing notes across the cloud and various mobile devices.