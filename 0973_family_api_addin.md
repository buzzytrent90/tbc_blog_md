---
post_number: "0973"
title: "Family API Add-in – Load Family and Place Instances"
slug: "family_api_addin"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'python', 'revit-api', 'selection', 'transactions', 'views']
source_file: "0973_family_api_addin.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0973_family_api_addin.html"
---

### Family API Add-in – Load Family and Place Instances

The day before yesterday, I presented the contents of Steven Campbell's
[key family concepts](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#3) and
[parametric family](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#3b) classes
from the
[Revit API DevCamp in Moscow](http://www.autodesk.ru/adsk/servlet/pc/index?id=21516340&siteID=871736), and
an overview of the related
[family API topics](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#5) representing
my humble contribution to the latter.

If you read that post immediately after it was first published, I suggest you go back and have a quick look again now, since I revisited it and added a significant chunk of information after the initial publication.

Now I would like to delve deeper into the detailed API aspects.

Before proceeding with that, let me mention two other good bits of news:

- [Trillions – thriving in the emerging information technology](#02)
- [Visual Studio 2013 supports 64 bit edit and continue](#03)

#### Trillions – Thriving in the Emerging Information Technology

I just finished reading
[Trillions – Thriving in the Emerging Information Technology](http://eu.wiley.com/WileyCDA/WileyTitle/productCd-1118176073.html) by
Peter Lucas, Joe Ballay, Mickey McManus of
[MAYA Design](http://en.wikipedia.org/wiki/MAYA_Design).

This is the first pure technology related book I have read all the way through in ages, and I am very enthused.
It was distributed to all participants of the internal Autodesk Tech Summit in Boston and discusses design and technology for pervasive computing.

It provides numerous shockingly obvious and often quite old insights, a very critical view of our current use of Internet and so-called cloud technologies, and an inspiring and thrilling read.

The topic is how to harness and live **in** – as opposed to 'with' – the unbounded complexity of the trillion-node network of connected devices that we are inevitable heading towards, and will lead to catastrophe unless we radically change some fundamental aspects of the ways we handle and share information.

I highly recommend reading this book.

I highly recommend abstaining from all use of the word 'cloud computing' until you have completed it :-)

#### Visual Studio 2013 Supports 64 Bit Edit and Continue

Visual Studio 2013 and the CLR 4.5.1 now support 64-bit Edit and Continue when debugging C# and VB applications.
[Stephen Preston is happy](http://adndevblog.typepad.com/autocad/2013/06/coming-soon-x64-edit-and-continue.html) about that.

You can download the
[preview version of Visual Studio 2013](http://www.microsoft.com/visualstudio/eng/2013-downloads) to
test it right away.

64-bit Edit and Continue for C++ is not yet supported.
Please
[vote for 64-bit Edit and Continue for C++ to be implemented](http://visualstudio.uservoice.com/forums/121579-visual-studio/suggestions/4126415-x64-edit-and-continue-for-c-) as well.

You have three votes, and you can apply them all to this one wish, if you like.

#### Family API Add-in

Getting back to the Revit API and programmatic work with families in particular:

As I pointed out, Steve's presentation covers key family editor concepts from a user point of view.
From a programming point of view, there are two main family related areas to consider:

1. Creation of a family, i.e. working in the context of family document.
2. Use of a family, mostly in the context of project document.

Since a family instance can also be inserted into a family document to create a nested family structure, the second point is actually relevant both in the family and project context.
It includes tasks such as loading a family, placing instances, manipulating types and existing instances.

Both of these aspects represent large important topics that have been frequently discussed here in the past.

Before Revit 2010, only the second area was covered by the API.
Programmatic creation of families and the entire use of the API in the family context was impossible before the introduction of the
[Family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html).

The basic steps to programmatically create a family from scratch are demonstrated in detail in section 3, Revit Family API, of the
[ADN Revit API training material](http://thebuildingcoder.typepad.com/blog/2013/06/migrating-the-adn-training-labs-to-revit-2014.html).

The programmatic
[creation and insertion of an extrusion family](http://thebuildingcoder.typepad.com/blog/2011/06/creating-and-inserting-an-extrusion-family.html) provides
a more recent and simplified example of defining and loading a family and placing an instance without exploring the more complex and interesting aspects of the definition such as setting alignments and adding types.

The
[family API topics](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#5) concluding
the preceding post list the programming concepts that Steve and I decided to revisit:

1. Load family and place instances
2. Pick interaction and modify instances
3. Instance and symbol retrieval, nested type modification

It also provides a full snapshot of the source code and Visual Studio solution implementing the three external commands implementing these concepts and an external application wrapper defining a nice compact user interface to access and launch them:

1. CmdTableLoadPlace
2. CmdTableNewTypeModify
3. CmdKitchenUpdate

Here is the list of topics to explain their implementation and functionality in full detail:

- [Scenario 1 – load family and place instances](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#10)

- [Checking whether a family is loaded](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#11)
- [Find a database element by type and name](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#12)
- [Loading a family](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#13)
- [Placing family instances](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#14)
- [Accessing the newly placed instances](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#15)
- [Complete source code of CmdTableLoadPlace](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#16)

- [Scenario 2 – select and modify instances](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-create-type-select-and-modify-instances.html#20)

- [Creating a new family type](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-create-type-select-and-modify-instances.html#21)
- [Selecting instances with pre- and post-selection](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-create-type-select-and-modify-instances.html#22)
- [Modifying a family instance symbol](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-create-type-select-and-modify-instances.html#23)

- [API scenario 3 – instance and symbol retrieval, nested type modification](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#30)

- [Retrieve specific family symbols](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#31)
- [Retrieve specific family instances](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#32)
- [Display available door panel types](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#33)
- [Modify a nested family type](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#34)

- [External application](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#40)
- [Conclusion and download](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#50)

I'll discuss scenario 1 right here and now, and return to the rest in the next few days.

#### Scenario 1 – Load Family and Place Instances

The external command CmdTableLoadPlace demonstrates loading a family and placing family instances.

The loading requires a prior check whether the family is already present in the project.

Placing the instances can be done either completely automatically or by prompting the user for their locations.

The former makes use of one of the many
[overloads of the NewFamilyInstance method](http://thebuildingcoder.typepad.com/blog/2011/01/newfamilyinstance-overloads.html) (how to
[test all NewFamilyInstance overloads](http://thebuildingcoder.typepad.com/blog/2010/11/place-site-component.html#2)) and
is not discussed here.

The latter can be implemented using PromptForFamilyInstancePlacement, in which case the
[retrieval of the newly placed instances](http://thebuildingcoder.typepad.com/blog/2010/06/place-family-instance.html) requires
an interesting additional little twist.

#### Checking Whether a Family is Loaded

Before we can start placing any family instances, we need to load our family definition.

Loading it into the project will cause an error if it is already present beforehand, so we need to check for its prior existence before attempting.

All retrieval of database elements is performed using filtered element collectors.
Effective element filtering requires application of as many filters as possible to limit the number of elements retrieved.

Of these, the quick filters are highly preferred, since they enable filtering of elements without loading the entire element information into memory.
Slow filters are also effective, since they enable checking of element properties inside the Revit memory space before the element information is marshalled and transferred out into the .NET universe.
The slowest filtering is achieved by post-processing the element data in .NET after extracting it from Revit.

Assuming that we do not have a terribly large number of families loaded in the project, we can get by in this case by looking at all the families and applying a .NET post-processing step filtering for the desired family name, e.g. using a language integrated
[LINQ](http://thebuildingcoder.typepad.com/blog/2009/07/language-integrated-query-linq.html)
query and the generic FirstOrDefault method like this:

```csharp
  FilteredElementCollector a
    = new FilteredElementCollector( doc )
      .OfClass( typeof( Family ) );

  Family family = a.FirstOrDefault<Element>(
    e => e.Name.Equals( FamilyName ) )
      as Family;
```

#### Find a Database Element by Type and Name

We check whether the family is already loaded by retrieving all element by filtering of a specific class, also known as .NET type, in this case the Family class, and then post-processing the results looking for a given target name.

This functionality is useful for many different kinds of searches, which led to us already discussing a helper function implementing it, as well as
[possible optimisations](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html) to it.

In this add-in, for the same of simplicity, I implement it like this without any further attempts at optimisation:
```csharp
  /// <summary>
  /// Retrieve a database element
  /// of the given type and name.
  /// </summary>
  public static Element FindElementByName(
    Document doc,
    Type targetType,
    string targetName )
  {
    return new FilteredElementCollector( doc )
      .OfClass( targetType )
      .FirstOrDefault<Element>(
        e => e.Name.Equals( targetName ) );
  }
```

This add-in makes use of this helper method for three different purposes:

- Retrieve a specific family to check whether it has been loaded into the project
- Retrieve a specific family symbol to check whether it has been defined in the project
- Retrieve a specific material to apply it to a family symbol

#### Loading a Family

Once we have determined that the required family is not yet present in the project, loading it is a trivial task:

```python
  if( null == family )
  {
    // It is not present, so check for
    // the file to load it from:

    if( !File.Exists( FamilyPath ) )
    {
      Util.ErrorMsg( string.Format(
        "Please ensure that the sample table "
        + "family file '{0}' exists in '{1}'.",
        FamilyName, \_family\_folder ) );

      return Result.Failed;
    }

    // Load family from file:

    using( Transaction tx = new Transaction( doc ) )
    {
      tx.Start( "Load Family" );
      doc.LoadFamily( FamilyPath, out family );
      tx.Commit();
    }
  }
```

I added a check to ensure that the family definition file is available in the expected location.

Loading the family modifies the database, so a transaction is required.

If you only need a few symbols (also known as types) from a large family, you can load them more effectively one by one by using LoadFamilySymbol instead of loading them all at once through the single call to LoadFamily.

Here is a detailed discussion and implementation of
[loading only selected family types](http://thebuildingcoder.typepad.com/blog/2011/05/loading-only-selected-family-types.html).

#### Placing Family Instances

As said, we decided to manually place the table instances in this particular sample application, which is easily implemented using PromptForFamilyInstancePlacement.

The only input required is the symbol to be placed:

```csharp
  // Determine the family symbol

  FamilySymbol symbol = null;

  foreach( FamilySymbol s in family.Symbols )
  {
    symbol = s;

    // Our family only contains one
    // symbol, so pick it and leave

    break;
  }

  // Place the family symbol:

  // PromptForFamilyInstancePlacement cannot
  // be called inside transaction.

  uidoc.PromptForFamilyInstancePlacement( symbol );
```

Note that this method call requires that no transactions be open, presumably because Revit prefers taking care of that internally instead.

#### Accessing the Newly Placed Instances

As already noted, the
[retrieval of the instances placed](http://thebuildingcoder.typepad.com/blog/2010/06/place-family-instance.html) by
the call to PromptForFamilyInstancePlacement requires an interesting additional little twist.

Theoretically, this method could return a list of the instance element ids.
Lacking that, we can find out for ourselves by temporarily registering to the DocumentChanged event before the call, unregistering immediately afterwards, and making a note of all element ids added in between.

Here is the OnDocumentChanged event handler and the element id container I use for that:

```csharp
  /// <summary>
  /// Collection of newly added family instances
  /// </summary>
  List<ElementId> \_added\_element\_ids
    = new List<ElementId>();
  void OnDocumentChanged(
    object sender,
    DocumentChangedEventArgs e )
  {
    \_added\_element\_ids.AddRange(
      e.GetAddedElementIds() );
  }
```

Here is the full source code showing the event registration details to activate this.

#### Complete Source Code of CmdTableLoadPlace

For completeness sake, here is the full source code of the CmdTableLoadPlace external command showing the steps described above in their complete glorious, elegant and cooperative context:

```python
/// <summary>
/// Load table family if not already present and
/// place table family instances.
/// </summary>
[Transaction( TransactionMode.Manual )]
public class CmdTableLoadPlace : IExternalCommand
{
  /// <summary>
  /// Family name.
  /// </summary>
  public const string FamilyName = "family\_api\_table";

  /// <summary>
  /// Family file path.
  /// Normally, you would either search the  library
  /// paths provided by Application.GetLibraryPaths
  /// method. In this case, we store the sample
  /// family in the same location as the add-in.
  /// </summary>
  //const string \_family\_folder = "Z:/a/rvt";
  static string \_family\_folder
    = Path.GetDirectoryName(
      typeof( CmdTableLoadPlace )
        .Assembly.Location );

  /// <summary>
  /// Family filename extension RFA.
  /// </summary>
  const string \_family\_ext = "rfa";

  /// <summary>
  /// Family file path
  /// </summary>
  static string \_family\_path = null;

  /// <summary>
  /// Return complete family file path
  /// </summary>
  static string FamilyPath
  {
    get
    {
      if( null == \_family\_path )
      {
        \_family\_path = Path.Combine(
          \_family\_folder, FamilyName );

        \_family\_path = Path.ChangeExtension(
          \_family\_path, \_family\_ext );
      }
      return \_family\_path;
    }
  }

  /// <summary>
  /// Collection of newly added family instances
  /// </summary>
  List<ElementId> \_added\_element\_ids
    = new List<ElementId>();

  /// <summary>
  /// External command mainline
  /// </summary>
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    // Retrieve the family if it is already present:

    Family family = Util.FindElementByName(
      doc, typeof( Family ), FamilyName ) as Family;

    if( null == family )
    {
      // It is not present, so check for
      // the file to load it from:

      if( !File.Exists( FamilyPath ) )
      {
        Util.ErrorMsg( string.Format(
          "Please ensure that the sample table "
          + "family file '{0}' exists in '{1}'.",
          FamilyName, \_family\_folder ) );

        return Result.Failed;
      }

      // Load family from file:

      using( Transaction tx = new Transaction( doc ) )
      {
        tx.Start( "Load Family" );
        doc.LoadFamily( FamilyPath, out family );
        tx.Commit();
      }
    }

    // Determine the family symbol

    FamilySymbol symbol = null;

    foreach( FamilySymbol s in family.Symbols )
    {
      symbol = s;

      // Our family only contains one
      // symbol, so pick it and leave

      break;
    }

    // Place the family symbol:

    // Subscribe to document changed event to
    // retrieve family instance elements added by the
    // PromptForFamilyInstancePlacement operation:

    app.DocumentChanged
      += new EventHandler<DocumentChangedEventArgs>(
        OnDocumentChanged );

    \_added\_element\_ids.Clear();

    // PromptForFamilyInstancePlacement cannot
    // be called inside transaction.

    uidoc.PromptForFamilyInstancePlacement( symbol );

    app.DocumentChanged
      -= new EventHandler<DocumentChangedEventArgs>(
        OnDocumentChanged );

    // Access the newly placed family instances:

    int n = \_added\_element\_ids.Count();

    string msg = string.Format(
      "Placed {0} {1} family instance{2}{3}",
      n, family.Name,
      Util.PluralSuffix( n ),
      Util.DotOrColon( n ) );

    string ids = string.Join( ", ",
      \_added\_element\_ids.Select<ElementId, string>(
        id => id.IntegerValue.ToString() ) );

    Util.InfoMsg2( msg, ids );

    return Result.Succeeded;
  }

  void OnDocumentChanged(
    object sender,
    DocumentChangedEventArgs e )
  {
    \_added\_element\_ids.AddRange(
      e.GetAddedElementIds() );
  }
}
```

As said, the detailed description of the other two commands will follow soon, and the complete Visual Studio solution is available from the overview of the
[family API topics](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#5) concluding
the preceding post.

I wish you a wonderful weekend.

---

# Cloud and Mobile

### Trillions – Thriving in the Emerging Information Technology

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

I just finished reading
[Trillions – Thriving in the Emerging Information Technology](http://eu.wiley.com/WileyCDA/WileyTitle/productCd-1118176073.html) by
Peter Lucas, Joe Ballay, Mickey McManus of
[MAYA Design](http://en.wikipedia.org/wiki/MAYA_Design),
distributed to all Tech Summit presenters.

It discusses design and technology for pervasive computing, provides numerous shockingly obvious and often quite old insights, a very critical view of our current use of Internet and so-called cloud technologies, and an inspiring and thrilling read.

The topic is how to harness and live **in** – as opposed to 'with' – the unbounded complexity of the trillion-node network of connected devices that we are inevitable heading towards, and will lead to catastrophe unless we radically change some fundamental aspects of the ways we handle and share information.

I highly recommend reading this book.

I highly recommend abstaining from all use of the word 'cloud computing' until you have completed it :-)