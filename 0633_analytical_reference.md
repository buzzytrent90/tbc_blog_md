---
post_number: "0633"
title: "Reference to Analytical Curve"
slug: "analytical_reference"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'references', 'revit-api', 'views']
source_file: "0633_analytical_reference.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0633_analytical_reference.html"
---

### Reference to Analytical Curve

The
[Autodesk University 2011 Class Catalog](http://au.autodesk.com/?nd=au2011_class_catalog) is
now live and includes a full listing of both AU Las Vegas and AU Virtual Classes.
Here are the my three classes:

- Lecture [CP4451](http://au.autodesk.com/?nd=event_class&session_id=9263&jid=1725754): Extensible Storage in the Revit 2012 API- Hands-On Lab [CP6760-L](http://au.autodesk.com/?nd=event_class&session_id=9267&jid=1725754): Revit 2012 API Extensible Storage- Virtual [CP4453](http://au.autodesk.com/?nd=event_class&session_id=9726&jid=1725754): Everything in Place with Revit MEP Programming

So I have lots to prepare.
For a full list of the classes presented by my colleagues in the ADN team, please refer to
[Kean's post](http://through-the-interface.typepad.com/through_the_interface/2011/08/preview-of-the-au-2011-class-catalog.html).

Meanwhile, here is a structural issue for a change:

The Revit Structure API has undergone significant changes in every release lately.
Once the whole rebar functionality was updated, and in the last release it was the analytical model.
This obviously causes some complexity and upheaval.

In Revit Structure 2012, the analytical component was separated into its own independent Element, the AnalyticalModel element.
This is different from previous releases, where AnalyticalModel was a proxy for the analytical portion of the physical Element.
The interface to the AnalyticalModel was not changed, so the AnalyticalModel.getCurve method still works, for instance.

Therefore, each Structural Column and Structural Beam actually has two Elements in RST 2012: the physical portion, which is a FamilyInstance, and the
analytical portion, the AnalyticalModel.

One consequence of this is that the reference on the analytical model will be different than the reference on the physical element, because it
is not on the physical element any more.

Here is a question concerning this raised and also answered by Elizabeth Shulok of
[Structural Integrators, LLC](http://structuralintegrators.com)
on obtaining a
[stable geometric reference](http://wikihelp.autodesk.com/Revit/enu/2012/Help/API_Dev_Guide/0074-Revit_Ge74/0108-Geometry108/0111-Geometry111) to
the end point of the analytical curve of a structural member element that highlights some of these changes:

**Question:** I am trying to add a new point boundary condition to a Revit project via the API.
It requires a geometry reference to a beam, brace or column analytical line end.
In my code, after adding beams and columns and calling Regenerate, I find the column whose end coincides with the point of the boundary condition I want to add using a BoundingBoxContainsPointFilter ANDed with a StructuralInstanceUsageFilter.
The problem is that I cannot get a stable reference from the column's curve which I am getting from the AnalyticalModel.
Calling get\_EndPointReference(0) returns null.
What could cause the curve of a newly added column not to have a reference?
Another alternative approach that I also tried instead of calling GetCurve or Curve on the analytical model was to call get\_Geometry on the Element itself with the option argument ComputeReferences property set to true.
I was then able to obtain the curve from the GeometryElement returned from get\_Geometry, but that did not help either.

**Answer:** In versions prior to 2012, you could get the analytical model curve from the Element Geometry property (using the get\_Geometry method in C#).
However, in Revit 2012, the analytical model curves cannot be retrieved this way.
They can only be obtained from the AnalyticalModel class.

In this case, you can use the AnalyticalModelSelector.

We just added a new code sample using AnalyticalModelSelector to the Revit API help wiki;
look at the
[29-15 NewPointLoad() sample](http://wikihelp.autodesk.com/Revit/enu/2012/Help/API_Dev_Guide/0000-API_Deve0/0122-Disciple122/0126-Revit_St126/0129-Loads129) at
the bottom of the page.

Getting the geometric reference to the analytical curve start point like this works well:
```csharp
  AnalyticalModel analyticalModel = fiColumn
    .GetAnalyticalModel() as AnalyticalModel;

  Reference startReference = null;

  if( null != analyticalModel )
  {
    Curve curveCol = analyticalModel.GetCurve();
    if( null != curveCol )
    {
      AnalyticalModelSelector amSelector
        = new AnalyticalModelSelector( curveCol );

      amSelector.CurveSelector
        = AnalyticalCurveSelector.StartPoint;

      startReference = analyticalModel
        .GetReference( amSelector );
    }
  }
```

Many thanks to Elizabeth for her research and sharing this solution!