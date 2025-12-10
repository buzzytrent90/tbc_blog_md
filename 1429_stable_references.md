---
post_number: "1429"
title: "Stable References"
slug: "stable_references"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'levels', 'python', 'references', 'revit-api', 'selection', 'sheets', 'views']
source_file: "1429_stable_references.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1429_stable_references.html"
---

### Reference Stable Representation Magic Voodoo
Let's end the week with a truly magnificent contribution and research result provided by Scott Wilson in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160).
Scott responded to Pat Hague's recent thread
on [converting local family instance coordinate of a selected edge to project coordinates](http://forums.autodesk.com/t5/revit-api/convert-local-family-instance-coordinate-of-selected-edge-to/m-p/6282821),
saying:
> Yeah the Stable Reference Strings can be used to get at areas of the Geometry API that aren't fully exposed – I love playing around with them.
> Sometimes I stumble upon something cool such as this solution for a situation in which
> the [geometry returns no reference for a family instance](http://forums.autodesk.com/t5/revit-api/geometry-returns-no-reference-for-familyinstance/m-p/6025697).
I asked Scott whether he could summarise the gist of that discussion to publish here for easier reference and better legibility, and he very kindly obliged:
> No worries, glad to help.
> I made a post you can use for your blog if you like: [stable reference strings for Jeremy](http://forums.autodesk.com/t5/revit-api/for-jeremy-stable-reference-strings/m-p/6284505).
> Here's a summary of my investigations into stable reference strings created using `Reference.ConvertToStableRepresentation` and the Voodoo Magic that they can perform.
#### Analysis and Background Information
A reference stable representation string is a collection of UniqueIds combined with table indices, delimited by the colon `:` character.
A reference string for a `PlanarFace` of a `FamilyInstance` element – accessed through its `GetSymbolGeometry` method – looks like this:

```
  66a421da-90cb-4df2-8efe-c312e4f78aa0-0003d32b:0:INSTANCE:a2177cbb-03d8-4416-a4d8-8ada6fc0165a-0003ebf0:78:SURFACE
```

The string obtained from the reference of the same face accessed using `GetInstanceGeometry` looks like this:

```
  a2177cbb-03d8-4416-a4d8-8ada6fc0165a-0003ebf0:78:SURFACE
```

As you can see, this is identical to the last 3 sections of the symbol geometry reference.
The SymbolGeometry reference looks like 2 references of 3 tokens each.
The structured data contained in the string can be used to reliably access geometry objects that are not completely exposed by the API.
This is done by building a custom string using parts of an existing valid string obtained from the API.
The fist step is to tokenise the string:
```csharp
string refString = myReference
.ConvertToStableRepresentation( dbDoc );
string[] refTokens = refString.Split(
new char[] { ':' } );
```
This will give you an array of string tokens.
Here is my interpretation of each token's purpose – I could be wrong on some of them, as I've just inferred this information through experimentation, using symbol geometry string as an example:
- token0: 66a421da-90cb-4df2-8efe-c312e4f78aa0-0003d32b - UniqueId of the FamilyInstance Element.
- token1: 0 - Presumably an index into a collection within the FamilyInstance (I've not seen anything other than zero here so it could be that the collection always contains a single entry, or that other entries have some other special unknown use)
- token2: INSTANCE - signals that the referenced geometry object belongs to A FamilyInstance Element.
- token3: a2177cbb-03d8-4416-a4d8-8ada6fc0165a-0003ebf0 - UniqueId of the FamilySymbol Instance that describes the geometry of this FamilyInstance (multiple FamilyInstances that have identical Geometry will reference the same FamilySymbol Instance.
- token4: 78 - the index of the referenced geometry object within the FamilySymbol Instance's geometry table. index positions 0 through 8 are reserved for the special reference planes that have been specified in the family editor (explained later).
- token5: SURFACE - the type of geometry object referenced by this string.
I think the best way to try to follow the hierarchy is to read the tokens in reverse order as follows:
The referenced geometry object of type: SURFACE is at index: 78 of the FamilySymbol Instance with UniqueId: a2177cbb-03d8-4416-a4d8-8ada6fc0165a-0003ebf0 as represented by the FamilyInstance with UniqueId: 66a421da-90cb-4df2-8efe-c312e4f78aa0-0003d32b.
It bothers me that references from geometry objects accessed through `GetSymbolGeometry` specify the FamilyInstance while those from `GetInstanceGeometry` do not, yet the `GetInstanceGeometry` objects will be correctly transformed into project coordinates according to the FamilyInstance, where the `GetSymbolGeometry` objects are left in family coordinate space. This seems a little backwards to me.
Another strange behaviour is that the ElementId property of the reference from an object returned by `GetInstanceGeometry` does not even match the FamilyInstance it came from – it actually refers to the FamilySymbol instance instead.
Now for the Voodoo Magic that I promised:
#### Voodoo Magic
If you have a reference in SymbolGeometry format, as obtained from a Dimension or through the `Selection.PickObject` method, and you need to get the geometry object in its correct position according to the FamilyInstance, you can tokenise the reference string as demonstrated above and then rebuild it as an InstanceGeometry reference string by discarding the first 3 tokens and joining the remaining 3 using, ':' as the delimiter.
You then obtain the InstanceGeometry from the element linked to the original reference and loop through it until you find an object that has a stable reference string that exactly matches your custom made one.
```csharp
public static Edge GetInstanceEdgeFromSymbolRef(
Reference symbolRef,
Document dbDoc )
{
Edge instEdge = null;
Options gOptions = new Options();
gOptions.ComputeReferences = true;
gOptions.DetailLevel = ViewDetailLevel.Undefined;
gOptions.IncludeNonVisibleObjects = false;
Element elem = dbDoc.GetElement( symbolRef.ElementId );
string stableRefSymbol = symbolRef
.ConvertToStableRepresentation( dbDoc );
string[] tokenList = stableRefSymbol.Split(
new char[] { ':' } );
string stableRefInst = tokenList[3] + ":"
+ tokenList[4] + ":" + tokenList[5];
GeometryElement geomElem = elem.get_Geometry(
gOptions );
foreach( GeometryObject geomElemObj in geomElem )
{
GeometryInstance geomInst = geomElemObj
as GeometryInstance;
if( geomInst != null )
{
GeometryElement gInstGeom = geomInst
.GetInstanceGeometry();
foreach( GeometryObject gGeomObject
in gInstGeom )
{
Solid solid = gGeomObject as Solid;
if( solid != null )
{
foreach( Edge edge in solid.Edges )
{
string stableRef = edge.Reference
.ConvertToStableRepresentation(
dbDoc );
if( stableRef == stableRefInst )
{
instEdge = edge;
break;
}
}
}
if( instEdge != null )
{
// already found, exit early
break;
}
}
}
if( instEdge != null )
{
// already found, exit early
break;
}
}
return instEdge;
}
```
As I mentioned earlier, the first 9 positions are occupied by the special references that have been allocated in the family editor, with the following magic values:
- 0 = Left
- 1 = Center Left/Right
- 2 = Right
- 3 = Front
- 4 = Centre Front/Back
- 5 = Back
- 6 = Bottom
- 7 = Center Elevation
- 8 = Top
As you can see, this sequence matches the order they appear in the family editor reference type drop-down list.
These references are handy for creating Dimensions, but the API does not expose them fully. They exist in the element geometry but have the generic type of GeometryObject with no indication of their special purpose other than the index found in the reference string. To get one of these references, all you need to do is get any geometry object from the FamilyInstance using `GetSymbolGeometry` and use it's reference string as a template to build a custom reference that points to the special reference.
You just need to tokenise the reference string and replace `token4` with the index value from the list above for the reference you want.
`token5` should also be Replaced with `SURFACE`, but I've found that you can safely leave out the last token and the reference string will still be valid.
To simplify things, I created an enumeration for the indices and a static method that simplifies the process:
```csharp
public enum SpecialReferenceType
{
Left = 0,
CenterLR = 1,
Right = 2,
Front = 3,
CenterFB = 4,
Back = 5,
Bottom = 6,
CenterElevation = 7,
Top = 8
}
public static Reference GetSpecialFamilyReference(
FamilyInstance inst,
SpecialReferenceType refType )
{
Reference indexRef = null;
int idx = (int) refType;
if( inst != null )
{
Document dbDoc = inst.Document;
Options geomOptions = dbDoc.Application.Create
.NewGeometryOptions();
if( geomOptions != null )
{
geomOptions.ComputeReferences = true;
geomOptions.DetailLevel = ViewDetailLevel.Undefined;
geomOptions.IncludeNonVisibleObjects = true;
}
GeometryElement gElement = inst.get_Geometry(
geomOptions );
GeometryInstance gInst = gElement.First()
as GeometryInstance;
String sampleStableRef = null;
if( gInst != null )
{
GeometryElement gSymbol = gInst
.GetSymbolGeometry();
if( gSymbol != null )
{
foreach( GeometryObject geomObj in gSymbol )
{
if( geomObj is Solid )
{
Solid solid = geomObj as Solid;
if( solid.Faces.Size > 0 )
{
Face face = solid.Faces.get_Item( 0 );
sampleStableRef = face.Reference
.ConvertToStableRepresentation(
dbDoc );
break;
}
}
else if( geomObj is Curve )
{
Curve curve = geomObj as Curve;
sampleStableRef = curve.Reference
.ConvertToStableRepresentation( dbDoc );
break;
}
else if( geomObj is Point )
{
Point point = geomObj as Point;
sampleStableRef = point.Reference
.ConvertToStableRepresentation( dbDoc );
break;
}
}
}
if( sampleStableRef != null )
{
String[] refTokens = sampleStableRef.Split(
new char[] { ':' } );
String customStableRef = refTokens[0] + ":"
+ refTokens[1] + ":" + refTokens[2] + ":"
+ refTokens[3] + ":" + idx.ToString();
indexRef = Reference
.ParseFromStableRepresentation(
dbDoc, customStableRef );
GeometryObject geoObj = inst
.GetGeometryObjectFromReference(
indexRef );
if( geoObj != null )
{
String finalToken = "";
if( geoObj is Edge )
{
finalToken = ":LINEAR";
}
if( geoObj is Face )
{
finalToken = ":SURFACE";
}
customStableRef += finalToken;
indexRef = Reference
.ParseFromStableRepresentation(
dbDoc, customStableRef );
}
else
{
indexRef = null;
}
}
}
else
{
throw new Exception( "No Symbol Geometry found..." );
}
}
return indexRef;
}
```
There's a little bit of unnecessary code in that method, as it was quickly cobbled together from the code I was using for research, sorry.
That's all I have for now.
Hopefully someone finds it useful.
\*\*Response:\*\* Wow!
I am sure we will!
A number of people commented on Scott's original post and were extremely impressed, overawed even, by this analysis and truly magical voodoo.
So am I!
Many thanks to Scott for all his research, creating, sharing, documenting this, many other solutions, all the support and profound insights he provides so selflessly in the discussion forum!
P.S. I added Scott's sample code
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2016.0.127.3](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.127.3) in the
module [CmdDimensionInstanceOrigin.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdDimensionInstanceOrigin.cs#L27-L251).
#### Importance of Setting the Detail Level
An addendum by Pat Hague to Scott's original post:
I would like to stress the importance of setting
```csharp
gOptions.DetailLevel = ViewDetailLevel.Undefined;
```
I just spent a solid day trying to figure out why your code would work on some families, and not others.
Finally, I was able to determine that the family itself was controlling visibility of faces based on the view detail level.
Strangely enough, when I was blindly troubleshooting the code I tried setting the following to true:
```csharp
gOptions.IncludeNonVisibleObjects = true; // false
```
This still did not work.
Maybe I had something else wrong with my code at the time.
I don't know.
What I've been able to get to work every time (so far) is the following:
```csharp
gOptions.DetailLevel = doc.ActiveView.DetailLevel;
```
To me, this ensures that I can't select a face that isn't visible through a PickObject.
Thanks again for all of your voodoo!
\*\*Response by Scott:\*\* I'm a bit confused pat; are you saying that using `Undefined` is good or bad?
\*\*Answer:\*\* Lets do some testing... for instance, this structural column, I think from a stock Autodesk family, shows straight edges in `Medium` or `Coarse` views:
![Medium and Coarse](img/stable_reference_mc.png)
It shows fillet edges on `Fine`:
![Fine](img/stable_reference_fine.png)
If I try to get the instance face or edge of the above-mentioned structural column from the symbol reference (using your code), I get a NullReferenceException error if DetailLevel is set to Undefined:
```csharp
gOptions.DetailLevel = ViewDetailLevel.Undefined;
```
Presumably because the PickObject is allowing me to select an edge or face that is non visible – I'm not sure how to confirm this.
The same is also true if I purposely mismatch my View Detail Level and my Options.Detail Level. For example:
- Set View Detail Level to `Fine`
- Set Options.DetailLevel to `Medium`
If I try that same code on a family instance where visibility of edges or faces is not controlled by the view's detail level, then everything works out fine.
Looking up this property in the API help file I see the following:
> Type: Autodesk.Revit.DB.ViewDetailLevel: Value of the detail level. ViewDetailLevel.Undefined means no override is set.
My guess is that there is some sort of disconnect between what PickObject is allowing you to select and the Options class. I should also note that I tested this again to confirm that setting `IncludeNonVisibleObjects` to `true` or `false` made no difference in my test case.
To summarize, if I always make sure my `Options.DetailLevel` is set to that of my view detail level, then I shouldn't run into any problems (I hope).
\*\*Response:\*\* The Undefined setting was an attempt to make the code more generalised, It worked fine on a few cases I tested it with before posting it, but i might have just got lucky (or unlucky...). I thought that having `ViewDetailLevel` set to `Undefined` would provide the full set of geometry from which to find the matching face no-matter what the setting of the view where the selection was made. Maybe that's not how `ViewDetailLevel.Undefined` works. If it is as you say and it signifies to only return geometry that does not have the detail visibility override set, then to fix it, I would loop through on each `ViewDetailLevel` setting until the geometry was found. Thanks for testing it out and giving a heads-up. Btw, which documentation mentions "ViewDetailLevel.Undefined means no override is set"? I just checked and the docs say "View does not use Detail Level" which I think means something completly different to both mine and your interpretations; odd.
Out of interest have you changed the code to include the view in the geometry options?
If so, does removing it help?
\*\*Answer:\*\* Scott, I may have been looking in the wrong location on the api help documents.
I found that snippet of text under OverrideGraphicSettings.SetDetailLevel Method.
I am currently using the active view's detail level in my geometry options. For my case its the only reliable way I can ensure that the user doesn't get an exception error.
```csharp
gOptions.DetailLevel = doc.ActiveView.DetailLevel;
```
#### Reformat Stable Representation String for Dimensioning
Joshua Lumley added an important note on using the stable references to create dimensioning in
his [comment below](http://thebuildingcoder.typepad.com/blog/2016/04/stable-reference-string-magic-voodoo.html#comment-4068022721):
When creating dimensions based on objects from linked files, you may see an error message saying \*Invalid number of references\*.
To resolve that, all you do is rearrange the string a little to match the following format:

```
  fb332d47-8286-4829-bd40-46c26de8ebac-000258d5:0:RVTLINK:2796184:1:SURFACE/5"
```

You do NOT want this:

```
  fb332d47-8286-4829-bd40-46c26de8ebac-000258d5:RVTLINK/fb332d47-8286-4829-bd40-46c26de8ebac-000258d4:2796184:1:SURFACE/5"
```

Many thanks to Joshua for pointing it out!
#### Handle Element Category Override
Mustafa Khalil invested some research in this and added
another [comment below](https://thebuildingcoder.typepad.com/blog/2016/04/stable-reference-string-magic-voodoo.html#comment-4710182554),
explaining:
I played with this for a while and was impressed with this detailed study.
For some reason, however, it sometimes did not work for me.
While cracking on every angle to find out why, I realized I should consider how the element is actually displayed in the current view.
Meaning, I should not only rely on the `ViewDetail` of the current view, but I should also check if there is any ViewDetail category overridden as well.
If there is no override to an element category, it will report `ViewDetailLevel.Undefined`.
Then, it is trustworthy to select the ViewDetail of the current view.
So, I considered the following resolution:
```csharp
var view = doc.ActiveView;
var overriddenCatView = view.GetCategoryOverrides(
elem.Category.Id );
op = new Options()
{
DetailLevel = overriddenCatView.DetailLevel
== ViewDetailLevel.Undefined
? view.DetailLevel
: overriddenCatView.DetailLevel,
IncludeNonVisibleObjects = true
};
```
Instead of this:
```csharp
op = new Options()
{
DetailLevel = m_doc.ActiveView.DetailLevel,
IncludeNonVisibleObjects = true
};
```
Many thanks to Mustafa for his research and explanation!