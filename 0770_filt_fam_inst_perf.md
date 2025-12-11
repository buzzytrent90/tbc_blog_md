---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.5
content_type: code_example
optimization_date: '2025-12-11T11:44:14.557278'
original_url: https://thebuildingcoder.typepad.com/blog/0770_filt_fam_inst_perf.html
post_number: '0770'
reading_time_minutes: 8
series: general
slug: filt_fam_inst_perf
source_file: 0770_filt_fam_inst_perf.htm
tags:
- csharp
- elements
- family
- filtering
- revit-api
- vbnet
- views
title: Family Usage Filtered Element Collector Performance
word_count: 1509
---

### Family Usage Filtered Element Collector Performance

Since all access to the Revit database and its elements takes place through the filtered element collectors, it is extremely important to make efficient use of them.
We have not looked at that topic for quite a while now.
Here is an issue that recently cropped up which illustrates some interesting points.

I presented a filtered element sample for creating an
[XML family usage report](http://thebuildingcoder.typepad.com/blog/2010/12/xml-family-usage-report.html) which
iterates over all the loaded families and launches a new filter for each of them looking for its instances.

If you are only interested in families of certain categories, and furthermore only in ones that actually have instances in the model, then the approach presented there will almost certainly turn out to be inefficient in a large model.

That leads to the following query:

**Question:** I tried to get a list of families which have instance in the model (not all families), but the performance is very bad if the model has loaded hundreds of family types.

Here is the VB.NET code (copy and paste to an editor to see the truncated lines in full):
```vbnet
Sub GetFamilyType(ByVal mydoc As Document)
  Dim collector As New Autodesk.Revit.DB.FilteredElementCollector(mydoc)
  Dim collection As ICollection(Of Autodesk.Revit.DB.Element) = collector.OfClass(GetType(DB.Family)).ToElements()
  ' Get all families belong to a Category of current document
  Dim families As List(Of Family)
  Dim category As DB.Category = Nothing
  'Dim familylist As New Dictionary(Of String, Family)
  Dim familyTypeCategoryList As New Dictionary(Of String, List(Of Family)) 'Family list with Family Instance in the current model
  Dim categoryList As New Dictionary(Of String, Category) 'Category list with Family Instance in the current mode
  For Each family As DB.Family In collection
    category = Nothing
    If family.Symbols.Size > 0 Then
      Dim symbols As FamilySymbolSetIterator = family.Symbols.ForwardIterator()
      symbols.Reset()
      Dim symbol As Autodesk.Revit.DB.FamilySymbol = Nothing
      While symbols.MoveNext()
        symbol = TryCast(symbols.Current, Autodesk.Revit.DB.FamilySymbol)
        '-------------------------
        'Bad performance with following code when the model has hundreds of family types
        'Anyway to improve it?
        '-------------------------
        'Find the symbol of which at least a Family instance exists
        Dim filter As New DB.FamilyInstanceFilter(mydoc, symbol.Id)
        collector = New Autodesk.Revit.DB.FilteredElementCollector(mydoc)
        Dim familyInstances As ICollection(Of DB.Element) = collector.WherePasses(filter).ToElements()
        If familyInstances.Count > 0 Then
          'All symbols of a family are in the same category, different family might in then same category
          category = symbol.Category
          Exit While
        End If
      End While
      If category IsNot Nothing Then
        If familyTypeCategoryList.ContainsKey(category.Name) Then
          familyTypeCategoryList.Item(category.Name).Add(family)
        Else
          families = New List(Of DB.Family)
          families.Add(family)
          familyTypeCategoryList.Add(category.Name, families)
          categoryList.Add(category.Name, category)
        End If
      End If
    End If
  Next
End Sub
```

Is there a quick filter to retrieve only the families having instances in the model?

**Answer:** There are a few ways that this can be improved directly, and there are also some alternative approaches which may or may not improve performance, and which you would have to test yourself in your own environment using your real world models.

Remember to always
[benchmark your add-in performance](http://thebuildingcoder.typepad.com/blog/2012/01/timer-code-for-benchmarking.html).

One thing that I note which will cost you some unnecessary time is the conversion of collectors to collections, for instance:
```csharp
  Dim collection As ICollection(
      Of Autodesk.Revit.DB.Element)
    = collector.OfClass(GetType(DB.Family))
      .ToElements()
```

There is no need for this conversion; you can iterate over the collector itself directly, so there is no reason to create a collection, which requires memory allocation and a conversion overhead.

Eliminating the conversion to collections will only have a small impact, though.
Try it out; benchmark this and let me know the result, please.

Much more significantly, you are first collecting all the families in the model, and then launching a separate new filter for each in a loop to find all of its potential instances.

For each of the families, if any instance is found, you retrieve its category and check whether it is one of the categories of interest to you.

If not, or no instances were found, the family is skipped and all the effort spent on that family so far was wasted.

I would suggest a completely different algorithm:

1. Define the list of categories of interest to you.- Implement one single family instance filter for all of these categories.- Iterate over the result and determine which families they belong to, or which symbols they use.- If you use these families or symbols (or their names) as dictionary keys, you will create the desired list.- The dictionary value for each can be something minimal, like a count of all the family instances of each type, or larger, like a list of all the instance element ids.

The first two steps are illustrated in several places on the blog, for instance in the examples to retrieve
[MEP elements, connectors](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html), and
[structural elements](http://thebuildingcoder.typepad.com/blog/2010/07/retrieve-structural-elements.html).

I'm sure the remaining steps are demonstrated somewhere as well.

You can also look at the numerous other
[filtered element collector samples](http://thebuildingcoder.typepad.com/blog/2010/12/filtered-element-collector-sample-overview.html).

The
[XML family usage report](http://thebuildingcoder.typepad.com/blog/2010/12/xml-family-usage-report.html)
shows how to iterate specifically over families, which may however not help much in your case, since it looks at all families, regardless of their categories and whether they have any instances in the model.

It is thus similar to your original approach.

Actually, looking at your code, I think it may be based on that very post :-)

I hope this solves the problem for you... I think it will, actually.

**Response:** Thank you for your suggestion.
Step 1 does not apply to our case since we don't know the categories until after we filter the family instances in the model.

Here is the code that we ended up with:
```vbnet
Public Shared Sub CreateFamilyTreeTest( \_
  ByVal myDoc As Document)
  Dim collector As New FilteredElementCollector(myDoc)
  collector.WhereElementIsNotElementType()
  Dim familyInstances = From elem In collector \_
    Where elem.Category IsNot Nothing \_
    And TypeOf elem Is FamilyInstance()
  Dim familySymbols = From elem In collector \_
    Join sb In familyInstances \_
    On elem.UniqueId Equals CType(sb.ObjectType,
      FamilySymbol).Family.UniqueId \_
    Where TypeOf elem Is Family Select elem
  familySymbols = familySymbols.Distinct()
  Dim mapCatToFam As \_
    New Dictionary(Of String, List(Of Family))
  Dim categoryList As \_
    New Dictionary(Of String, Category)
  Dim families As List(Of Family)
  Dim category As Category
  Dim symbol As FamilySymbol
  For Each family As Family In familySymbols
    If family.Symbols.Size > 0 Then
      symbol = family.Symbols(0)
      category = symbol.Category
      If mapCatToFam.ContainsKey(category.Name) Then
        mapCatToFam.Item(category.Name).Add(family)
      Else
        families = New List(Of Family)
        families.Add(family)
        mapCatToFam.Add(category.Name, families)
        categoryList.Add(category.Name, category)
      End If
    End If
  Next
End Sub
```

It uses LINQ to improve the performance and takes about 10 seconds compared to the original 3-5 minutes.

**Addendum:**
[Guy Robinson](mailto:info@r-e-d.co.nz)
[suggested below](http://thebuildingcoder.typepad.com/blog/2012/05/family-usage-filtered-element-collector-performance.html?cid=6a00e553e1689788330168eba6f389970c#comment-6a00e553e1689788330168eba6f389970c)
making use of the .NET generic GroupBy method to simplify this further as follows:
```csharp
public static void CreateFamilyTreeTest(
  Document myDoc )
{
  IEnumerable<Element> familiesCollector =
    new FilteredElementCollector( myDoc )
      .OfClass( typeof( FamilyInstance ) )
      .WhereElementIsNotElementType()
      .Cast<FamilyInstance>()
      // (family, familyInstances):
      .GroupBy( fi => fi.Symbol.Family )
      .Select( f => f.Key );

  var mapCatToFam = new Dictionary<string,
    List<Element>>();

  var categoryList = new Dictionary<string,
    Category>();

  foreach( var f in familiesCollector )
  {
    var catName = f.Category.Name;

    if( mapCatToFam.ContainsKey( catName ) )
    {
      mapCatToFam[catName].Add( f );
    }
    else
    {
      mapCatToFam.Add( catName,
        new List<Element> { f } );

      categoryList.Add( catName,
        f.Category );
    }
  }
}
```

If you are interested in some background on the GroupBy method and an encouragement to use it whenever possible, here is a short
[introduction and tutorial](http://geekswithblogs.net/Martinez/archive/2009/10/29/linq-groupby-method-tutorial.aspx).

#### Class Property Initialiser

As one thing leads to another, I wondered about the missing pair of parenthesis in the new List initialiser above, and asked Guy about that.

Guy responds: When using collection initialisers the parentheses () are optional.
The code will compile and run both with and without them.

So 'new List<Element> { f, f };' is the same as writing:
```csharp
  var list = new List<Element>();
  list.Add( f );
  list.Add( f );
```

You can also use initialisers for class properties.
So say I have a class like this:
```csharp
  public class Klass
  {
    public int Prop1 { get; set; }
    public string Prop2 { get; set; }
  }
```

A new instance of that class can be initialised like this:
```csharp
  var klass = new Klass
  {
    Prop1 = 23,
    Prop2 = "some string"
  };
```

Saves a bit of typing ;-)

Many thanks to Guy for all his interesting hints and explanations!