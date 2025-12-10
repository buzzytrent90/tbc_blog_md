---
post_number: "0221"
title: "Adding a Category to a Parameter Binding"
slug: "add_category_to_binding"
author: "Jeremy Tammik"
tags: ['elements', 'parameters', 'revit-api']
source_file: "0221_add_category_to_binding.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0221_add_category_to_binding.html"
---

### Adding a Category to a Parameter Binding

Here is another interesting question raised and answered by Henrik Bengtsson of
[Lindab](http://www.lindab.se).
It is another example of how you sometimes need to dig in rather deeply into Revit to update certain internal information stored in the Revit application or document object, a little bit similar to but also a bit more complex than the discussion on how to
[modify the application options library paths](http://thebuildingcoder.typepad.com/blog/2009/08/library-paths.html).

Before getting into Henrik's issue, here is some background information on the classes involved in defining the parameter binding.
The Binding class is a base class used to manage this kind of relationship.
This is what the Revit API documentation has to say about this class:

There are currently two types of bindings available, instance and type binding.
The key difference between the two is that the instance bound parameters appear on all instances of the elements in those categories.
Changing the parameter on one does not affect the other instances of the parameter.
The Type bound parameters appear only on the type object and is shared by all the instances that use that type.
Changing the type bound parameter affects all instances of the elements that use that type.
Note that a definition can only be bound to an instance or a type and not both.

The API defines a derived class named ElementBinding as a base class for all types of binding that attach to an element, and two further classes InstanceBinding and TypeBinding derived from that.

The ElementBinding class has a property named Categories returning a CategorySet class instance which allows us to retrieve or modify the categories to which a parameter definition will be bound.
Unfortunately, simply adding a new category to this collection does not have the desired effect of updating the categories bound to a parameter in the Revit document.
Here is Henrik's original question:

**Question:** I have a set of existing parameter bindings which each bind a parameter definition to one or more categories.
I now wish to implement a method to add a new category to an existing binding.
Something like this, which retrieves an external parameter definition from my shared parameters file and then checks whether it is connected to all the different required categories.
If not, I loop through my desired categories and add the missing ones:

```
Dim extDef an ExternalBinding

Dim b As InstanceBinding _
  = doc.ParameterBindings.Item(extDef)

For Each c As Category In categorySet
  If Not b.Categories.Contains(c) Then
    b.Categories.Insert(c)
  End If
Next
```

From the external definition, I obtain the instance binding object.
I then try to add new categories to that.
It works fine programmatically, but has no effect on the Project Parameters displayed in the Revit user interface.

After a rather convoluted communication process, a couple of misunderstandings, a period of rest for Henrik to enjoy a well-earned holiday in Spain, various demonstration projects and lots of research, here is Henrik's final solution:

**Answer:** Here is part of the routine which checks and adds required parameters, including four statements marked 'HB'.
If a given external definition for a specific shared parameter has not yet been added to the document for a given element binding, it does so.
Otherwise, it retrieves the existing element binding and checks whether all required categories are present in it.
If not, they are added.
If the 'HB' lines are commented out, this is not reflected in the project.
Uncommenting the lines marked 'HB' causes the ReInsert method to be called after modifying the binding's category collection.
This is apparently the only way to force the update to have an effect on the project document:
```vbnet
  For Each extDef As ExternalDefinition In Binding.ParametersCol
    If doc.ParameterBindings.Contains(extDef) Then
      Dim Added As Boolean = False
      Dim b As ElementBinding = doc.ParameterBindings.Item(extDef)
      For Each c As Category In categorySet
        If Not b.Categories.Contains(c) Then
          b.Categories.Insert(c)
          'Added = True  ' HB
        End If
      Next
      'If Added = True Then  ' HB
      ' doc.ParameterBindings.ReInsert(extDef, b)  ' HB
      'End If  ' HB
    Else
      doc.ParameterBindings.Insert(extDef, elementBinding)
    End If
  Next
```
Here are the steps to reproduce the issue:

1. Run the external command with the lines marked HB commented out.
   Look at the Project Parameters section afterwards.
   The parameter is only is connected to ProjectInformation, like it was originally.
   No changes have been applied.- Remove the remark character in the lines marked HB and run the external command.
     Now there is a connection between the parameter and both ProjectInformation and Column, i.e. the changes have been successfully applied to the Revit project document.

Apparently, you have to make a call to the ReInsert method for the changes to have any effect.
Nothing else seems to be good enough.

Many thanks to Henrik for all his research and this clean solution!