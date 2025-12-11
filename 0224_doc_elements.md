---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.580195'
original_url: https://thebuildingcoder.typepad.com/blog/0224_doc_elements.html
post_number: '0224'
reading_time_minutes: 3
series: elements
slug: doc_elements
source_file: 0224_doc_elements.htm
tags:
- csharp
- elements
- filtering
- revit-api
- views
title: Document Elements
word_count: 637
---

### Document Elements

We have already used the Document.Elements property a couple of hundred times in previous posts to this blog, so this explanation is rather belated, but addresses a question that does keep on popping up from time to time anyway:

**Question:** How can I access the Document.Elements property in C#?

The Revit help document lists the property named Document.Elements, stating that it is an overloaded property providing access to a set of elements from the document. The overloads are listed as

- Elements: Provides access to all elements within the document.- Elements( Filter ): Provides access to all elements which satisfy specified filter.- Elements( Type ): Provides access to all elements of specified type.- Elements( Filter, ElementArray ): Collects all elements which satisfy specified filter.- Elements( Filter, ICollection of Element ): Collects all elements which satisfy specified filter.- Elements( Type, ElementArray ): Collects all elements which satisfy specified type.- Elements( Type, ICollection of Element ): Collects all elements which satisfy specified type.

However, when I try to access these properties in C#, the Visual Studio compiler reports an error and says that they do not exist.
How can I access these properties in C#?

**Answer:** You can look at the C# source code generated automatically by the definition of the Document class by Visual Studio, by positioning the cursor over the Document type and pressing F12.
This shows a completely different list of overloads:

- public ElementIterator Elements;- public ElementIterator get\_Elements( Filter );- public ElementIterator get\_Elements( Type );- public int get\_Elements( Filter, ElementArray );- public int get\_Elements( Filter, ICollection );- public int get\_Elements( Type, ElementArray );- public int get\_Elements( Type, ICollection );

As you can see, only the Elements property with no filter or type argument is available using the name given in the help file.
The other Document.Elements members are considered to be methods, not properties, and have therefore been automatically decorated with the accessor prefix "get\_".
To call them, you need to prepend "get\_" and obviously also provide the required arguments.

**Question:** How can I access the Document.Elements property in VB?

I have converted some C# sample the code to VB but have an error with a function in the line
```csharp
m\_doc.getElements(viewFilter, views)
```

The error states that .GetElements is not a member of Autodesk.Revit.Document, although I have defined m\_doc as an Autodesk.Revit.Document.

**Answer:** You can use the Visual Studio Intellisense functionality or the object browser to determine the real name of the method you are trying to invoke in whatever language you prefer to use.

The method or property you are accessing is called Elements in the Revit API help. In C#, it is accessed using the call
```csharp
m\_doc.get\_Elements(viewFilter, views);
```

In VB, you have to drop the "get\_" prefix, so the call is
```csharp
m\_doc.Elements(viewFilter, views)
```

**Reply:** Thanks for the response.
I did try with Elements, but this causes an error saying "property access must assign to the property or use its value", as it is returning an Integer.
This problem goes away if I use
```vbnet
Dim i as Integer = m\_doc.getElements(viewFilter, views)
```

What are we using the return value from this line for anyway?

**Answer:** Oops, yes, right you are.
The return value of these properties is the number of elements matching the specified criteria.

In VB, this is a read-only property, and the VB syntax requires you to make use of it as such.
This means that you must read the value returned by the property, regardless of whether you make use of it or not.

Maybe this is part of the reason why these members are considered methods instead of properties in C#.
In C#, you are not forced to read the return value, but you can, if you like.