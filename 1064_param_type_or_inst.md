---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: code_example
optimization_date: '2025-12-11T11:44:15.227405'
original_url: https://thebuildingcoder.typepad.com/blog/1064_param_type_or_inst.html
post_number: '1064'
reading_time_minutes: 4
series: elements
slug: param_type_or_inst
source_file: 1064_param_type_or_inst.htm
tags:
- elements
- family
- parameters
- python
- references
- revit-api
- transactions
- walls
title: Determining Whether Parameter is Type or Instance Bound
word_count: 871
---

### Determining Whether Parameter is Type or Instance Bound

Here is a question that came up now and then in the past, on how to determine whether a given parameter retrieved from the document shared parameter bindings is bound to a type or instances.

**Question:** I have not been able to find a way in which to determine if a parameter is a type or instance in the Revit API.
The only reference I could find was to the FamilyParameter.IsInstance property.
However, this only works if you are in the family editor.
I need to determine whether parameters in the project file are set to type or instance.
Is there a way?

**Answer:** Normally, you could just check the source element class.

The Parameter.Element property tells you the element to which a given parameter belongs.

If it is derived from the ElementType base class, you have a type parameter, else an instance one.

However, if you are iterating over the parameter definitions in the document ParameterBindings container, you have no element available to check.

The parameter itself will not contain this information, since it is generic and does not need to know how it will be used.

The ParameterBindings contains a map of definitions and bindings, so let's take a closer look at the latter.

Here are some previous discussions on the InstanceBinding and TypeBinding binding subclasses, and on checking instance versus type and shared versus not for family parameters:

- Creating a [Model Group Shared Parameter](http://thebuildingcoder.typepad.com/blog/2009/06/model-group-shared-parameter.html)
- [Adding a Category to a Parameter Binding](http://thebuildingcoder.typepad.com/blog/2009/09/adding-a-category-to-a-parameter-binding.html)
- [Shared Type Parameter](http://thebuildingcoder.typepad.com/blog/2010/07/shared-type-parameter.html)
- [Adding a Category to a Shared Parameter Binding](http://thebuildingcoder.typepad.com/blog/2012/04/adding-a-category-to-a-shared-parameter-binding.html)
- [Categories Collection Item Accessor Fails](http://thebuildingcoder.typepad.com/blog/2012/09/categories-collection-item-accessor-fails.html)
- [FamilyParameter IsShared Property](http://thebuildingcoder.typepad.com/blog/2012/09/familyparameter-isshared-property.html)
- [Changing a Family Parameter from Type to Instance](http://thebuildingcoder.typepad.com/blog/2013/04/changing-a-family-parameter-from-type-to-instance.html)

Let's go back to basics for a moment and look at the Revit API help file RevitAPI.chm documentation of the classes involved:

The **ElementBinding** class is a base class for all types of binding that attach to an element.

The **TypeBinding** class is derived from ElementBinding and defines objects used to bind a property to a Revit type, such as a wall type.

This differs from instance bindings in that the property is then shared by all instances that use that type. Changing the parameter for one type affects all other instances that use that type.

Sample code is included that shows how to create a TypeBinding instance to add a shared parameter to a wall type.

The **InstanceBinding** class, derived from ElementBinding, defines objects used to bind a parameter definition and a parameter to each instance of an element, such as a wall.

Once bound, the parameter appears on all instances of the element and changing the parameter on any one single instance will not change the value on any other instance.

Sample code is included that shows how to create an InstanceBinding instance to add a shared parameter to all walls.

So in your case, we should be able to identify the binding type by looking at the type of the binding instance.

I reimplemented an obsolete test command in The Building Coder samples that never did anything useful in the past, CmdListSharedParams, to classify instance versus type shared parameter definitions.

It is very simple.
All you need to do it retrieve the binding and check whether it is an instance or type one.
The entire external read-only command implementation looks like this:

```python
[Transaction( TransactionMode.ReadOnly )]
class CmdListSharedParams : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    UIDocument uidoc = app.ActiveUIDocument;
    Document doc = uidoc.Document;

    BindingMap bindings = doc.ParameterBindings;

    int n = bindings.Size;

    Debug.Print( "{0} shared parementer{1} defined{2}",
      n, Util.PluralSuffix( n ), Util.DotOrColon( n ) );

    if( 0 < n )
    {
      DefinitionBindingMapIterator it
        = bindings.ForwardIterator();

      while( it.MoveNext() )
      {
        Definition d = it.Key as Definition;
        Binding b = it.Current as Binding;

        Debug.Assert( b is ElementBinding,
          "all Binding instances are ElementBinding instances" );

        Debug.Assert( b is InstanceBinding
          || b is TypeBinding,
          "all bindings are either instance or type" );

        // All definitions obtained in this manner
        // are InternalDefinition instances, even
        // if they are actually associated with
        // shared parameters, i.e. external.

        Debug.Assert( d is InternalDefinition,
          "all definitions obtained from BindingMap are internal" );

        string sbinding = ( b is InstanceBinding )
          ? "instance"
          : "type";

        Debug.Print( "{0}: {1}", d.Name, sbinding );
      }
    }
    return Result.Succeeded;
  }
}
```

The updated version of
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) CmdListSharedParams
command described above is published as
[release 2014.0.105.3](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.105.3).

Many thanks to Miro for clarifying this again and his trusty old
[HelperParams.cs](http://thebuildingcoder.typepad.com/blog/2012/04/adding-a-category-to-a-shared-parameter-binding.html) module.