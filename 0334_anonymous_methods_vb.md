---
post_number: "0334"
title: "Anonymous Methods in VB"
slug: "anonymous_methods_vb"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'python', 'revit-api', 'transactions', 'vbnet']
source_file: "0334_anonymous_methods_vb.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0334_anonymous_methods_vb.html"
---

### Anonymous Methods in VB

By the time you read this, I will be well on my way to the Revit API training that I am giving in Warsaw.
The Revit API introduction labs that I am using for that include a few examples of post-processing the results of a Revit filtered collector query.
I discussed several examples of such post-processing before Easter when
[profiling the collector performance](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html).
I and my colleagues were having some trouble converting the code from C# to VB, and I thought I would share some of our insights with you here.
For instance, our command Lab3\_7\_DeleteFamilyType selects and deletes a specific hard-coded column type named "475 x 610mm":

![Column types](img/column_types.png)

We were using the following C# code and helper methods:

- GetElementsOfType –
  Return all elements of the requested class,
  i.e. System.Type, matching the given built-in
  category in the given document.- GetFamilySymbols –
    Return all family symbols in the given document
    matching the given built-in category.

Here are the helper method implementations:
```csharp
public static FilteredElementCollector
  GetElementsOfType(
    Document doc,
    Type type,
    BuiltInCategory bic )
{
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfCategory( bic );
  collector.OfClass( type );

  return collector;
}
public static FilteredElementCollector
  GetFamilySymbols(
    Document doc,
    BuiltInCategory bic )
{
  return GetElementsOfType( doc,
    typeof( FamilySymbol ), bic );
}
```

Here is the trivial command mainline Execute method implementation making use of these to first retrieve all family symbols of the specified category, and then post-process the results to extract one specific instance matching the hard-coded name "475 x 610mm" and delete that specific individual type from the column family.

Actually, since we are still in the process of getting used to the new Revit 2011 API attributes and other paraphernalia, here is the complete command class implementation:
```python
[Transaction( TransactionMode.Automatic )]
[Regeneration( RegenerationOption.Manual )]
public class Lab3\_7\_DeleteFamilyType : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    FilteredElementCollector collector
      = LabUtils.GetFamilySymbols( doc,
        BuiltInCategory.OST\_Columns );

    var column\_types = from element in collector
      where element.Name.Equals( "475 x 610mm" )
      select element;

    FamilySymbol symbol = column\_types
      .Cast<FamilySymbol>()
      .First<FamilySymbol>();

    doc.Delete( symbol );

    return Result.Succeeded;
  }
}
```

As you can see, we are still deriving the command class from our beloved IExternalCommand interface.
We are also adding the new non-optional attributes defining its transaction mode and regeneration option.

