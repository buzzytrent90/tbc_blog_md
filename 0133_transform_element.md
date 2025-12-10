---
post_number: "0133"
title: "Transform an Element"
slug: "transform_element"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'parameters', 'references', 'revit-api', 'walls']
source_file: "0133_transform_element.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0133_transform_element.html"
---

### Transform an Element

**Question:**
How can I apply a transformation to a Revit element or family instance?

I used to work in the AutoCAD API, where I could apply a transformation using the transformBy() method.
It takes an AcGeMatrix3d parameter, which can represent a composed transformation including rotation, scaling, translation and mirroring.

In the Revit API, I see the use of the 4x3 matrix represented by the Transform class, which I suppose serves the same purpose.
However, I am unable to find a method like transformBy() to apply this transformation to a FamilyInstance or any other graphic element.

Therefore, my questions are:

1. Is there a way to apply a transformation matrix to a Revit element?
2. Can this matrix be applied containing simultaneous rotation, translation and mirroring transformations?
3. If there is no such function, how can I decompose a Transform to apply the transformation through the Move, Rotate and Mirror document commands?

**Answer:**
As much as possible I would recommend to avoid comparing the AutoCAD and Revit APIs.
In AutoCAD, everything is disconnected and anything can be done.
In Revit, everything is totally parameterised and connected, and almost no independent manipulation of objects is possible, because almost everything affects almost everything else.

For instance, imagine the effect of a transformation including a non-unit scale factor on a Revit element.
In the case of a wall, the wall type includes information on the wall thickness.
Blindly applying such a transformation to the wall would change its thickness, which would require it to change its type.
This is a rather far-reaching modification, more than you might have expected, coming from an environment like AutoCAD.
Therefore, the Revit API does not provide any method for scaling elements, or more generally applying general transformations to them.

The FamilyInstance class provides a property Location, which can be used to find the physical location of an instance within project. It overrides the base class Element.Location, which provides the same functionality for generic elements. As the description says, it is a read-only property.

The location object provides the ability to translate and rotate elements. More detailed location information and control can be found by using the derivatives of this object, such as LocationPoint and LocationCurve.

You might be able to make use of the Location.Move and Rotate methods for your purposes:

- Move: move the element within the project by a specified vector.
- Rotate: rotate the element within the project by a specified number of degrees around a given axis.

The document class provides similar methods to manipulate several elements simultaneously:

- Move: overloaded, moves one or more elements within the document by a specified vector.
- Rotate: overloaded, rotates one or more elements within the document by a specified number of degrees around a given axis.

The overloads are provided so you can specify the affected elements by individual element or element id, or by a whole set of each respectively.

The Document and Element classes also provide analogous methods to mirror one or more elements.

In exact details of what you can and cannot do will obviously depend on the circumstances, such as what kind of elements you are manipulating and what transformations you wish to apply.

The Revit API does not provide methods to decompose an arbitrary Transform instance, beyond the ones listed in the help file. For instance, you can check whether a transformation is a translation using IsTranslation, in which case the translation vector can be obtained from the Origin property.

