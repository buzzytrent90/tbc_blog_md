---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.552078'
original_url: https://thebuildingcoder.typepad.com/blog/0208_shared_param_visibility.html
post_number: 0208
reading_time_minutes: 2
series: views
slug: shared_param_visibility
source_file: 0208_shared_param_visibility.htm
tags:
- csharp
- parameters
- revit-api
- windows
- views
title: Shared Parameter Visibility
word_count: 373
---

### Shared Parameter Visibility

Here is an interesting question and a workaround for a problem accessing the visibility property on shared parameters, raised by Jon Smith of
[Construction Industry Solutions COINS](http://www.constructionindustrysolutions.com):

**Question:** We need to be able to tell if a parameter is visible or invisible, so we can honour that setting in our own interface and not display it if set to invisible.
We try to do this as follows:

```
ExternalDefinition externalDef
  = param.Definition as ExternalDefinition;

if( externalDef == null || externalDef.Visible )
{
  // show the parameter
}
```

However, all definitions returned by this method for any parameter whatsoever are InternalDefinition, including for shared parameters that have been marked as invisible.
The InternalDefinition does not have the Visible property exposed.

It also seems, from looking in the Visual Studio watch window, that the base Definition class does contain a visible property (whose value is correct), but it is internal and not exposed:
![Parameter visibility](img/VisibleParameter.png)

**Answer:** Yes, there is a known issue accessing the definition class of a shared parameter.
The Parameter.Definition property on a shared parameter erroneously returns an InternalDefinition, even though it actually should be returning an ExternalDefinition.
This issue is being looked into and will be resolved soon.
It may be possible to implement a workaround to get the proper external definition through the sequence Application.OpenSharedParameterFile > DefinitionFile > DefinitionGroup > ExternalDefinition.

Happily, if Visual Studio is able to display the visibility property that you are interested in, then you should also be able to access it from your application source code using .NET
[reflection](http://en.wikipedia.org/wiki/Reflection_%28computer_science%29).

**Response:** Excellent - thanks for the information on the issue with the Parameter.Definition property, and thanks for pointing me towards using reflection.
We have created the following function that solves the problem for the current release:
```csharp
public static bool isParameterVisible( Parameter p )
{
  bool bParameterIsVisible = true;

  try
  {
    BindingFlags flags
      = BindingFlags.Instance
      | BindingFlags.FlattenHierarchy
      | BindingFlags.Public
      | BindingFlags.NonPublic
      | BindingFlags.InvokeMethod;

    Type t = typeof( Definition );
    object result = t.InvokeMember(
      "get\_Visible", flags, null, p.Definition, null );

    if( null != result && result is Boolean )
    {
      bParameterIsVisible = (Boolean) result;
    }
  }
  catch( System.Exception )
  {
    // in case of any problems, assume parameter is visible
  }
  return bParameterIsVisible;
}
```

Many thanks to Jon for this cool little workaround!