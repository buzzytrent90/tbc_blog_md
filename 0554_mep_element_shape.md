---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.6
content_type: code_example
optimization_date: '2025-12-11T11:44:14.162125'
original_url: https://thebuildingcoder.typepad.com/blog/0554_mep_element_shape.html
post_number: '0554'
reading_time_minutes: 10
series: elements
slug: mep_element_shape
source_file: 0554_mep_element_shape.htm
tags:
- csharp
- elements
- family
- parameters
- python
- revit-api
- selection
- transactions
title: Distinguishing MEP Element Shape
word_count: 2047
---

### Distinguishing MEP Element Shape

Normally, when given a family instance, you can access the family name to obtain some information about it.
For Revit MEP ducts, however, the duct shape is determined by an internal system family whose name is visible in the user interface but is not accessible through the API.

For instance, with ducts, you cannot access the family of a round duct.
In the user interface, it appears to belong to a family named "Round Duct", but in fact this built-in system family and its name are not accessible through the API.

Fittings are represented by normal standard families, so this problem does not apply to them, only to ducts with the BuiltInCategory OST\_DuctCurves
.

Several people have asked for and probably implemented workarounds for this.
Here is one solution by Max, [Maciej Szlek](http://maciejszlek.pl), based on an analysis of the string value of the duct's Size parameter.
He starts off by asking:

**Question:** I wonder if there is some way to clearly determine the part type of a ductwork element, i.e. round, oval or rectangular.
How to distinguish this?
I can do it by regular expression matching for size parameter, but there must be a better way.

**Answer:** Unfortunately, I am not aware of any good solution for this.
What does your solution look like?

**Response:** I assumed that if the family name like "round duct" is shown in Revit it would also be accessible through the API. As I see, that was a hasty assumption :)

To my work around: I simply noticed that the size parameter is displayed in a manner sufficient to clearly identify most duct elements.
Here is a code snippet to explain exactly what I mean:
```csharp
  if( size.Split( 'x' ).Length == 3 ) // could use a regex "[0-9]x[0-9]+-[0-9]+/[0-9]+" but splitting is less costly
    return "rectangular2rectangular";
  else if( size.Split( '/' ).Length == 3 )
    return "oval2oval";
  else if(
    new Regex( @"[0-9]+x[0-9]+-[0-9]+/[0-9]+" )
      .IsMatch( size ) )
        return "rectangular2oval";
  else if(
    new Regex( @"[0-9]+/[0-9]+-[0-9]+x[0-9]+" )
      .IsMatch( size ) )
        return "oval2rectangular";
  else if(
    new Regex( @"[0-9]+[^0-9]-[0-9]+x[0-9]+" )
      .IsMatch( size ) )
        return "round2rectangular";
  else if(
    new Regex( @"[0-9]+x[0-9]+-[0-9]+[^0-9]" )
      .IsMatch( size ) )
        return "rectangular2round";
  else if(
    new Regex( @"[0-9]+[^0-9]-[0-9]+/[0-9]+" )
      .IsMatch( size ) )
        return "round2oval";
  else if(
    new Regex( @"[0-9]+/[0-9]+-[0-9]+[^0-9]" )
      .IsMatch( size ) )
        return "oval2round";
  else if(
    new Regex( @"[0-9]+[^0-9]-[0-9]+[^0-9]" )
      .IsMatch( size ) )
        return "round2round";
  else { return "other case"; }
```

By the way, is there any convenient way to check if some parameter exists?
Element.get\_Parameter("some\_param") throws an exception if parameter doesn't exist.
There is a method Element.Parameters.Contains, but I don't know how to use it for a given parameter name.
I want to check if an element has my shared parameter attached.

**Answer:** Thank you for your interesting sample code.
Now I see what you mean by regular expressions, of course.

I have one possible enhancement suggestion to make: if you call this method many times, I would suggest compiling the regular expressions and caching the compiled version instead of re-instantiating them all on each call.

Regarding checking whether a parameter exists, I always suggest using built-in parameters as much as possible.
If you call Element.get\_Parameter( BuiltInParameter ) with an enum value that does not exist on the given element, it simply returns null without throwing an exception, I believe.
Of course that will not work for family parameters with no corresponding built-in parameter enum value. You could also try finding the Definition class instance representing the named parameter and calling Element.get\_Parameter( Definition ).
That might return null as well instead of throwing an exception, but I don't know for sure.

