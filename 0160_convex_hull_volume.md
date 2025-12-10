---
post_number: "0160"
title: "Convex Hull and Volume Computation"
slug: "convex_hull_volume"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'revit-api', 'rooms', 'walls']
source_file: "0160_convex_hull_volume.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0160_convex_hull_volume.html"
---

### Convex Hull and Volume Computation

Max raised an interesting question in a comment on the discussion on the
[calculation of 2D polygon areas](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html):

**Question:**
If I have an array of 3d points, how can I do to get volume information?

**Answer:**
The answer is maybe not quite as easy as you expected.
To calculate that volume, you have to solve two tasks:

- Determine the
  [convex hull](http://en.wikipedia.org/wiki/Convex_hull)
  of the given point cloud.
- Calculate the volume of the resulting 3D
  [polyhedron](http://en.wikipedia.org/wiki/Polyhedron).

Both of these steps are non-trivial.
A number of different
[convex hull algorithms](http://en.wikipedia.org/wiki/Convex_hull_algorithms)
exist both for the two-dimensional and for higher dimensional cases.
Several open source libraries for solid modelling or computational geometry implement these.
One of the best known and most reliable tools which is specifically targeted at these two issues is
[Qhull](http://www.qhull.org).
Here is the blurb from its home page:

Qhull computes the convex hull, Delaunay triangulation, Voronoi diagram, halfspace intersection about a point, furthest-site Delaunay triangulation, and furthest-site Voronoi diagram.
The source code runs in 2-d, 3-d, 4-d, and higher dimensions.
Qhull implements the Quickhull algorithm for computing the convex hull.
It handles roundoff errors from floating point arithmetic.
It computes volumes, surface areas, and approximations to the convex hull.

I downloaded the current version of Qhull to explore exactly what you might be able to use for your purposes.
The Qhull distribution includes a list of sample programs.
One of these is qconvex, and one of its output options is FA to compute total area and volume of the input points, which is pretty exactly what you want.

Now how can you make use of this inside the Revit API?

Qhull is implemented in standard C, the Revit API is a .NET environment.
The easiest way to make use of C source code from .NET is to compile a DLL and call it from .NET.
In this case you would need to analyse the source code for the qconvex program, which is contained in the file qconvex.c, and package the required functionality in a DLL that you make accessible from .NET.

The relevant lines of code from qconvex.c are:

```csharp
qh\_checkflags (qh qhull\_command, hidden\_options);
qh\_initflags (qh qhull\_command);
points= qh\_readpoints (&numpoints, &dim, &ismalloc);
qh\_init\_B (points, numpoints, dim, ismalloc);
qh\_qhull();
qh\_check\_output();
qh\_produce\_output();
```

This program reads its input from a file or the console standard input stream, performs its calculations, and produces its output into a file or the standard output stream.
You would need to adapt this to pass the information from and to the .NET calling application.

Another alternative, possible much simpler, but obviously less efficient, would be to set up the .NET application to write and read files in the expected Qhull input and output formats and then execute the unchanged qconvex executable provided by the Qhull package.

**Reply** from Max:
Very good this library, but we need a wrapper for .NET, and how can I encapsulate this library in a C# project?

What do you think about this alternative solution for
[polyhedra volume calculation](http://www.codeproject.com/KB/scripting/PolyhedraVolumeCalc.aspx?display=Print)?

**Answer:**
The polyhedra volume calculation article looks interesting, it seems like a simple approach for solving the second part of the problem, the volume calculation.
It still needs to be ported from Java to .NET, though.
It also does nothing to help you with the first part of the problem, the determination of the convex hull.

Revit will not automatically provide a convex hull for a room or any element's faces and edges.
Imagine an L-shaped room: its convex hull does not include some of the vertices of the walls in the inner corner.
Imagine any shape at all that is not already convex. You need to eliminate all 'inner' vertices to obtain the convex hull.

Regarding access to Qhull from .NET, I explained two methods for making use of the library from a .NET client above:

- Create a DLL from the C code, and implement a C function that can be called from .NET, read the input from .NET, and return the output to .NET.- Create a .NET application that writes and reads files in the expected Qhull input and output formats, and then execute the unchanged qconvex executable provided by the Qhull package as an external process using the files to communicate.
    Maybe you can use pipes instead of physical files on the disk.