Decomposing a matrix in the manner you describe is not a trivial task. Here is some old Silicon Graphics C++ code I found which achieves this and also includes some documentation and references for further reading and research. More references and background information is provided by the Wikipedia article on
[matrix decomposition](http://en.wikipedia.org/wiki/Matrix_decomposition):

```csharp
//
// Decompose a rotation into translation etc, based on scale
//
// Decomposes the matrix into a translation, rotation, scale,
// and scale orientation.  Any projection information is discarded.
// The decomposition depends upon choice of center point for
// rotation and scaling, which is optional as the last parameter.
// Note that if the center is 0, decompose() is the same as
// factor() where "t" is translation, "u" is rotation, "s" is scaleFactor,
// and "r" is ScaleOrientattion.
//
void
SbMatrix::getTransform(
  SbVec3f & translation,
  SbRotation & rotation,
  SbVec3f & scaleFactor,
  SbRotation & scaleOrientation,
  const SbVec3f ¢er ) const
{
  SbMatrix so, rot, proj;
  if (center != SbVec3f(0,0,0)) {
    // to get fields for a non-0 center, we
    // need to decompose a new matrix "m" such
    // that [-center][m][center] = [this]
    // i.e., [m] = [center][this][-center]
    // (this trick stolen from Showcase code)
    SbMatrix m,c;
    m.setTranslate(-center);
    m.multLeft(\*this);
    c.setTranslate(center);
    m.multLeft(c);
    m.factor(so,scaleFactor,rot,translation,proj);
  }
  else {
    this->factor(so,scaleFactor,rot,translation,proj);
  }
  scaleOrientation = so.transpose();  // have to transpose because factor gives us transpose of correct answer.
  rotation = rot;
}
//
// Factors a matrix m into 5 pieces: m = r s r^ u t, where r^
// means transpose of r, and r and u are rotations, s is a scale,
// and t is a translation. Any projection information is returned
// in proj.
//
// routines for matrix factorization taken from BAGS code, written by
// John Hughes (?).  Original comment follows:
//
/\* Copyright 1988, Brown Computer Graphics Group.  All Rights Reserved. \*/
/\* --------------------------------------------------------------------------
 \* This file contains routines to do the MAT3factor operation, which
 \* factors a matrix m:
 \*    m = r s r^ u t, where r^ means transpose of r, and r and u are
 \* rotations, s is a scale, and t is a translation.
 \*
 \* It is based on the Jacobi method for diagonalizing real symmetric
 \* matrices, taken from Linear Algebra, Wilkenson and Reinsch, Springer-Verlag
 \* math series, Volume II, 1971, page 204.  Call number QA251W623.
 \* In ALGOL!
 \* -------------------------------------------------------------------------\*/
/\*
 \* Variable declarations from the original source:
 \*
 \* n  : order of matrix A
 \* eivec: true if eigenvectors are desired, false otherwise.
 \* a  : Array [1:n, 1:n] of numbers, assumed symmetric!
 \*
 \* a  : Superdiagonal elements of the original array a are destroyed.
 \*    Diagonal and subdiagonal elements are untouched.
 \* d  : Array [1:n] of eigenvalues of a.
 \* v  : Array [1:n, 1:n] containing (if eivec = TRUE), the eigenvectors of
 \*    a, with the kth column being the normalized eigenvector with
 \*    eigenvalue d[k].
 \* rot  : The number of jacobi rotations required to perform the operation.
 \*/
SbBool
SbMatrix::factor(
  SbMatrix & r,
  SbVec3f & s,
  SbMatrix & u,
  SbVec3f & t,
  SbMatrix & proj ) const
{
  double    det;        /\* Determinant of matrix A    \*/
  double    det\_sign;    /\* -1 if det < 0, 1 if det > 0    \*/
  double    scratch;
  int        i, j;
  int        junk;
  SbMatrix    a, b, si;
  float    evalues[3];
  SbVec3f    evectors[3];

  a = \*this;
  proj.makeIdentity();
  scratch = 1.0;

  for (i = 0; i < 3; i++) {
    for (j = 0; j < 3; j++) {
      a.matrix[i][j] \*= scratch;
    }
    t[i] = matrix[3][i] \* scratch;
    a.matrix[3][i] = a.matrix[i][3] = 0.0;
  }
  a.matrix[3][3] = 1.0;

  /\* (3) Compute det A. If negative, set sign = -1, else sign = 1 \*/
  det = a.det3();
  det\_sign = (det < 0.0 ? -1.0 : 1.0);
  if (det\_sign \* det < 1e-12)
    return(FALSE);        // singular

  /\* (4) B = A \* A^  (here A^ means A transpose) \*/
  b = a \* a.transpose();

  b.jacobi3(evalues, evectors, junk);

  // find min / max eigenvalues and do ratio test to determine singularity

  r = SbMatrix(evectors[0][0], evectors[0][1], evectors[0][2], 0.0,
        evectors[1][0], evectors[1][1], evectors[1][2], 0.0,
        evectors[2][0], evectors[2][1], evectors[2][2], 0.0,
        0.0, 0.0, 0.0, 1.0);

  /\* Compute s = sqrt(evalues), with sign. Set si = s-inverse \*/
  si.makeIdentity();
  for (i = 0; i < 3; i++) {
    s[i] = det\_sign \* sqrt(evalues[i]);
    si.matrix[i][i] = 1.0 / s[i];
  }

  /\* (5) Compute U = R^ S! R A. \*/
  u = r \* si \* r.transpose() \* a;

  return(TRUE);
}
```

I believe that if the matrix is mirroring, then the scaling vector produced by the decomposition above will have one or two negative and one or two positive components. If all three are either positive or negative, then it is not a mirroring.

An easy way to determine whether a matrix is a mirroring transformation or not is to check its
[determinant](http://en.wikipedia.org/wiki/Determinant).
If it is negative, the transformation is a mirroring, I believe. Happily, the determinant is provided directly by the Revit API as a property of the Transform class.

In Revit 2010, we have some updated API functionality affecting this area, specifically the Instance.Transformed method and geometry of Instances. There is a paragraph in the What's New section of the help file on this topic:

#### Instance.Transformed[Transform] and geometry of Instances

This property has been removed. Obtain the transformed geometry of the instance using Instance.GetInstanceGeometry(), Instance.GetSymbolGeometry(Transform) and Instance.GetInstanceGeometry(Transform).

The two overloads compute the geometric representation of the instance and a transformation it, respectively.