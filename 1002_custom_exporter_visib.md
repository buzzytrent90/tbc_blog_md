---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.096458'
original_url: https://thebuildingcoder.typepad.com/blog/1002_custom_exporter_visib.html
post_number: '1002'
reading_time_minutes: 7
series: general
slug: custom_exporter_visib
source_file: 1002_custom_exporter_visib.htm
tags:
- elements
- filtering
- revit-api
- vbnet
- views
title: Determining Absolutely All Visible Elements
word_count: 1478
---

### Determining Absolutely All Visible Elements

Yesterday, I discussed the
[view filter API](http://thebuildingcoder.typepad.com/blog/2013/08/view-filter-api.html).

One issue mentioned that was not completely answered was the following:

**Question 3:** I would like to export only visible objects.
Using the element IsHidden method reports whether it was hidden using the Hide in View > Elements menu option, but ignores Hide in View filters, and so does the
[IsHiddenElementOrCategory](http://thebuildingcoder.typepad.com/blog/2009/11/visible-elements.html) method.

How can my application determine exactly what the user actually sees on the screen?
Our customers obviously expect this functionality for the export.

The cavalry is coming to the rescue here in the shape of Scott Conover of the Revit development team and Joel Spahn, software developer for
[Lighting Analysts, Inc.](http://www.agi32.com) and
[ElumTools](http://www.elumtools.com).

Scott and Joel explain how the new
[custom exporter framework](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html) can
be used to solve this completely, cleanly and efficiently.

I already showed how to make use of it to export to
[XML](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html#4),
[Collada](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html#5) and the
[ADN mesh data JSON](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html) file formats.

Scott and Joel point out that it can also be used in a more trivial fashion simply to determine the visibility of building elements, including all the different ways of controlling visibility, also covering elements in linked files.

Here is the description and part of the evolution of this project:

**Goal:** I want to find out if an element is visible in a view taking into account:

- Everything

1. Whether the element is 'hidden'.
2. Whether the element category (or one of its parent categories) is not visible.
3. Visibility filters, e.g. set in visibility graphics.
4. Etc.

- Crop Regions
- View Range
- Scope Box
- Depth Clipping
- Differences between Floor & Ceiling views
- Phases
- Design Options
- Etc.

I have built methods to take into account whether the category is hidden.
Must I now test all the filters and filter rules associated with the given view?
Or is there a shortcut?

A filtered element collector taking a view id argument would cover all of this, which would constitute a big time saver.

That works great!

Now for the inevitable 'however':

Given an element from a linked document (found via a filtered element collector), how can I tell if the linked element is visible in a view that lives in the main document?

The view settings don’t necessarily carry over from host to link, so using FilteredElementCollector in this way won’t work.

It is not possible to create a filtered element collector where the document being filtered is a linked document, but the view being filtered is in the main document.

It may be possible to use a tool that is seemingly not appropriate to the task: CustomExporter.

This new capability processes a 3D view and has callbacks for when Links begin and end, and Elements begin and end.

So it may be possible to build a tree of elements that are considered visible in this view by using this and always returning RenderNodeAction.Skip for elements so that the child nodes (faces, meshes) are not gathered and passed.

I was able to get a CustomExporter up and running in minutes to produce a map of visible element ids that I store in a Dictionary mapping Document to HashSet of ElementId.

With larger models I was experiencing an undesirable performance hit until I started returning RenderNodeAction.Skip from the OnElementBegin method.
I can return that for all elements, since all I need is the element id in the first place.

Returning Skip immediately is the right approach to avoid generation of data you don’t care about.

There is also the small matter of keeping track which document each element belongs to. After exporting the desired views you can call the ElementVisible property to determine whether an element is visible in one of the exported views.

One other point of note here is the choice between using the Document instance itself or the Doc.PathName as a dictionary key. I chose to use the document path as the key because I wanted to persist the data for a while, and wasn’t sure how long I could hold on to the Document object before it became invalid. It would certainly be cleaner to use the Document object as the key. Also I may at some point use a HashSet of Integer instead of ElementId if I need to persist the data.

I think Document.PathName is better for a key as of now.
The Document class does override Equals and GetHashCode, but they do not seem to work correctly in all scenarios.

All in all, this should provide a complete solution to the issue #3 listed above.

It also shows the implementation of a custom export context in VB.NET.

Here is the code implementing ElementsVisibleInViewExportContext and its ElementVisible property:

```vbnet
Public Class ElementsVisibleInViewExportContext
  Implements IExportContext

  Private Documents As New Stack(Of Document)
  Private Property Elements As New Dictionary( \_
    Of String, HashSet(Of ElementId))

  Public ReadOnly Property ElementVisible( \_
    ByVal doc As Document, ByVal id As ElementId) \_
    As Boolean

    Get
      Dim ids As HashSet(Of ElementId)

      If Elements.TryGetValue(doc.PathName, ids) Then
        If ids.Contains(id) Then

          Return True

        End If
      End If

      Return False
    End Get

  End Property

  Public Sub New(ByVal mainDocument As Document)

    Documents.Push(mainDocument)
    Elements.Add(mainDocument.PathName,
                 New HashSet(Of ElementId))

  End Sub

  Public Function Start() As Boolean \_
    Implements IExportContext.Start

    Return True

  End Function

  Public Sub Finish() \_
    Implements IExportContext.Finish

    'Nothing.

  End Sub

  Public Function OnViewBegin( \_
    ByVal node As ViewNode) \_
    As RenderNodeAction \_
    Implements IExportContext.OnViewBegin

    Return RenderNodeAction.Proceed

  End Function

  Public Sub OnViewEnd( \_
    ByVal elementId As ElementId) \_
    Implements IExportContext.OnViewEnd

    'Nothing.

  End Sub

  Public Function OnLinkBegin( \_
    ByVal node As LinkNode) \_
    As RenderNodeAction \_
    Implements IExportContext.OnLinkBegin

    Dim doc = node.GetDocument

    Documents.Push(doc)
    If Not Elements.ContainsKey(doc.PathName) Then
      Elements.Add(doc.PathName, \_
                   New HashSet(Of ElementId))
    End If

    Return RenderNodeAction.Proceed

  End Function

  Public Sub OnLinkEnd(ByVal node As LinkNode) \_
    Implements IExportContext.OnLinkEnd

    Dim doc = Documents.Pop()

  End Sub

  Public Function OnElementBegin( \_
    ByVal elementId As ElementId) \_
    As RenderNodeAction \_
    Implements IExportContext.OnElementBegin

    Elements(Documents.Peek.PathName).Add(elementId)

    Return RenderNodeAction.Skip

  End Function

  Public Sub OnElementEnd(ByVal elementId As ElementId) \_
    Implements IExportContext.OnElementEnd

    'Nothing.

  End Sub

  Public Function OnInstanceBegin( \_
    ByVal node As InstanceNode) As RenderNodeAction \_
  Implements IExportContext.OnInstanceBegin

    Return RenderNodeAction.Skip

  End Function

  Public Sub OnInstanceEnd(ByVal node As InstanceNode) \_
    Implements IExportContext.OnInstanceEnd

    'Nothing.

  End Sub

  Public Function OnFaceBegin(ByVal node As FaceNode) \_
    As RenderNodeAction \_
    Implements IExportContext.OnFaceBegin

    Return RenderNodeAction.Skip

  End Function

  Public Sub OnFaceEnd(ByVal node As FaceNode) \_
    Implements IExportContext.OnFaceEnd

    'Nothing.

  End Sub

  Public Sub OnMaterial(ByVal node As MaterialNode) \_
    Implements IExportContext.OnMaterial

    'Nothing.

  End Sub

  Public Sub OnPolymesh(ByVal node As PolymeshTopology) \_
    Implements IExportContext.OnPolymesh

    'Nothing.

  End Sub

  Public Sub OnRPC(ByVal node As RPCNode) \_
    Implements IExportContext.OnRPC

    'Nothing.

  End Sub

  Public Sub OnDaylightPortal( \_
    ByVal node As DaylightPortalNode) \_
    Implements IExportContext.OnDaylightPortal

    'Nothing.

  End Sub

  Public Sub OnLight(ByVal node As LightNode) \_
    Implements IExportContext.OnLight

    'Nothing.

  End Sub

  Public Function IsCanceled() As Boolean \_
    Implements IExportContext.IsCanceled

    Return False

  End Function

End Class
```

For the sake of completeness, here is
[CustomExporterElementVisibleInView.zip](zip/CustomExporterElementVisibleInView.zip) providing
this code in pure VB as well.

This works wonderfully and may be the only solution for determining the visibility of an element in a linked model due to the visibility settings in the host model.

I hope you find this useful.

Many thanks to Scott and Joel for their idea, development, testing and sharing!

**Addendum** by Arnošt Löbel: This information is very correct.
Using filters and visibility settings may give somehow adequate results, but widespread opinion in the Revit graphics team considers the only way of truly figuring out what is or is not visible in a view running it through an export context, e.g. via a Custom Exporter.

However, you do need to keep in mind and note that the current Custom Exporter only processes (and sends to an export context) items that would be rendered, which may not include all objects that are actually visible in a 3D view.
For example, model lines are not processed, because they do not render.

mein haus besichtigen
ueberlegen, ob es eventuell irgendwie in unser konzept und weitere ueberlegungen beruecksichtigt werden koennte und sollte :-)
sich weiter kennenlernen
nett zusammen zu abend essen