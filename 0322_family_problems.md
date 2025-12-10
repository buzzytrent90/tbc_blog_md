---
post_number: "0322"
title: "Family Problems Missing Components"
slug: "family_problems"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'revit-api']
source_file: "0322_family_problems.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0322_family_problems.html"
---

### Family Problems Missing Components

The title of this question from my colleague Greg Wesner of Autodesk is so nice, I just had to add it to the blog, even though we have actually already provided the explanation a long time ago.

**Question: Family problems...** ... no, not that kind Revit Family problems :-)

This seems so simple, yet I'm way too tired to try to figure this out.
I'm porting some code that I didn't write from Revit 2009 to 2010.
I'm getting an error because of this line:
```csharp
foreach( Element e in m\_family.Components )
{
  // do something to element e
}
```

I looked it up in the Revit Platform API Changes document and found the following:

New document methods – Document.LoadFamily and Document.EditFamily – are introduced ... as a result, the properties of Family which access the contents have been removed (Family.SolidForms, Family.VoidForms, Family.Components, Family.LoadedSymbols, Family.Others).

I've been looking around for a while now, and I don't see any obvious way to get to the equivalent of 'Components' in the 2010 Family API.

The Revit 2009 SDK included the FamilyExplorer sample, but it seems it was dropped for 2010. Pity.
I searched your blog and haven't found anything obvious for this one.

Any ideas? Even just pointing me to a sample would be great.

**Answer:** Yes, that property was removed.
In Revit 2010, you can just open the family document instead and iterate over its elements in the normal fashion.
I actually did provide an explanation and explicit replacement code in the post describing
[porting The Building Coder samples](http://thebuildingcoder.typepad.com/blog/2009/05/porting-the-building-coder-samples.html) from
2009 to 2010.