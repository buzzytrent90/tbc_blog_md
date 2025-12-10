---
post_number: "0323"
title: "Duplicate Mark Values"
slug: "duplicate_mark_values"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'filtering', 'parameters', 'revit-api', 'schedules', 'transactions', 'views', 'windows']
source_file: "0323_duplicate_mark_values.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0323_duplicate_mark_values.html"
---

### Duplicate Mark Values

We already discussed the issue of
[door marks](http://thebuildingcoder.typepad.com/blog/2009/09/door-marks.html) and
implemented the Building Coder sample command CmdListMarks to demonstrate listing and modifying them programmatically.
Here are a couple of additional questions that came up recently in this context from Dave Echols of
[Hankins and Anderson, Inc.](http://www.haengineers.com) which
led to some important hints on performance and exception handling:

**Question:** We have some large Revit models that have thousands of errors due to duplicate mark values.
On several of these, we have cleaned out the errors manually in order to speed up the loading of Revit.
I have written the following external command to automate this process:
```csharp
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  Element e;
  int num = 1;
  ElementIterator it = doc.Elements;
  while( it.MoveNext() )
  {
    e = it.Current as Element;
    try
    {
      // get the BuiltInParameter.ALL\_MODEL\_MARK paremeter.
      // If the element does not have this paremeter,
      // get\_Parameter method returns null:

      Parameter p = e.get\_Parameter(
        BuiltInParameter.ALL\_MODEL\_MARK );

      if( p != null )
      {
        // we found an element with the
        // BuiltInParameter.ALL\_MODEL\_MARK
        // parameter. Change the value and
        // increment our value:

        p.Set( num.ToString() );
        ++num;
      }
    }
    catch( Exception ex )
    {
      Util.ErrorMsg( "Exception: " + ex.Message );
    }
  }
  doc.EndTransaction();
  return CmdResult.Succeeded;
}
```

I have several questions concerning this code and any consequences that might arise from the way I am renumbering all elements with mark parameters:

1. In my code, I am iterating through all elements in the model in order to find elements that have the BuiltInParameter.ALL\_MODEL\_MARK parameter. I have tried using the Filter.NewParameterFilter method to find all of these occurrences, but have not been successful. Can you provide a sample that uses filters to find these elements more efficiently?- In my code, I am incrementing the value by one to provide a unique value for each element that has the BuiltInParameter.ALL\_MODEL\_MARK parameter. Are there any unintended consequences I need to be aware of if I implement this code in our production models?- Some other examples I have seen that fix the Duplicate Mark Values errors provide a unique indexed value for each element based on the category of the element.
       For example, I have 5 doors and 15 windows.
       My door index values would range from 1 to 5 and my window index values would range from 1 to 15.
       This approach took over 4 hours on one of our models.
       Performance is not a huge problem, because the solution is working, even though it is a little slow.
       We do not use this command every day, so the importance to optimize it is not very high.
       Still, do I need to implement my command following this approach?- After digging into the Revit database using RvtMgdDbg and the Visual Studio debugger, I noticed the BuiltInParameter.ALL\_MODEL\_MARK parameter is tied to or is the BuiltInParameter.DOOR\_NUMBER parameter (even for a DimensionType).
         You can see that by following these steps in RvtMgdDbg:

- Snoop DB from the add-in menu.- Select an element.- Double-click on the Parameters property to open the Snoop Parameters dialogue.- Select the Built-in Enum Snoop button.- Select the ALL\_MODEL\_MARK built-in parameter.- Double-click the Definition property to view the Internal Definition.- The Built-in Param property has the value DOOR\_NUMBER:

![ALL_MODEL_MARK definition](img/duplicate_mark_value.png)

Are these two parameters the same object in the database?
If I change these mark values, will values change in schedules I generate from elements that have been changed?

Thanks for shedding some light on this issue.

**Answer:** Thank you for your interesting query.
As said, we already had a look at
[listing and modifying door marks](http://thebuildingcoder.typepad.com/blog/2009/09/door-marks.html).
Now to address your specific questions:

1. The following method should do what you need:

```csharp
/// <summary>
/// Retrieve all elements in the current active document
/// having a non-empty value for the given parameter.
/// </summary>
static int GetElementsWithParameter(
  List<Element> elements,
  BuiltInParameter bip,
  Application app )
{
  Document doc = app.ActiveDocument;

  Autodesk.Revit.Creation.Application a
    = app.Create;

  Filter f = a.Filter.NewParameterFilter(
    bip, CriteriaFilterType.NotEqual, "" );

  return doc.get\_Elements( f, elements );
}
```

2. Not that I am aware of. As you can see in my door mark sample, I set the mark parameter to a completely arbitrary value and know of no ill consequences.- I am pretty sure that you do not need to implement this approach.
     It is probably done for individual reasons, users preferring to have their different element types numbered individually. There is nothing stopping you from doing it this way, but nothing forcing you to either, as far as I know.

     I am also pretty sure that performance can be improved.
     Look at the
     [Revit 2010 subscription pack API enhancements](http://thebuildingcoder.typepad.com/blog/2009/10/revit-2010-subscription-pack.html#4).
     In previous versions, Revit regenerated the model after each and every modification, such as your update to the model mark parameter. From the subscription pack onwards, this regeneration can be avoided.

     I see another serious performance issue in your code as well, by the way.
     To check whether the model mark has been set, you are starting up an exception handling block for each and every element in the Revit database, of which there are many!
     On very many of the elements, the mark value will not be present and an exception will be thrown and handled.
     This is a very costly operation.
     In general, you should never use exceptions to handle expected situations, such as the absence of this parameter.
     [Exceptions](http://en.wikipedia.org/wiki/Exception_handling) should always be
     [exceptional, for handling unexpected errors](http://www.jacopretorius.net/2009/10/exceptions-should-be-exceptional.html).
     Performance will probably be very significantly improved if you simply check whether e.get\_Parameter returns null, in which case that element can be skipped, instead of causing an exception to be thrown.
     Actually, since you are already checking whether p is null, you should be able to simply remove the try-catch statements completely, or alternatively move them into the if block so that they only protect the actual setting of the parameter value.- I suggest you try this out yourself and see. Trusting my word for it would be fine, but controlling it for yourself is always better.