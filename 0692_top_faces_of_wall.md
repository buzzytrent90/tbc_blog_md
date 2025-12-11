---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.406992'
original_url: https://thebuildingcoder.typepad.com/blog/0692_top_faces_of_wall.html
post_number: 0692
reading_time_minutes: 5
series: general
slug: top_faces_of_wall
source_file: 0692_top_faces_of_wall.htm
tags:
- csharp
- levels
- revit-api
- views
- walls
title: Top Faces of Sloped Wall Update
word_count: 1060
---

### Top Faces of Sloped Wall Update

On Saturday we left Paris and arrived in Gteborg, or Go:teborg as the local tourist office appears to like spelling it.

On Sunday we went for a trip on one of the public transportation ferry boats out to Vrångö.
From right to left you see Jim, Partha, Philippe, and me, and Adam is taking the picture:

![DevTech team on Vrångö](img/vrango_jeremy_philippe_partha_jim.jpg)

Fittingly enough, I also had some correspondence on a Revit API issue with a Swedish developer on the same day:
Henrik Bengtsson of
[Lindab](http://www.lindab.se) submitted a
[comment](http://thebuildingcoder.typepad.com/blog/2011/07/top-faces-of-wall.html?cid=6a00e553e1689788330162fda970a9970d#comment-6a00e553e1689788330162fda970a9970d) on
the retrieval of the
[top faces of a sloped wall](http://thebuildingcoder.typepad.com/blog/2011/07/top-faces-of-wall.html):
> *When I tried parts of this code, I discovered that the .Normal function of the PlanarFace object sometimes shows incorrect values for faces with a positive or negative z-direction.
>
> This especially affects the code part:*
> ```csharp
> if( f is PlanarFace && PointsUpwards( ((PlanarFace)f).Normal ) )
> ```
> *A solution that so far has given me correct values is to replace the .Normal function of the PlanarFace object with the .ComputeNormal function.
>
> I found problems when analyzing faces that had a positive or negative z-value of its normal. So, not only the boundary faces of the wall but also top and bottom faces of its openings...*

Henrik also sent me a simple model with two simple quadrilateral sloped walls in which the existing
[CmdWallTopFaces](http://thebuildingcoder.typepad.com/blog/2011/07/top-faces-of-wall.html) command
implementation fails:

![Two sloped walls](img/wall_top_faces_v2.png)

The command reports zero top faces:

```
0 top faces found on Walls <194639 Lindab FR E120/120 202 M0 c450>
0 top faces found on Walls <195080 Lindab FR E120/120 202 M0 c450>
```

This is obviously false:

![Wall top faces](img/wall_top_faces_v2_3d.png)

Henrik implemented the following improved solution in VB calling ComputeNormal:
```vbnet
  For Each f As DB.Face In Solid.Faces
    '
    If TypeOf f Is DB.PlanarFace Then
      '
      Dim pf As DB.PlanarFace = f
      Dim p As XYZ = pf.Origin
      If pf.ComputeNormal(New DB.UV(p.X, p.Y)).Z > 0 Then
        '
        Dim faceVertices As IList(Of DB.XYZ) \_
          = pf.Triangulate().Vertices
        For Each v As DB.XYZ In faceVertices
          '
          If sideVertices.Contains(v, comparer) Then
            '
            If Not ret.Contains(f) Then
              '
              ret.Add(f)
              '
            End If
            Exit For
            '
          End If
          '
        Next
        '
      End If
      '
    End If
    '
  Next
```

He says:
> *I tried the .ComputeNormal function and it helps solving the problem. The .Normal function shows an incorrect x as well as z value for the walls that was in the project file.
>
> Depending on the vector of the top face, I am sure that x, y and z-directions will be incorrect.
>
> The function that this routine is implemented in looks like this (more or less a VB copy of your blog code):
>
> I have tried a couple of walls and sometimes a wall works correct, then if I run a mirror command on it, the new one becomes incorrect.
>
> A positive or negative top face slope has showed to have an impact as well.
>
> If there are openings inside the wall they can have an incorrect .Normal value as well. That was actually how I discovered it. The top face of a wall was pointing upwards
> and ended up in my collection as well.*

I decided I might as well clean up this command a bit more to ensure that top faces are really found.

If we make use of the Face.ComputeNormal method instead of the planar face normal vector, we might as well remove the restriction to planar faces at the same time.

To handle all kinds of faces, I can use the following simple method to determine whether a given face is facing upwards:

- Determine the face UV bounding box.- Pick a UV point in the middle.- Call ComputeNormal to determine the face normal vector at that point.- Check whether it points upwards.

This is still an extremely simplistic test, so there is no guarantee that it will handle the general case.
For instance, if the face has holes or a complex boundary, the middle point calculated may not belong to it at all.
It is up to you to test it for the cases you need.

The command already implements a bunch of other stuff to eliminate faces that belong to openings, and thus are not top faces of the wall, but just faces into the wall openings.
I left that part completely untouched.

I first replaced the line of code that I was previously using which was only checking for planar faces by a call to this new method which does the same thing:
```csharp
  /// <summary>
  /// Super-simple test whether a face is planar
  /// and its normal vector points upwards.
  /// </summary>
  static bool IsTopPlanarFace( Face f )
  {
    return f is PlanarFace
      && PointsUpwards( ( (PlanarFace) f ).Normal );
  }
```

I then went and implemented the new more general approach described above like this to replace it:
```csharp
  /// <summary>
  /// Simple test whether a given face normal vector
  /// points upwards in the middle of the face.
  /// </summary>
  static bool IsTopFace( Face f )
  {
    BoundingBoxUV b = f.GetBoundingBox();
    UV p = b.Min;
    UV q = b.Max;
    UV midpoint = p + 0.5 \* ( q - p );
    XYZ normal = f.ComputeNormal( midpoint );
    return PointsUpwards( normal );
  }
```

Lo and behold!
Running this on the simple model provided by Henrik returned the two faces, just as expected:

![Wall top face edges](img/wall_top_faces_v2_edges.png)

Problem apparently fixed, a least for this simple case.

I also ran the command on rac\_basic\_sample\_project.rvt again, like I did the first implementation.
It now reports 119 walls selected with 34 top faces when I select all walls in the Level 1 plan view, and 605 walls selected with 47 top faces when I do it in 3D view.
These results are both different from what I found in the simplified model last time, but I am not going to worry about that.

Anyway, here is
[version 2012.0.96.2](zip/bc_12_96.zip) of
The Building Coder samples including the updated command.
I am looking forward to hearing what new issues you run into with it.

Many thanks to Henrik for prompting this improvement!