By the way, your code includes the following expression:
```csharp
  return e.Category.Id.Equals(
    e.Document.Settings.Categories.get\_Item(
      c ).Id );
```

This can be significantly shortened and also implemented more effectively by using the
[category comparison casting the built-in category to an integer](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html):
```csharp
  return e.Category.Id.IntegerValue.Equals(
    (int) c );
```

I implemented a new Building Coder sample command CmdMepElementShape to test your method, and included the regular expression caching that I suggested.
It fails for imperial units, though.

**Response:** Hmm, I didn't think about imperial units.
I work with the metric unit system and the regular expressions in my function are compatible with this.
I extended it to work with imperial units as well, for some selected part types.
There was not that much to modify, actually.

There are quite a lot part types, though, and many are not handled yet.

I think it would be sufficient to make this function work fully correctly only for transitions and elbows, to start with.
That will demonstrate the main principle.
If someone needs to handle other part types, it can be done analogously.

I couldn't switch my Revit to imperial units, so I'm not 100% sure if my regular expressions will work correctly.
I could switch parameters like Length in Manage > Project Units, but I couldn't switch Size.
I read that I can choose units system during Revit installation, so I tried to do this with a fresh installation, but I couldn't switch units in "Configure" because the option was blocked – maybe because it saw that country was Poland, I don't know.
I even downloaded the newest update especially, and both metric and imperial templates – with no result.

Therefore, I don't know how Size parameter can be displayed by Revit in other than metric units – as "fractional inches" or "decimal inches" or else... and I had to write these regular expressions blindly.
In metric units, for example, the Size parameter is always rounded to millimetres.

I would be grateful if you check them again and where possible improve them, because that would be useful information for me as well.
The regular expressions for metric units, as previously, work fine.
The important thing is that the principles be clear for everyone.

Thinking further about it, I noticed two disadvantages of this solution.

