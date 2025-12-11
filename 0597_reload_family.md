---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.5
content_type: code_example
optimization_date: '2025-12-11T11:44:14.234324'
original_url: https://thebuildingcoder.typepad.com/blog/0597_reload_family.html
post_number: 0597
reading_time_minutes: 4
series: structural
slug: reload_family
source_file: 0597_reload_family.htm
tags:
- csharp
- elements
- family
- parameters
- python
- revit-api
- transactions
- structural
title: Reloading a Family
word_count: 825
---

### Reloading a Family

Here is an impressively complete list of pretty fundamental beginner's issues that you might potentially run into when reloading a family, starting from scratch, from a recent case handled by my colleague Joe Ye.
One of the interesting points that Joe ends up making is that in order to reload a family that has already been loaded into the document, you can use the LoadFamily overload taking an
[IFamilyLoadOptions](http://thebuildingcoder.typepad.com/blog/2010/02/ifamilyloadoptions-and-gemini.html) argument.
Before and after getting to that stage, here are some other issues that you can potentially run into and need to resolve:

1. [Replacing the family type or symbol of a family instance with another type](#1).- [Setting up an appropriate transaction](#2).- [Reloading an already loaded family](#3).- [Creating a new family type](#4).

#### 1. Changing the Symbol of a Family Instance

**Question:** How can I replace the family type or symbol of a family instance with some other type via the API?

**Answer:** The type of a family instance can be changed by modifying its Symbol property.
You can simply assign a new FamilySymbol instance to FamilyInstance.Symbol property.
Here is some pseudo code to illustrate this:
```csharp
// the code to retrieve the new family symbol:
FamilySymbol newSymbol = ... ;
// the code to access the family instance:
FamilyInstance famInstance = ... ;
// change the family instance type:
famInstance.Symbol = symbol1;
```

By the way, the Revit SDK provides the Revit Developer Guide PDF file, which can guide you to the knowledge of programming on Revit and also provides some real sample code to achieve exactly what you need.

#### 2. Setting up an Appropriate Transaction

**Question:** I followed your advice, but now I am facing an exception saying "A sub-transaction can only be active inside an open Transaction" in the following line:
```csharp
// add a new type and edit its parameters
FamilyType newFamilyType
= familyManager.NewType("2X2");
```

I am trying to add a new type to the family in order to assign that as a replacement symbol to the family instance.
What could be going wrong here?

**Answer:** The reason might be that you have set up a manual transaction mode for your command.
You have to specify either manual or automatic transaction mode for you command.
You probably set up manual mode but omitted to explicitly start a transaction.
You can either change the mode to Automatic, or leave it as Manual and add the code to start and commit the transaction.

Here is an example of automatic transaction mode:
```python
[TransactionAttribute( TransactionMode.Automatic )]
public class RevitCommand : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    EditFamilyTypes( doc, famInstance );

    return Result.Succeeded;
  }
}
```

Here is an example of retaining the manual transaction model: start your own transaction first, and then commit it after executing the code to change the model:
```python
[TransactionAttribute( TransactionMode.Manual )]
public class RevitCommand : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {

    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    Transaction trans = new Transaction( doc, "ExComm" );
    trans.Start();

    EditFamilyTypes( doc, famInstance );

    trans.Commit();
    return Result.Succeeded;
  }
}
```

#### 3. Reloading an Already Loaded Family

**Question:** The transaction handling improved matters.
Now the execution stops at the line
```csharp
family = familyDoc.LoadFamily(doc);
```
It returns an error saying "Family loading failed".

**Answer:** The error occurs because the family is already present in the current model.
If you use the Document.LoadFamily method overload taking a Document argument only to reload an existing family document, it will always fail.
Instead, you should use the LoadFamily overload that accepts an argument implementing the IFamilyLoadOptions interface.
Here is a simple implementation of such a class:
```python
class FamilyOption : IFamilyLoadOptions
{
  public bool OnFamilyFound(
    bool familyInUse,
    ref bool overwriteParameterValues )
  {
    overwriteParameterValues = true;
    return true;
  }

  public bool OnSharedFamilyFound(
    Family sharedFamily,
    bool familyInUse,
    ref FamilySource source,
    ref bool overwriteParameterValues )
  {
    return true;
  }
}
```

With the IFamilyLoadOptions implementation in place, you can call the appropriate LoadFamily overload:
```csharp
family = familyDoc.LoadFamily(
doc, new FamilyOption() );
```

#### 4. Creating a New Family Type

**Question:** The updated code works fine when I only select one object.
If I select more than one object, it gives me error in the following line:
```csharp
newFamilyType = familyManager.NewType("xx");
```
The error says "The type name xx is already in use".
I need to able to select and modify the type of more than one object, and assign all of them the new type "xx".
How can I achieve that, please?

**Answer:** If you are trying to change the type of more than one family instance, the new type does not need to be created repeatedly.
For the second family instance, you can bypass the code creating a new type using an 'if' conditional statement.

For completeness sake, here is [ReloadFamily.zip](zip/ReloadFamily.zip) containing the complete Revit 2011 source code and Visual Studio solution of these various code snippets.

Many thanks to Joe for this exhaustive explanation.