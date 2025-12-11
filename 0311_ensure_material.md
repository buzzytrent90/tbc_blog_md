---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: code_example
optimization_date: '2025-12-11T11:44:13.725495'
original_url: https://thebuildingcoder.typepad.com/blog/0311_ensure_material.html
post_number: '0311'
reading_time_minutes: 4
series: materials
slug: ensure_material
source_file: 0311_ensure_material.htm
tags:
- elements
- family
- levels
- parameters
- python
- revit-api
- walls
- materials
title: Ensure Valid Material is Set
word_count: 752
---

### Ensure Valid Material is Set

In some of the very early posts, we explored how to access the materials assigned to an individual
[element](http://thebuildingcoder.typepad.com/blog/2008/10/element-materials.html), a
[family instance](http://thebuildingcoder.typepad.com/blog/2008/10/family-instance-materials.html), or a
[wall](http://thebuildingcoder.typepad.com/blog/2010/02/wall-solid-versus-face-materials.html).
However, we never presented anything to handle materials set at the category level.
Here is recent case handled by my colleague Saikat Bhattacharya on how to ensure that the material of a beam or column is set to a valid value, either at the element or category level:

**Question:** I am trying to extract the material associated with a framing or column family instance using the following code:
```vbnet
''' <summary>
''' Return the element id of the material for a given RST element.
''' </summary>
Shared Function MaterialId( \_
  ByRef familyComponent As FamilyInstance, \_
  ByRef revit As Application) As String

  Dim doc As Document = revit.ActiveDocument

  Dim oParameter As Parameter
  Dim o As Object
  Dim uid As String = ""

  For Each o In familyComponent.Parameters
    If Not TypeOf o Is Parameter Then
      Continue For
    End If
    oParameter = DirectCast(o, Parameter)

    Dim parameterName As String \_
      = oParameter.Definition.Name

    If (parameterName = "Column Material" Or \_
      parameterName = "Beam Material") And \_
      oParameter.StorageType = StorageType.ElementId Then

      Dim e As Element \_
        = doc.Element(oParameter.AsElementId())

      If TypeOf e Is Material Then
        uid = e.UniqueId
        Exit For
      End If
    End If
  Next o

  Debug.Assert(Not String.IsNullOrEmpty(uid))

  Return uid
End Function
```
This code works fine with the Revit Structure 2009 API but not with Revit Structure 2010.
I don't see any explanation for this change in the 'API changes and Additions' document.
I also notice that if I run the snoop tool on a family instance and select the 'Beam Material' parameter in the ParameterSet, I see a null value in some cases in 2010 whereas there is a valid value in 2009.
The code I used is similar to the Materials SDK sample.

**Answer:** I did a quick check by creating a set of columns and framing instances in RST 2010.
Using the
[RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html)
([for 2010](http://thebuildingcoder.typepad.com/blog/2009/10/rvtmgddbg-for-revit-2010.html))
snoop tool, I was able to see the material assigned to the family instances.
It does seem to work in 2010 in most circumstances.
However in some models, a null material value is returned.
This has to do with the beam or column material being set to 'By Category' in the element properties dialogue.
So we can refine your question to the following:

- How do I obtain a family instance element material through the API if its material is set to 'By Category'?- How can I set the material for the entire category through the API?

Looking at a specific model in more detail, I see that you can check whether the ByCategory material information is null though the user interface by looking at Manage > Project Settings > Settings > Object styles.
In this dialogue, you can see that the material information for structural columns and beams can be null.
In this case, we will see a null value using RvtMgdDbg even though at the instance level it is set to ByCategory.

I implemented the following code to help answer both your questions.
It returns the family instance element material, either for the given instance or the entire category.
If no element material is specified and the ByCategory material information is null, it creates a new material named GoodConditionMat on the fly and assigns it at the category level:
```python
public Material GetMaterial(
  Document doc,
  FamilyInstance fi )
{
  Material material = null;

  foreach( Parameter p in fi.Parameters )
  {
    Definition def = p.Definition;

    // the material is stored as element id:

    if( p.StorageType == StorageType.ElementId
      && def.ParameterGroup == BuiltInParameterGroup.PG\_MATERIALS
      && def.ParameterType == ParameterType.Material )
    {
      ElementId materialId = p.AsElementId();

      if( -1 == materialId.Value )
      {
        // invalid element id, so we assume
        // the material is "By Category":

        if( null != fi.Category )
        {
          material = fi.Category.Material;

          if( null == material )
          {
            MaterialOther mat
              = doc.Settings.Materials.AddOther(
                "GoodConditionMat" );

            mat.Color = new Color( 255, 0, 0 );

            fi.Category.Material = mat;

            material = fi.Category.Material;
          }
        }
      }
      else
      {
        material = doc.Settings.Materials.get\_Item(
          materialId );
      }
      break;
    }
  }
  return material;
}
```

Once this code is executed, you can see that the new material is now assigned at category level too:

![Category material](img/category_material.png)

Many thanks to Saikat for this solution!