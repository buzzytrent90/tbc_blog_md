---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.5
content_type: code_example
optimization_date: '2025-12-11T11:44:14.871520'
original_url: https://thebuildingcoder.typepad.com/blog/0911_displayunittype.html
post_number: 0911
reading_time_minutes: 7
series: elements
slug: displayunittype
source_file: 0911_displayunittype.htm
tags:
- csharp
- elements
- family
- parameters
- python
- references
- revit-api
- rooms
- selection
- views
- windows
title: Parameter DisplayUnitType Enhancement, Bretagne and Decompilers
word_count: 1335
---

### Parameter DisplayUnitType Enhancement, Bretagne and Decompilers

After looking at several performance enhancement possibilities last week, e.g. the
[sweep family performance enhancement](http://thebuildingcoder.typepad.com/blog/2013/03/sweep-family-performance-enhancement.html) and
[LoadFamily and collector iteration performance](http://thebuildingcoder.typepad.com/blog/2013/03/loadfamily-and-collector-iteration-performance.html),
here is another exciting and pretty sophisticated exploration of internal Revit API implementation details by
[Victor Chekalin](http://www.facebook.com/profile.php?id=100003616852588) that
can be used to achieve a significant speed improvement when accessing large numbers of parameters and checking their DisplayUnitType property.

#### Brittany and Konk Leon

![Double rainbow in Le Conquet](file:///j/photo/jeremy/2013/2013-03-18_brest/jeremy_brest_rainbow.jpg)
–
![View over the Atlantic from the converted chapel meeting room window](file:///j/photo/jeremy/2013/2013-03-18_brest/brest_view_1_cropped.jpg)

Before getting to that, though, here is an update on my current whereabouts.

Yesterday I spent travelling to Brest and Le Conquet
([Konk Leon](http://br.wikipedia.org/wiki/Konk-Leon) in Breton)
for a meeting with my European DevTech colleagues.

This is the western-most point of northern France, home of my boss Cyrille, and my first visit to the Bretagne.

The weather is exciting and challenging, and the views spectacular.

A quick rainfall in the east and simultaneous glorious evening sunlight from the west produced a beautiful double rainbow.

We have a meeting room in a converted chapel with a view straight out onto the Atlantic.

#### Parameter DisplayUnitType Enhancement and Research

Back to the DisplayUnitType enhancement, here is the description of the research leading up to it in Victor's own words:

Now I'm working with the element parameters again and I need to get all parameters of a lot of elements in the project including their DisplayUnitType.

The problem is that not all parameters have a display unit type.
If I try to retrieve a display unit type for a parameter where it does not exist, an Autodesk.Revit.Exceptions.InvalidOperationException is thrown.
Unfortunately, there is no direct way to determine beforehand whether a parameter has a DisplayUnitType defined or not, i.e. whether the call will succeed or fail.

The most obvious way would be to wrap the code where I retrieve the display unit type in a try... catch block.
But throwing and catching an exception is a very expensive operation and greatly reduces the performance –
[exceptions should be exceptional](http://www.jacopretorius.net/2009/10/exceptions-should-be-exceptional.html).

I decided to go deeper and looked at the RevitAPI.dll using
[dotPeek – free .NET decompiler](http://www.jetbrains.com/decompiler).

At first I went to the DisplayUnitType property of the Parameter class.
```csharp
public DisplayUnitType DisplayUnitType
{
  get
  {
    Definition definition = this.Definition;
    ParamTypeSpec paramTypeSpec1;
    if ((ParamTypeEnum) \*(int\*)
      definition.getParamTypeSpec(¶mTypeSpec1)
      != (ParamTypeEnum) 15)
    {
      throw new Autodesk.Revit.Exceptions
        .InvalidOperationException(new FunctionId(
          "n:\\build\\2013\_ship\_x64\_inst\_20120221\_2030\\source\\api\\revitapi\\objects\\parameters\\APIParameter.cpp",
          581, "Autodesk::Revit::DB::Parameter::DisplayUnitType::get"),
          string.Empty);
    }
    else
    {
      ParamTypeSpec paramTypeSpec2;
      return (DisplayUnitType)
        \u003CModule\u003E.FormatOptions\u002EgetDisplayUnits(
          \u003CModule\u003E.AUnits\u002EgetFormatOptions(
            \u003CModule\u003E.ADocument\u002EgetAUnits(
              (ADocument\*) \*(long\*) this.m\_pCDA),
                (UnitType.Enum) \*(int\*) ((IntPtr)
                definition.getParamTypeSpec(¶mTypeSpec2) + 8L)));
    }
  }
}
```

The most interesting thing here is 'if ((ParamTypeEnum) \*(int\*) definition.getParamTypeSpec(¶mTypeSpec1) != (ParamTypeEnum) 15)'.
The first idea is to call this method via reflection.
But getParamTypeSpec is an internal unsafe method.
Unfortunately, I don't know how to use reflector with pointers and internal structures.
So, I decided to go another way.

The next step is to look at the ParameterType property of the Definition class.
```csharp
public override ParameterType ParameterType
{
  get
  {
    REVIT\_MAINTAIN\_STATE revitMaintainState;
    \u003CModule\u003E.REVIT\_MAINTAIN\_STATE\u002E\u007Bctor\u007D(
      &revitMaintainState);

    ParameterType parameterType;
    try
    {
      ParamDef\* paramDefPtr = this.m\_pParamDef;
      ParamTypeSpec paramTypeSpec;
      \u003CModule\u003E.paramDefToParamType(
        ¶mTypeSpec, paramDefPtr);
      switch (^(int&) @paramTypeSpec)
      {
      case 1:
        parameterType = ParameterType.Text;
        break;
      case 6:
        parameterType = ParameterType.URL;
        break;
      case 9:
        parameterType = ParameterType.YesNo;
        break;
      case 11:
        parameterType = ParameterType.Integer;
        break;
      case 13:
        parameterType = ParameterType.Material;
        break;
      case 14:
        parameterType = ParameterType.FamilyType;
        break;
      case 15:
        switch (^(int&) ((IntPtr) ¶mTypeSpec + 8))
        {
        case 0:
          parameterType = ParameterType.Length;
          break;
        case 1:
          parameterType = ParameterType.Area;
          break;
        case 2:
          parameterType = ParameterType.Volume;
          break;
        case 3:
          parameterType = ParameterType.Angle;
          break;
        case 4:
          parameterType = ParameterType.Number;
          break;
        case 26:
          parameterType = ParameterType.Force;
          break;
        case 27:
          parameterType = ParameterType.LinearForce;
          break;
        case 28:
          parameterType = ParameterType.AreaForce;
          break;
        case 29:
          parameterType = ParameterType.Moment;
          break;
        default:
          parameterType = (ParameterType) (
            ^(int&) ((IntPtr) ¶mTypeSpec + 8) + 100);
          break;
        }
      case 16:
        parameterType = ParameterType.NumberOfPoles;
        break;
      case 24:
        parameterType = ParameterType.FixtureUnit;
        break;
      case 30:
        parameterType = ParameterType.LoadClassification;
        break;
      default:
        parameterType = ParameterType.Invalid;
        break;
      }
    }
    \_\_fault
    {
      \u003CModule\u003E.\_\_\_CxxCallUnwindDtor(
        (\_\_FnPtr<void (void\*)>) \_\_methodptr(REVIT\_MAINTAIN\_STATE\u002E\u007Bdtor\u007D),
          (void\*) &revitMaintainState);
    }
    \u003CModule\u003E.REVIT\_MAINTAIN\_STATE\u002E\u007Bdtor\u007D(
      &revitMaintainState);
    return parameterType;
    }
  }
}
```

Here we can see that a parameter will have a DisplayUnitType only if the ParameterType is Length, Area, etc..
So we can just check if the ParameterType is one of the Length, Area, etc. ones.
If so, its parameter has a DisplayUnitType, otherwise not.

It is necessary to pay attention to the line
```csharp
  default:
    parameterType = (ParameterType) (
      ^(int&) ((IntPtr) ¶mTypeSpec + 8) + 100);
    break;
```

Looking at the ParameterType enumeration suggests that every parameter type whose integer value is greater than 100 belongs to paramTypeSpec = 15.

As a result, I created the following extension method for the Parameter class:
```csharp
public static class ParameterExtensions
{
  public static bool HasDisplayUnitType(
    this Parameter parameter )
  {
    var parameterType =
        parameter.Definition.ParameterType;

    switch( parameterType )
    {
      case ParameterType.Length:
      case ParameterType.Area:
      case ParameterType.Volume:
      case ParameterType.Angle:
      case ParameterType.Number:
      case ParameterType.Force:
      case ParameterType.LinearForce:
      case ParameterType.AreaForce:
      case ParameterType.Moment:
        return true;
    }

    /\* At the reflector I can see the following code
          default:
            parameterType = (ParameterType)
              (^(int&) ((IntPtr) ¶mTypeSpec
              + 8) + 100);
            break;
        looking at the ParameterType enumeration
        suggests that every parameter type whose
        integer value is greater than 100 belongs
        to paramTypeSpec = 15
    \*/
    return 100 < (int) parameterType;
  }
}
```

#### Comparison Benchmarking

To wrap this up, here is a little benchmark comparing the two methods.

The first one exercises the try... catch block:

```python
  Reference r;

  try
  {
    r = uidoc.Selection.PickObject(
      ObjectType.Element );
  }
  catch( OperationCanceledException )
  {
    message = "Cancelled";
    return Result.Cancelled;
  }

  var e = doc.GetElement( r.ElementId );

  Stopwatch sw = Stopwatch.StartNew();

  foreach( Parameter parameter in e.Parameters )
  {
    try
    {
      DisplayUnitType dut =
          parameter.DisplayUnitType;

      Debug.Print( dut.ToString() );
    }
    catch( Autodesk.Revit.Exceptions
      .InvalidOperationException )
    {
    }
  }
  sw.Stop();

  TaskDialog.Show( "Benchmark result",
    sw.Elapsed.ToString() );
```

Pick an element.
The result is 0.41 seconds.

Second measurement, with my new method:

```python
  Reference r;

  try
  {
    r = uidoc.Selection.PickObject(
      ObjectType.Element );
  }
  catch( OperationCanceledException )
  {

    message = "Canceled";
    return Result.Cancelled;
  }

  var e = doc.GetElement( r.ElementId );

  Stopwatch sw = Stopwatch.StartNew();

  foreach( Parameter parameter in e.Parameters )
  {
    if( parameter.HasDisplayUnitType() )
    {
      DisplayUnitType dut =
          parameter.DisplayUnitType;

      Debug.Print( dut.ToString() );
    }
  }
  sw.Stop();

  TaskDialog.Show( "Benchmark result",
    sw.Elapsed.ToString() );
```

The result is 0.029 seconds for the same element!

You may say that 0.41 seconds is not so much, and users will not notice the difference between 0.41 seconds and 0.028.
Of course the difference is not huge if you work with only one element.
But note that the exception handling code works 15 times slower, repeatedly, ever single time it is called!

If you retrieve parameters for 100 elements, it takes 41 seconds using try... catch instead of just 3 seconds with my new method.

I hope this enhancement will be useful for other developers as well.

Many thanks to Victor for his very professional analysis and valuable result!

The best of luck to you in your new job, Victor, and congratulations on that achievement!

#### IL Decompilation

I talked about
[Reflector](http://thebuildingcoder.typepad.com/blog/2008/10/converting-between-vb-and-c-and-net-decompilation.html) way
back in the early days of the blog.
It since became commercial.

Victor mentions using
[dotPeek](http://www.jetbrains.com/decompiler) for
his explorations above.

When I brought this up with my colleagues, Adam added that he uses the
[ILSpy .NET Decompiler](http://ilspy.net) and
is perfectly happy with that as well.