GetFamilySymbols returns a FilteredElementCollector, and we process that collection using
[LINQ](http://thebuildingcoder.typepad.com/blog/2009/07/language-integrated-query-linq.html)
to extract the one and only symbol matching our target name in order to demonstrate deleting the symbol from the model.

The same code ported to VB.NET looks like this:
```vbnet
<Transaction(TransactionMode.Automatic)> \_
<Regeneration(RegenerationOption.Manual)> \_
Public Class Lab3\_7\_DeleteFamilyType
  Implements IExternalCommand
  Public Function Execute( \_
    ByVal commandData As ExternalCommandData, \_
    ByRef message As String, \_
    ByVal elements As ElementSet) As Result \_
    Implements IExternalCommand.Execute
    Dim app As UIApplication = commandData.Application
    Dim doc As Document = app.ActiveUIDocument.Document
    Dim collector As FilteredElementCollector \_
      = LabUtils.GetFamilySymbols( \_
        doc, BuiltInCategory.OST\_Columns)
    Dim column\_types = From element In collector \_
      Where element.Name.Equals("475 x 610mm") \_
      Select element
    Dim column\_types\_ienum As IEnumerable(Of Element)
    column\_types\_ienum = CType(column\_types, IEnumerable(Of Element))
    Dim column\_types\_famsym As IEnumerable(Of FamilySymbol)
    column\_types\_famsym = column\_types\_ienum.Cast(Of FamilySymbol)()
    Dim symbol As FamilySymbol = column\_types\_famsym.First()
    doc.Delete(symbol)
    Return Result.Succeeded
  End Function
End Class
```

We added several extraneous lines of casting code to circumvent some runtime casting exceptions thrown when executing the VB code.

Since I did not like all those extra lines of casting code, I suggested making use of an anonymous method instead of the explicit LINQ statement.
Here is my suggestion to reduce the length of code and casts.
Please note that it does not work, as we will explain below:

```csharp
  Dim collector As FilteredElementCollector \_
    = LabUtils.GetFamilySymbols( \_
      doc, BuiltInCategory.OST\_Columns)
  Dim name\_equals = Function(e) e.Name.Equals("475 x 610mm")
  Dim element As Element = collector.First(name\_equals)
  Dim symbol As FamilySymbol = CType(element, FamilySymbol)
  doc.Delete(symbol)
```

This is much shorter and easy to read.
It also compiles perfectly well.
Unfortunately, it throws the following exception during runtime:

```
Unable to cast object
  of type 'VB$AnonymousDelegate_0`2[System.Object,System.Boolean]'
  to type 'System.Func`2[Autodesk.Revit.DB.Element,System.Boolean]'.
```

What does this mean?
Well, apparently the definition of 'name\_equals' using 'Function(e)' is generating an anonymous VB delegate, whereas the generic First method is expecting a more specialised System.Func delegate.
As a first step, I replaced 'Function(e)' by 'Function(e As Element)'.
That improved things somewhat, because now at least the argument has the correct type, but the delegate is still an anonymous VB delegate and not a System.Func one.
In a second step, I corrected that as well, and now I have the following, which works fine and is nice and short:

```csharp
  Dim name\_equals As Func(Of Element, Boolean) \_
    = Function(e As Element) e.Name.Equals("475 x 610mm")
```

This prompted me to return to the C# version and try to shorten that a bit more as well.
My first step was to replace the from-where-select statement by the generic algorithm First, like this:

```csharp
  FilteredElementCollector collector
    = LabUtils.GetFamilySymbols( doc,
      BuiltInCategory.OST\_Columns );

  FamilySymbol symbol = collector.First<Element>(
    e => e.Name.Equals( "475 x 610mm" ) )
      as FamilySymbol;

  doc.Delete( symbol );
```

On second thoughts, I noticed that the doc.Delete method does not really care what kind of element I am passing in, so there is no need even to cast the retrieved symbol from Element to FamilySymbol, i.e. this shorter code does the job equally well:

```csharp
  FilteredElementCollector collector
    = LabUtils.GetFamilySymbols( doc,
      BuiltInCategory.OST\_Columns );

  Element symbol = collector.First<Element>(
    e => e.Name.Equals( "475 x 610mm" ) );

  doc.Delete( symbol );
```

Finally, let's take that insight back to the VB version and minimalise that also as follows:

```csharp
  Dim collector As FilteredElementCollector \_
    = LabUtils.GetFamilySymbols( \_
      doc, BuiltInCategory.OST\_Columns)
  Dim name\_equals As Func(Of Element, Boolean) \_
    = Function(e As Element) e.Name.Equals("475 x 610mm")
  Dim symbol As Element = collector.First(name\_equals)
  doc.Delete(symbol)
```

Even more finally, we can make true use of the anonymous function facility in VB as well and eliminate the intermediate named functor like this:

```csharp
  Dim symbol As Element = collector.First( \_
    Function(e As Element) e.Name.Equals("475 x 610mm"))
```

Now we have the same succinct sweetness in both versions.

As we noticed when
[benchmarking collector performance](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html),
the speed of the anonymous method is exactly the same as using LINQ.

More importantly, what we also noticed was that using a parameter filter and the Revit filtering API instead of the explicit post-processing we are discussing here was much faster still, by almost a factor of two.
The fact is that filtering for a specific element name can just as well be achieved by a parameter filter, and an example was given in that discussion, and it used half the time that the comparable LINQ query does.

Still, I hope this helps make the anonymous functions and the power of the Revit 2011 filtering API more accessible to all you VB folks as well.
Good luck and much success with that to you!