The first one is that it has to contain regular expressions handling all Size parameter display cases due to the different units (optimistically only two: metric/imperial, pessimistically: many cases from "project units" formats – I don't know Revit well enough yet to be sure how it looks like but looking at metric example I think that would be the optimistic one).

Secondly, it returns the element shape well, but if one needs information about shape changes in the flow direction (like me), additional information about at least the preceding element is needed.

Here is a screen snapshot from a fragment of a sample project to show what I mean:
![Duct shape sequence](img/duct_shape_sequence.png)

At first I assumed optimistically that the Size parameter value is dependent on the sequence in the flow direction, but it isn't.
The shape is returned from the family definition.
I tried to find some other parameter responsible for storing information about element's rotation in relation to flow direction or even the preceding path element but I only found parameters storing rotation/facing orientation/hand orientation in relation to absolute coordinates.
Actually, this doesn't have to be a disadvantage at all – it depends on the application. :)

You told me that the Element.get\_Parameter method doesn't throw an exception for BuiltInParameters. This could be good solution for the first disadvantage (elements of different part types has different set of parameters), because it is independent on units system but it has its own weakness: we lose information about element shapes sequence completely.

I think the best solution would be using mix of the two above methods: checking existence of specific parameters for specified part type/category and the Size parameter analysis.

**Answer:** Some additional information on flow direction is available from the element connectors.

Actually, come to think of it, the shape of the duct is available from the connectors as well.
That might be a much more reliable and effective method to address this issue.
Sorry for thinking of this so late.

Anyway, here is the complete code of the CmdMepElementShape sample command in its current state.
First, we have the regular expression cache implementation that I mentioned above:
```python
class RegexCache : Dictionary<string, Regex>
{
  /// <summary>
  /// Apply regular expression pattern matching
  /// to a given input string. The compiled
  /// regular expression is cached for efficient
  /// future reuse.
  /// </summary>
  /// <param name="pattern">Regular expression pattern</param>
  /// <param name="input">Input string</param>
  /// <returns>True if input matches pattern, else false</returns>
  public bool Match( string pattern, string input )
  {
    if( !ContainsKey( pattern ) )
    {
      Add( pattern, new Regex( pattern ) );
    }
    return this[pattern].IsMatch( input );
  }
}
```

Then the predicate method to determine whether an element has a given built-in category:
```csharp
static bool is\_element\_of\_category(
  Element e,
  BuiltInCategory c )
{
  //return e.Category.Id.Equals(
  //  e.Document.Settings.Categories.get\_Item(
  //    c ).Id );

  return e.Category.Id.IntegerValue.Equals(
    (int) c );
}
```

Here is the main method we have been discussing, to determine a ductwork element's shape from its MEP PartType and Size parameter:
```csharp
static string GetElementShape( Element e )
{
  if( is\_element\_of\_category( e,
    BuiltInCategory.OST\_DuctCurves ) )
  {
    // simple case, no need to use regular expression

    string size = e.get\_Parameter( "Size" )
      .AsString();

    if( size.Split( 'x' ).Length == 2 )
      return "rectangular";
    else if( size.Split( '/' ).Length == 2 )
      return "oval";
    else
      return "round";
  }
  else if( is\_element\_of\_category( e,
    BuiltInCategory.OST\_DuctFitting ) )
  {
    FamilyInstance fi = e as FamilyInstance;

    if( fi != null && fi.MEPModel is MechanicalFitting )
    {
      string size = e.get\_Parameter( "Size" )
        .AsString();

      PartType partType = ( fi.MEPModel as
        MechanicalFitting ).PartType;

      if( PartType.Elbow == partType
        || PartType.Transition == partType )
      {
        // more complex case

        if( size.Split( 'x' ).Length == 3 ) // or use Regex("[0-9]x[0-9]+-[0-9]+/[0-9]+") but splitting is less costly
          return "rectangular2rectangular";
        else if( size.Split( '/' ).Length == 3 ) // but if in imperial units size is in fractional inches format it has to be replaced by another regular expression
          return "oval2oval";
        else if( \_regexCache.Match(
          "[0-9]+\"?x[0-9]+\"?-[0-9]+\"?/[0-9]+\"?", size ) )
            return "rectangular2oval";
        else if( \_regexCache.Match(
          "[0-9]+\"?/[0-9]+\"?-[0-9]+\"?x[0-9]+\"?", size ) )
            return "oval2rectangular";
        else if( \_regexCache.Match(
          "[0-9]+\"?[^0-9]-[0-9]+\"?x[0-9]+\"?", size ) )
            return "round2rectangular";
        else if( \_regexCache.Match(
          "[0-9]+\"?x[0-9]+\"?-[0-9]+\"?[^0-9]", size ) )
            return "rectangular2round";
        else if( \_regexCache.Match(
          "[0-9]+\"?[^0-9]-[0-9]+\"?/[0-9]+\"?", size ) )
            return "round2oval";
        else if( \_regexCache.Match(
          "[0-9]+\"?/[0-9]+\"?-[0-9]+\"?[^0-9]", size ) )
            return "oval2round";
        else if( \_regexCache.Match(
          "[0-9]+\"?[^0-9]-[0-9]+\"?[^0-9]", size ) )
            return "round2round";
        else { return "other case"; }
      }
      // etc (for other part types)
      else
      {
      }
    }
    // etc (for other categories)
    else
    {
    }
  }
  return "unknown";
}
```

Finally, here is the external command class implementation and its Execute method, which ties it all together, prompts the user to select a test element, and displays the result:
```python
[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
class CmdMepElementShape : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    Element e = null;

    try
    {
      e = Util.SelectSingleElementOfType(
       uidoc, typeof( Element ), "an element", true );
    }
    catch( OperationCanceledException )
    {
      message = "No element selected";
      return Result.Failed;
    }

    Util.InfoMsg( string.Format(
      "{0} is {1}",
      Util.ElementDescription( e ),
      GetElementShape( e ) ) );

    return Result.Succeeded;
  }
}
```

Here is
[version 2011.0.87.0](zip/bc_11_87.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.

Many thanks to Max for his research, in-depth explanation and useful sample solution.
By the way, if you have suggestions for other ways to solve this, they will be more than welcome.
Please let us know.
Thank you!

\*\*\*

The advantage of this is that it works in any flavour of Revit, even without RME, since it does not use the connector manager, which is only available in RME.