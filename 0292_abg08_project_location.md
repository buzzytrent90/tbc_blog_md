---
post_number: "0292"
title: "Project Location"
slug: "abg08_project_location"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'python', 'references', 'revit-api', 'walls', 'windows']
source_file: "0292_abg08_project_location.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0292_abg08_project_location.html"
---

### Project Location

This is part 8 of Scott Conover's AU 2009 class on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html),
dealing with the project location and its effect on element transformations.
We aready looked at the project location in the discussion on
[unrotate north](http://thebuildingcoder.typepad.com/blog/2009/10/unrotate-north.html).
However, as Scott pointed out, the results presented there are somewhat misleading and are subsumed by the following discussion.
The project location obviously also affects the
[azimuth](http://thebuildingcoder.typepad.com/blog/2008/10/azimuth.html) of
an element, i.e. the angle between the element and true north, and our previous discussion on that topic can also be replaced by this post.

#### Project location coordinates

Revit projects have another default coordinate system to take into account: the project location. The Document.ActiveProjectLocation provides access to the active ProjectPosition object, which contains:

- EastWest east/west offset (X offset).- NorthSouth the north/south offset (Y offset).- Elevation the difference in elevation (Z offset).- Angle the angle from true north.

        You can use these properties to construct a transform between the default Revit coordinates and the actual coordinates in the project:

        ```csharp
        // Obtain a rotation transform for the angle about true north
        Transform rotationTransform = Transform.get\_Rotation(
          XYZ.Zero, XYZ.BasisZ, project\_north\_angle );

        // Obtain a translation vector for the offsets
        XYZ translationVector = new XYZ(
          projectPosition.EastWest,
          projectPosition.NorthSouth,
          projectPosition.Elevation );

        Transform translationTransform
          = Transform.get\_Translation(
            translationVector );

        // Combine the transforms into one.
        Transform finalTransform
          = translationTransform.Multiply(
            rotationTransform );
        ```

        #### South Facing Elements Using Project Location

        In this example we adapt the
        [south-facing walls](http://thebuildingcoder.typepad.com/blog/2010/01/south-facing-walls.html) and
        [south-facing windows](http://thebuildingcoder.typepad.com/blog/2010/01/transformations.html#south_facing_windows) examples
        to deal with a project where true north is not aligned with the Revit coordinate system.
        The facing vectors are rotated by the angle from true north before the calculation is run, and the following walls are now determined to be south facing:

        ![South facing walls and windows using project location](img/abg8_south_facing_walls_using_project_location.png)

        The following windows are now facing south:

        ![South facing windows and windows using project location](img/abg8_south_facing_windows_using_project_location.png)

        Here is the method TransformByProjectLocation that we use to transform a direction vector by the rotation angle of the ActiveProjectLocation.
        It takes a given normalized direction as an input argument and returns the transformed location:

        ```python
        protected XYZ TransformByProjectLocation(
          XYZ direction )
        {
          // Obtain the active project location's position.

          ProjectPosition position
            = Document.ActiveProjectLocation.get\_ProjectPosition(
              XYZ.Zero );

          // Construct a rotation transform from the position angle.

          Transform transform = Transform.get\_Rotation(
            XYZ.Zero, XYZ.BasisZ, position.Angle );

          // Rotate the input direction by the transform
          XYZ rotatedDirection = transform.OfVector( direction );

          return rotatedDirection;
        }
        ```

        Please refer to Scott's
        [AU class material](http://au.autodesk.com/?nd=class&session_id=5256) for
        the full source code of his sample project.

        We will continue this series with a look at the powerful FindReferencesByDirection method coming up next.