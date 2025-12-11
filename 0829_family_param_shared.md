---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.9
content_type: qa
optimization_date: '2025-12-11T11:44:14.687050'
original_url: https://thebuildingcoder.typepad.com/blog/0829_family_param_shared.html
post_number: 0829
reading_time_minutes: 3
series: family
slug: family_param_shared
source_file: 0829_family_param_shared.htm
tags:
- csharp
- family
- parameters
- revit-api
title: FamilyParameter IsShared Property
word_count: 628
---

﻿

### FamilyParameter IsShared Property

A long time ago, we discussed a workaround to access the
[shared family parameter GUID](http://thebuildingcoder.typepad.com/blog/2011/01/access-to-shared-family-parameter-guid.html)
using
[reflection](http://en.wikipedia.org/wiki/Reflection_%28computer_programming%29),
more specifically (and obviously)
[.NET Reflection](http://msdn.microsoft.com/en-us/library/f7ykdhsy.aspx).
If you are new to this, here is an in-depth Code Project article on
[Reflection in .NET](http://www.codeproject.com/Articles/55710/Reflection-in-NET).

Now Victor Chekalin, or Виктор Чекалин, ran into this issue as well and presents a much improved solution.

Here is the background and our discussion on this:

**Question:** Why does the FamilyParameter class have no IsShared Property, like the Parameter class does?
I need to get list of all FamilyParameters and all their properties, including GUID.

Now, if I call the GUID property on a FamilyParameter that is not shared, Revit throws an InvalidOperationException.
The only way I can check whether the FamilyParameter is shared or not is try to get GUID property and handle the exception if it is not shared.

This is a problem, because intercepting the exception takes a lot of time, and if a family has a lot of FamilyParameters, the program freezes significantly.

Why did the API developers implement this odd behaviour and not add an IsShared property to the FamilyParameter?
Alternatively, they could just return null if the FamilyParameter is not shared.

**Answer:** Sorry about that.
These properties can be accessed
[using .NET Reflection](http://thebuildingcoder.typepad.com/blog/2011/01/access-to-shared-family-parameter-guid.html).

**Response:** Yes, that looks as if it would help me.

I looked at RevitAPI.dll using Reflector and find it harder still to understand why the API developers didn't add the IsShared property to the FamilyParameter class.
It is just a wrapper for the Parameter class, and it would just take one or two lines of code, e.g.
```csharp
  public bool IsShared
  {
    get { return getParameter().IsShared; }
  }
```

Re-examining your post, though, maybe I understand after all:
the GUID and IsShared properties were only added to the Parameter class in the Revit 2011 API.
I guess that it was simply an oversight and they forgot add the new properties to the FamilyParameters as well.
Maybe I'm wrong.

I hope this property is added in the future versions of the Revit API.
As I can see from the post comments, I am not the only one wanting to see an IsShared property.

#### Implementing a FamilyParameter IsShared Extension Method

Based on your sample in the post, I wrote a simple extension method:
```csharp
  public static bool IsShared(
    this FamilyParameter familyParameter )
  {
    MethodInfo mi = familyParameter
      .GetType()
      .GetMethod( "getParameter",
        BindingFlags.Instance
        | BindingFlags.NonPublic );

    if( null == mi )
    {
      throw new InvalidOperationException(
        "Could not find getParameter method" );
    }

    var parameter = mi.Invoke( familyParameter,
      new object[] { } ) as Parameter;

    return parameter.IsShared;
  }
```

I used the internal getParameter method instead the m\_Parameter field, because it is not recommended to use fields from other classes.
Also, the getParameter method is used in most of the code that I saw in Reflector, not the m\_Parameter field.

#### Using the FamilyParameter IsShared Extension Method

The use of the new method is completely obvious:
```csharp
  foreach( FamilyParameter fp in mgr.Parameters )
  {
    if( fp.IsShared() )
    {
      familyTypeParameter.Guid = fp.GUID;
    }
  }
```

It works much faster than triggering and catching the exception, e.g.
```csharp
  try { var guid = parameter.GUID; }
  catch() {}
```

Thank you again for reminding me about reflection.

**Answer:** Many thanks to **you**, Victor, for your analysis, clean implementation, and sharing this with us!

Here is an updated
[version 2013.0.99.2](zip/bc_13_99_2.zip) of
The Building Coder samples including the extension method and a test call to it in the CmdFamilyParamGuid external command.