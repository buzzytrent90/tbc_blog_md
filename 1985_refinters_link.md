---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.7
content_type: qa
optimization_date: '2025-12-11T11:44:17.195143'
original_url: https://thebuildingcoder.typepad.com/blog/1985_refinters_link.html
post_number: '1985'
reading_time_minutes: 21
series: general
slug: refinters_link
source_file: 1985_refinters_link.md
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- references
- revit-api
- selection
- sheets
- views
- walls
title: Refinters Link
word_count: 4245
---

### IFC, Dimension and Reference Intersector with Links
An extensive discussion on pros and cons of the reference intersector and how to use it with linked files and filtered element collectors, an update to the design automation IFC exporter, dimensioning with linked elements using the reference stable representation and some new forays with large language models:
- [Otto Desć](#1)
- [Reference intersector with filters and links](#2)
- [Revit IFC exporter for APS DA](#3)
- [Stable representation voodoo with links](#4)
- [Running Dalai LLaMa locally](#5)
- [ChatGPT invented a game – creative?](#6)
Before we get to the main topic, let's briefly mention
#### Otto Desć
The Oscar ceremony was punctuated by a series of advertisements by Autodesk.
Some pay homage to a mythical filmmaker and visual effects specialist named Otto Desćinski, or Otto Desć to his friends, e.g.,
the two-minute spot [Autodesk makes the software that makes the magic](https://youtu.be/MqMg9iAjwdw):
#### Reference Intersector with Filters and Links
A lot of interesting information and insights on pros and cons based on years of experience on using the reference intersector in conjunction with other filters and in linked files was discussed
by Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas
and Thomas Mahon of [Bimorph](https://bimorph.com) in the question
on [`ReferenceIntersector` + `BoundingBoxIntersectsFilter` fails on `RevitLinkInstance`](https://forums.autodesk.com/t5/revit-api-forum/referenceintersector-boundingboxintersectsfilter-fails-on/td-p/11782120) raised
by Chris Hanschen of [LKSVDD architecten](https://www.lksvdd.nl):
\*\*Question:\*\* I am a big fan of the ReferenceIntersector filter, finding elements like that is great!
In the post
on [how to use the `ElementIntersectsElementFilter` from the `RevitLinkInstance`](https://forums.autodesk.com/t5/revit-api-forum/how-to-use-the-elementintersectselementfilter-from-the/m-p/8440204),
Jeremy explains this slow filter and the use of combinations of filters.
The combination of `BoundingBoxIntersectsFilter` and `ReferenceIntersector` can boost the performance of the slow ReferenceIntersector Filter.
Even when using linked models, the ReferenceIntersector can find the link (at first) and the `ReferenceWithContext` can give you information about the element in the RevitLinkInstance, works great!
When the RevitLinkInstance is NOT moved, the combination of ReferenceIntersector + BoundingBoxIntersectsFilter works great, but when the RevitLinkInstance is moved, no elements are found.
It seems as if the BoundingBoxIntersectsFilter not only filters elements in the opened model, but also the elements in the linked model, by checking their coordinates or something.
But, when the RevitLinkInstance is moved in your opened models, the coordinates are different, and the BoundingBoxIntersectsFilter fails.
In this situation, the linked model is not found within the BoundingBoxIntersectsFilter, by the ReferenceIntersector, although it is there!
![ReferenceIntersector and BoundingBoxIntersectsFilter](img/referenceintersector_with_link_01.png "ReferenceIntersector and BoundingBoxIntersectsFilter")
The ray really hits the linked model, the linked element, within the (local) BoundingBoxIntersectsFilter, but nothing is found.
Looks like the ReferenceIntersector + BoundingBoxIntersectsFilter fails on moved RevitLinkInstance.
Am I doing something wrong or does the BoundingBoxIntersectsFilter really fail on the moved (elements in the) RevitLinkInstance?
\*\*Answer:\*\* Interesting question. The development team explain:
Since the filters mentioned do not know about the link's transform, they assume no transform exists.
You can't really affect the ElementInteresectsElementFilter, as it looks directly at the Element's geometry within the link, but you can apply the transform to the bounding box for BoundingBoxIntersectsFilter before you pass it in.
Note that rotations might cause a different size bounding box to be generated as the input bounding box is always aligned with whatever coordinate system is in the host model.
\*\*Response:\*\* Applying the transform to the bounding box for the BoundingBoxIntersectsFilter before passing it in would only be a possibility when you are just investigating this one link, no other links (with different transform) or elements in the opened model with the same Ray.
You say the filters mentioned do not know about the link's transform... Why not? When hitting a RevitLinkInstance, the transform is known, can't this be used?
The ReferenceIntersector is a great way to find ALL elements (multiplex links, with different transforms, and model elements) at the same time, with just 1 Ray. But it is a slow filter, so you want to be able to speed this up by using BoundingBoxIntersectsFilter. The Elements are there! the RevitLinkInstance should be found.
Is there a way to get this working properly?
\*\*Answer (R):\*\* Filters work within the current document only, so the bounding box is relevant to the space within that, i.e., within link document.
The link document doesn't know where it is placed in the host document, since that is information associated with the link instance in the host document.
Hence, that is why the filter doesn't know.
The ReferenceIntersector is not a slow filter, or any kind of filter at all; it is a utility to strike something with a ray based on origin and direction.
The ReferenceIntersector works within the current document where the various links take up a final specific known position; so, it is not analogous to an element filter or limitation of such.
The ReferenceIntersector will only find things based upon visibility of the elements in the 3D view provided.
So, that is one way of filtering beforehand what the RefereceIntersector strikes.
Although visibility control of elements in links via the API is fairly limited still.
Regarding links, the suggestion I believe is to transform the bounding box into the space of the linked document.
Not sure I follow your issue with transforming the bounding box to the link space.
If you have a link instance in multiple positions, then you only need to check one instance of that link.
Since transforming the box into the link document will result in the same target position in the link document?
Depends how you are using the bounding box filter to begin with?
You are interested in a certain region, so what link instances are in that region to transform the box into and check for initial elements?
The only real limitation, as mentioned, is that the bounding box is parallel to the document space and this may not be the same in the linked document as the host document.
Since the object you use is `Outline`, and that doesn't have a transform to give you an alternative orientation for the model it is used in.
However, it is a rough exercise for an initial coarse result.
I would probably create the bounding box corner points in the host document for a box aligned with the link transform axis, so that they match up when transformed into the link.
If you create the corner points based on a box aligned with the host document, then it may not represent the same when those points are transformed into the associated linked document positions.
\*\*Response:\*\* My issue: combination of BoundingBoxIntersectsFilter and ReferenceIntersector fails for LinkedDocumentElements, and it fails only when the LinkedInstance is moved.
I understand why, and I understand this is not easily solved, but for now I can't use this combination of filters.
The use of only ReferenceIntersector gives me the result I need, but when firering more than 1000 rays, the use of only ReferenceIntersector is way slower than the combination of these 2 filters.
The geometry is there, at the location given, in the 3D view given, so i did not for see this behavior.
Still a huge fan of the ReferenceIntersector Filter, but a little bit disappointed in the combination of these filters.
\*\*Answer (R):\*\* You know the link instance position via its transform, so you know where the bounding box needs to be relocated to suit that.
Below is some code that demonstrates using a logical `OR` filter with bounding box for each inverse link instance location.
I tested this on two instances of the same link relocated from where they were inserted (X=0, Y=0):
![ReferenceIntersector with two link instances](img/referenceintersector_with_link_02.png "ReferenceIntersector with two link instances")

```
  Private Function Obj_230305a(ByVal commandData As Autodesk.Revit.UI.ExternalCommandData,
ByRef message As String, ByVal elements As Autodesk.Revit.DB.ElementSet) As Result

    Dim UIApp As UIApplication = commandData.Application
    Dim UIDoc As UIDocument = commandData.Application.ActiveUIDocument
    If UIDoc Is Nothing Then Return Result.Cancelled Else
    Dim IntDoc As Document = UIDoc.Document

    Dim FEC As New FilteredElementCollector(IntDoc)
    Dim RvtLnks As List(Of RevitLinkInstance) =
        FEC.OfClass(GetType(RevitLinkInstance)).OfType(Of RevitLinkInstance).ToList

    Dim BBOrds As XYZ() = New XYZ(1) {New XYZ(-11.3, 10, -1), New XYZ(2.3, 31.9, 0.1)}
    Dim EFs As ElementFilter() = New ElementFilter(RvtLnks.Count - 1) {}

    For i = 0 To RvtLnks.Count - 1
      Dim RInst As RevitLinkInstance = RvtLnks(i)
      Dim Tinv As Transform = RInst.GetTransform.Inverse
      Dim Min As XYZ = Tinv.OfPoint(BBOrds(0))
      Dim Max As XYZ = Tinv.OfPoint(BBOrds(1))
      Dim OL As New Outline(Min, Max)
      EFs(i) = New BoundingBoxIsInsideFilter(OL)
    Next

    Dim LorF As New LogicalOrFilter(EFs.ToList)
    Dim V3D As View3D = TryCast(UIDoc.ActiveGraphicalView, View3D)
    If V3D Is Nothing Then Return Result.Cancelled Else

    Dim REFInt As New ReferenceIntersector(LorF, FindReferenceTarget.Element, V3D) With {.FindReferencesInRevitLinks = True}

    Dim R As Reference = Nothing
    Try
      R = UIDoc.Selection.PickObject(Selection.ObjectType.Element, "Pick ray line")
    Catch ex As Exception
      Return Result.Cancelled
    End Try
    Dim CE As CurveElement = TryCast(IntDoc.GetElement(R), CurveElement)
    If CE Is Nothing Then Return Result.Cancelled Else
    Dim LN As Line = TryCast(CE.GeometryCurve, Line)
    If LN Is Nothing Then Return Result.Cancelled Else

    Dim Res As List(Of ReferenceWithContext) = REFInt.Find(LN.GetEndPoint(0), LN.Direction)

    For i = 0 To Res.Count - 1
      Dim RwC As ReferenceWithContext = Res(i)
      Dim Rf As Reference = RwC.GetReference
      Debug.WriteLine($"{Rf.ElementId.IntegerValue}, {Rf.LinkedElementId?.IntegerValue}, {RwC.Proximity}")
    Next
    Return Result.Succeeded

  End Function
```

Output:

```
432129, 432128, 6.68864267244516
432129, 432128, 3.38973099408717
432129, 432168, 13.2864660291611
432129, 432168, 9.98755435080315
432142, 432128, 17.9884761277055
432142, 432128, 14.6895644493475
432142, 432168, 24.5862994844214
432142, 432168, 21.2873878060635
```

Eight faces, two linked element ids in each of the two element ids representing link instances i.e. four unique permutations of ElementId, LinkedElementId.
Can be noted that the faces are not returned in order of proximity.
Note no link instance in position where bounding box points are transformed to in host document:
![ReferenceIntersector with two link instances](img/referenceintersector_with_link_03.png "ReferenceIntersector  with two link instances")
\*\*Answer (T):\*\* The limitations you've encountered are explained in the ReferenceIntersector documentation.
The long and short of it is you have one option available to get reliable results where links are concerned, and that's using the `ReferenceIntersector` overload taking a `View3D` argument only.
This method is really slow, so you could follow what @RPTHOMAS108 suggested, 'transform' the BB to the location of the link instance then provide your element filter.
The problem with this is you'll need to do this for each link in your document and you'll end up accumulating live elements with each ReferenceIntersector + link elements from other links if their origin-to-origin location happens to coincide with your transformed BB.
Subsequently, you'll have the additional problem of identifying duplicates and omitting them from your combined list of results once all your ReferenceIntersector's have run.
![ReferenceIntersector documentation](img/referenceintersector_with_link_04.png "ReferenceIntersector documentation")
\*\*Answer (R):\*\* In reality, you need only one bounding box filter per link instance, which could be combined into a logical or filter as above.
So, not multiple reference intersections, just a single one or one for all the links combined and one for other elements (non-linked).
You would need to include a non transformed bounding box if looking for elements not in links.
All of the bounding boxes used in the `OR` filter will be overlapping in a similar position if they come from a single bounding box position that the ReferenceIntersector is using to focus on in the model.
That transformed bounding box remote position will wrongly capture elements in the model away from the focus of the ReferenceIntersector, but the ReferenceIntersector itself will rule them out.
I think further testing is required for cases where the link instance is moved parallel with the ReferenceIntersector ray.
It would probably be a good idea to separate out the links into their own ReferenceIntersector and limit the length of the ray.
I think there are probably simple cases we can prove where the above may go wrong.
If using multiple ReferenceIntersectors, then identifying duplicates would not be a major issue.
In any scenario we get back an ElementId and a linked ElementId, so a comparison based on that combination could be done with Distinct/Union etc.
\*\*Answer (T):\*\* That's a good point, but you're only going to get a unique element using `FindNearest`.
If you use `Find`, then you'll end up with duplicates as the BBs in a LogicOrFilter are iterated over.
I guess it could be solved by adding all the BBs together, if they happen to intersect.
If they don't but are in close proximity, then the same problem will occur.
Untransformed BBs also wont help to preclude linked elements (if the flag is set); for example, if the origin-to-origin location of the link happens to coincide with the BB, linked elements will end up in the result.
There's no good way to go about it really, but your suggestion to transform the bb is the way to go if thats what the @c.hanschen needs, otherwise see if there are better ways of achieving your end-goal.
\*\*Answer (R):\*\* Yes, it is a bit more complicated than initially considered.
We get back references and I think it is true to say that the elements not in links can be found via Reference.LinkedElementId being -1.
So, we could perhaps reduce the problem to two ReferenceIntersectors one for the links where we have to filter out the non-linked elements. The second ReferenceIntersector would not have the flag set, so only find elements in the main document.
I did some simple testing of problematic cases I thought would cause issues but didn't encounter such issues in practice.
However I can't foresee every scenario used so suggest @c.hanschen satisfies themselves with testing the specific cases they are likely to encounter.
\*\*Answer (T):\*\* The ReferenceIntersectors accepting linked elements will still collect any live elements in the host document if the ray hits, so removing duplicates post-process would still be necessary given this could occur in any given model.
I've just discovered another problem using a filter with linked elements flagged: any live elements will be returned 3 times, irrespective of whether it passes the filter. Linked elements seem to be returned twice (which might make sense assuming the ray hits both sides of a wall element, but when omitting linked elements, only one live element is returned instead so its inconsistent?). Not sure if this is a bug?
Looks like the performance hit from ReferenceIntersector(Autodesk.Revit.DB.View3D view3d) overload is the lesser of all evils!

24. public IList<int> GetElementsFromRayshoot(
25. Document document,
26. IList boundingBoxes,
27. XYZ origin,
28. XYZ rayDirection)
29. {
30. var filters = new List();
31. foreach (var boundingBox in boundingBoxes)
32. {
33. var outline = new Outline(boundingBox.Min, boundingBox.Max);
35. filters.Add(new BoundingBoxIntersectsFilter(outline));
36. }
38. var logicFilter = new LogicalOrFilter(filters);
40. var referenceIntersector = new ReferenceIntersector(
41. logicFilter, FindReferenceTarget.Element,
42. document.ActiveView)
43. {
44. FindReferencesInRevitLinks = true
45. };
47. var refWithContextResults = referenceIntersector.Find(origin, rayDirection);
49. var elementIds = new List<int>();
50. foreach (var refWithContext in refWithContextResults)
51. {
52. var reference = refWithContext.GetReference();
54. var linkedElementId = reference.LinkedElementId;
55. if (linkedElementId.Equals(ElementId.InvalidElementId))
56. elementIds.Add(reference.ElementId.IntegerValue);
57. else
58. elementIds.Add(linkedElementId.IntegerValue);
59. }
60. return elementIds;
61. }

Test model Revit 2022:

```
  BB1.Min/Max = (-12.4839, -8.3542, 0) , (-5.0602, -0.9306, 9.8425)
  BB2.Min/Max = (-11.4847, -12.3077, 0) , (-6.0974, -10.8641, 9.8425)
  Ray = XYZ.BasisX
  Origin = BB1.Min with Z+3
```

\*\*Answer (R):\*\* Interesting; I think it is varying the option set for FindReferenceTarget in order to find the elements (by nested references) in the link; then, as a consequence of that, it reports the references of the elements outside of the link.

```
FindReferencesInRevitLinks = True, FindReferenceTarget.Element
309836, 309843, REFERENCE_TYPE_SURFACE, 3.76375750430962 (Linked wall near face)
309836, 309843, REFERENCE_TYPE_SURFACE, 4.71520107386343 (Linked wall far face)
310123, -1, REFERENCE_TYPE_SURFACE, 2.84087372715355 (Wall far face)
310123, -1, REFERENCE_TYPE_SURFACE, 1.39730417334777 (Wall near face)
310123, -1, REFERENCE_TYPE_NONE, 1.39730417334777 (Wall element)

'FindReferencesInRevitLinks = False, FindReferenceTarget.Element
310123, -1, REFERENCE_TYPE_NONE, 1.39730417334777 (Wall element)

'FindReferencesInRevitLinks = False, FindReferenceTarget.All
310123, -1, REFERENCE_TYPE_SURFACE, 2.84087372715355 (Wall far face)
310123, -1, REFERENCE_TYPE_SURFACE, 1.39730417334777 (Wall near face)
310123, -1, REFERENCE_TYPE_NONE, 1.39730417334777 (Wall element)
```

![ReferenceIntersector with distances](img/referenceintersector_with_link_05.png "ReferenceIntersector with distances")
I've not checked regarding inclusion of elements outside of bounding box scope.
Currently, using the `OR` filter, I would have expected the two walls captured by the `OR` combination of the two bounding boxes.
When in reality only the top bounding box was required to capture both.
I think however it does demonstrate that the ReferenceIntersector isn't internally processing the two bounding boxes for the check in an unconsolidated way.
I think also we have not considered nested links but they are all instanced top level in reality. The requirement to supply a 3D view for the ReferenceIntersector often puts me off using it since there is the requirement to ensure the elements you are looking for are visible in the view and we know elements can be not visible for numerous reasons.
I wouldn't be using the ReferenceIntersector to find elements unless I knew the type of elements I was looking for with it i.e. unless I could limit the search to certain categories and understand what affects the visibility of such.
The bounding box and ReferenceIntersector combination is a bit of an odd one anyway since you know where the ray is pointing so know the region the bounding box should occupy. The bounding box size then therefore becomes about the extents along the ray to capture. The wider those extents the larger the overall bounding box but the ray should ideally pass through the centre of it (why put the box somewhere the ray will not hit). Would be easier to rule out the rays that don't pass through the box to start with if using some mass ray casting approach.
\*\*Answer (T):\*\* I would avoid using the ReferenceIntersector if possible; it's too inefficient.
Coincidentally, we just removed calls to the ReferenceIntersector in one of our apps having used it for over 2 years.
We found that as modelling complexity increased over this time, so did compute time, resulting in a performance increase of around 95% once we replaced it with an alternative (surface projection once, then storing this result in our object model).
The ReferenceIntersector is also inconsistent if other types of element filter are used, so, returning to the original post, your best bet (simple, consistent but slow) is to use the `ReferenceIntersector` overload taking *Autodesk.Revit.DB.View3D view3d*, as addition of filters create more problems than they solve; otherwise, see if there are alternatives you can use.
\*\*Response:\*\* Thanks for all your replies!
I agree with the final conclusion: combining the RefIntersector with other Filters creates more problems than it solves! Using Transform to change the Boundingboxfilter is no solution, it has to be done for all the link instances, so same conclusion here: creates more problems than it solves.
Thanks again for all your replies! Much appreciated!
#### Revit IFC Exporter for APS DA
My colleague Eason [@yiskang](https://twitter.com/yiskang) Kang updated
his [Revit IFC exporter for APS Design Automation](https://github.com/yiskang/forge-revit-ifc-exporter-appbundle#export-only-elements-visible-in-the-given-view-unique-id-via-inline-json).
It demonstrates how to implement a Revit exporter appbundle for the APS Design Automation API that supports the Revit IFC export options.
He now enhanced it to also support exporting IFC from a specific view.
Many thanks to Eason for implementing and documenting this useful solution!
#### Stable Representation Voodoo with Links
Eason also pointed out a neat use of the stable representation voodoo mentioned in the thread
on [creating dimensions for family instance in linked file](https://forums.autodesk.com/t5/revit-api-forum/create-dimensions-for-familyinstance-in-linked-file/m-p/8442391).
I want to create dimensions between grids in host and linked duct elements.
I have not found an official approach to address this need and am currently working around it by transforming the linked element’s reference like this:

24. private Reference MakeLinkedReference4Dimension(Document doc, Reference r)
25. {
26. if (r.LinkedElementId == ElementId.InvalidElementId) return null;
27. string[] ss = r.ConvertToStableRepresentation(doc).Split(':');
28. string res = string.Empty;
29. bool first = true;
30. foreach (string s in ss)
31. {
32. string t = s;
33. if (s.Contains("RVTLINK"))
34. {
35. if (res.EndsWith(":0")) { t = "RVTLINK"; }
36. else { t = "0:RVTLINK"; }
37. }
38. if (!first)
39. {
40. res = string.Concat(res, ":", t);
41. }
42. else
43. {
44. res = t;
45. first = false;
46. }
47. }
49. res += ":0:LINEAR"; //!<<< this line added by ADN team
50. return Reference.ParseFromStableRepresentation(doc, res);
51. }
53. var ductRef1 = new Reference(duct1);
54. Reference ductRefInHost = null;
55. Transform linkTtransform = null;
57. using (var collector = new FilteredElementCollector(this.Document))
58. {
59. var instance = collector.OfClass(typeof(RevitLinkInstance)).FirstElement() as RevitLinkInstance;
60. linkTtransform = instance.GetTotalTransform();
61. ductRefInHost = ductRef1.CreateLinkReference(instance);
62. ductRefInHost = this.MakeLinkedReference4Dimension(this.Document, ductRefInHost);
63. }

This solution is not ideal and seems only work for liner dimension with my tests.
I wonder if we can get the linked element reference in host without doing this.
#### Running Dalai LLaMa Locally
Dalai provides a 'dead simple way' to run LLaMA on your computer:
- [GitHub](https://github.com/cocktailpeanut/dalai)
- [Twitter](https://twitter.com/cocktailpeanut)
Huh? What is LLaMa?
LLaMa is
a [large language model](https://en.wikipedia.org/wiki/Large_language_model) from
Meta AI, similar to GPT, cf.
the [list of large language models](https://en.wikipedia.org/wiki/Large_language_model#List_of_large_language_models).
I went ahead and tried out the 'dead simple' approach.
All I did was enter the two prompts suggested, one to download and build, the second to run locally.
My first attempt to download and build failed after ten minutes or so, the second succeeded with no changes or intervention from my side.
So, after less than half an hour and just two command line prompts, I was up and running my own local LLaMa instance.
LLaMa handles prompts differently (and maybe less conveniently) than ChatGPT.
For example, if you want to know why the sky is blue, don't submit "Why is the sky blue?".
Instead, submit "The sky is blue because:" and press enter.
I followed that advice and prompted "The sky is blue because:".
The answer was rather weird.
Next, I tried "Gandhi is famous for:".
That produced a very sensible result, although I was surprised how long it went on for.
I noticed the parameter `n_predict` was set to 1000 and reduced it to 100.
With that setting, I was very happy indeed with the response to my third prompt, "grass is green because":
> grass is green because of the chlorophyll in leaves, but it would not be green without light.
Light energy enters plants through their surface and converts to chemical potential for growth;
this process takes place throughout most plant tissues including veins that conduct water and
nutrients from roots upwardly into growing shoots or leaf blades (Figure 1).
This is a fundamental dynamic of photosynthesis, the conversion of light energy by plants using CO2.
In addition to being an important
I see no Figure 1, and the result breaks off midsentence, after 79 words, including my four-word prompt.
But apparently `n_predict` really does control more or less how much to spout.
A final prompt to finish off, "The Revit API enables us to":
> The Revit API enables us to create our own custom applications for Revit.
There is a huge amount of functionality built into the base application which
we can use as-is, but there are also opportunities here that have not been
covered by Autodesk yet in their main product offerings (in my opinion).
For example – I’ve written an extension to allow users to create Revit
families from within Inventor.
This is a post about using the API to write applications for our own needs, so
This answer seems to include a pretty specific quote.
I repeated the same prompt several times.
The result was completely different for each, and each sounded as if taken from some specific source document.
![Dalai LLaMa](img/dalai_llama.png "Dalai LLaMa")
LLaMa does not seem to be as fine-tuned, well-managed and creative as ChatGPT (cf. [below](#6)),
but seems like a fun, useful and very impressive thing to have installed locally on my own system.
I tried it a few more times after writing this and am not so impressed after all.
ChatGPT is much more to my liking.
I wish I could install that locally.
#### ChatGPT Invented a Game – Creative?
[A puzzle aficionado used ChatGPT to create a new game that's basically 'reverse Sudoku'](https://www.businessinsider.com/sudoku-like-puzzle-game-online-chatgpt-sumplete-2023-3).
The result is online and has had tens of thousands of visitors play already:
- [Sumplete](https://sumplete.com)
![Sumplete](img/sumplete.png "Sumplete")