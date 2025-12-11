---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.330923'
original_url: https://thebuildingcoder.typepad.com/blog/0650_transfer_project_standard.html
post_number: '0650'
reading_time_minutes: 5
series: general
slug: transfer_project_standard
source_file: 0650_transfer_project_standard.htm
tags:
- elements
- family
- parameters
- python
- revit-api
- transactions
- views
- walls
- windows
title: Transfer Project Standards
word_count: 1006
---

### Transfer Project Standards

Developers occasionally ask about programmatic access to the 'Transfer Project Standards' user interface functionality.
Unfortunately, that is currently not provided by the Revit API.
However, it is possible to implement a workaround, although it would require quite a bit of work and testing to ensure that it fulfils your needs, as the following sample by Joe Ye shows:

**Question:** Is it possible through the 2012 Revit API to transfer project standards and or system families between Revit projects?

For example, through the Revit user interface, I can insert drafting views from another project.
I can also transfer wall types, fill patterns, line styles, text types, and various other project wide settings from one project to another.
Alternatively, if I only want to transfer the types related to one object, I can copy and paste a complex object like a railing from one project into another.
This will not only copy the railing type but also the related types such as rail profiles and baluster families.

Is there a way to automate this through the Revit 2012 API?

**Answer:** I am sorry to say that there is currently no API access to the built-in 'Transfer Project Standards' functionality.

A possible workaround would obviously be to read the system types of interest from the source project, determine similar existing elements in the target project, use
[the Duplicate method](http://thebuildingcoder.typepad.com/blog/2011/02/system-family-creation.html)
to create new target instances of them, and copy all relevant properties from the source to the new target elements.

I implemented a new external command CmdCopyWallType for The Building Coder solution from Joe's sample code that shows the principles of implementing this for a wall type.

The source wall type is selected from an existing project by name, an equivalent new target wall type is created in the current project, and the properties are copied across.

Some aspects are not handled here, e.g. element-id-valued and shared parameters, other properties, the compound structure, etc.

It does give you the general idea, however, proves feasibility in principle, and provides a starting point if you really want to take this further.

Here is the CmdCopyWallType implementation:
```python
[Transaction( TransactionMode.Manual )]
class CmdCopyWallType : IExternalCommand
{
  /// <summary>
  /// Source project to copy system type from.
  /// </summary>
  const string \_source\_project\_path
    = "C:/a/j/adn/case/bsd/06676034/test/NewWallType.rvt";

  /// <summary>
  /// Source wall type name to copy.
  /// </summary>
  const string \_wall\_type\_name = "NewWallType";

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    // Open source project

    Document docHasFamily = app.OpenDocumentFile( \_source\_project\_path );

    // Find system family to copy, e.g. using a named wall type

    WallType wallType = null;

    foreach( WallType wt in docHasFamily.WallTypes )
    {
      if( wt.Name.Equals( \_wall\_type\_name ) )
      {
        wallType = wt;
        break;
      }
    }

    if( null == wallType )
    {
      message = string.Format(
        "Cannot find source wall type '{0}'"
        + " in source document '{1}'. ",
        \_source\_project\_path,
        \_wall\_type\_name );

      return Result.Failed;
    }

    // Create a new wall type in current document

    Transaction t = new Transaction( doc );

    t.Start( "Transfer Wall Type" );

    WallType newWallType = null;

    foreach( WallType wt in doc.WallTypes )
    {
      if( wt.Kind == wallType.Kind )
      {
        newWallType = wt.Duplicate( \_wall\_type\_name )
          as WallType;

        Debug.Print( string.Format(
          "New wall type '{0}' created.",
          \_wall\_type\_name ) );

        break;
      }
    }

    // Assign parameter values from source wall type:

#if COPY\_INDIVIDUAL\_PARAMETER\_VALUE
    // Example: individually copy the "Function" parameter value:

    BuiltInParameter bip = BuiltInParameter.FUNCTION\_PARAM;
    string function = wallType.get\_Parameter( bip ).AsString();
    Parameter p = newWallType.get\_Parameter( bip );
    p.Set( function );
#endif // COPY\_INDIVIDUAL\_PARAMETER\_VALUE

    Parameter p = null;

    foreach( Parameter p2 in newWallType.Parameters )
    {
      Definition d = p2.Definition;

      if( p2.IsReadOnly )
      {
        Debug.Print( string.Format(
          "Parameter '{0}' is read-only.", d.Name ) );
      }
      else
      {
        p = wallType.get\_Parameter( d );

        if( null == p )
        {
          Debug.Print( string.Format(
            "Parameter '{0}' not found on source wall type.",
            d.Name ) );
        }
        else
        {
          if( p.StorageType == StorageType.ElementId )
          {
            // Here you have to find the corresponding
            // element in the target document.

            Debug.Print( string.Format(
              "Parameter '{0}' is an element id.",
              d.Name ) );
          }
          else
          {
            if( p.StorageType == StorageType.Double )
            {
              p2.Set( p.AsDouble() );
            }
            else if( p.StorageType == StorageType.String )
            {
              p2.Set( p.AsString() );
            }
            else if( p.StorageType == StorageType.Integer )
            {
              p2.Set( p.AsInteger() );
            }
            Debug.Print( string.Format(
              "Parameter '{0}' copied.", d.Name ) );
          }
        }
      }

      // Note:
      // If a shared parameter parameter is attached,
      // you need to create the shared parameter first,
      // then copy the parameter value.
    }

    // If the system family type has some other properties,
    // you need to copy them as well here. Reflection can
    // be used to determine the available properties.

    MemberInfo[] memberInfos = newWallType.GetType()
      .GetMembers( BindingFlags.GetProperty );

    foreach( MemberInfo m in memberInfos )
    {
      // Copy the writable property values here.
      // As there are no property writable for
      // Walltype, I ignore this process here.
    }

    t.Commit();

    return Result.Succeeded;
  }
}
```

It prints out some of its actions in the debug output window.

As you can see from the following list, some properties cannot be copied without adding more intelligence, so there is not even a guarantee that the resulting wall style will work as expected:

```
New wall type 'NewWallType' created.
Parameter 'Description' copied.
Parameter 'URL' copied.
Parameter 'Type Comments' copied.
Parameter 'Fire Rating' copied.
Parameter 'Cost' copied.
Parameter 'Assembly Code' copied.
Parameter 'Structure' not found on source wall type.
Parameter 'Coarse Scale Fill Pattern' is an element id.
Parameter 'Wrapping at Inserts' copied.
Parameter 'Function' copied.
Parameter 'Model' copied.
Parameter 'Manufacturer' copied.
Parameter 'Type Mark' copied.
Parameter 'Coarse Scale Fill Color' copied.
Parameter 'Assembly Description' is read-only.
Parameter 'Keynote' copied.
Parameter 'Width' is read-only.
Parameter 'Wrapping at Ends' copied.
```

As said, this is just an idea and a starting point for further exploration.
It is a long way removed from providing the full functionality to transfer project standards.

Here is
[version 2012.0.92.0](zip/bc_12_92.zip) of
The Building Coder samples including both the command
[CmdFilledRegionCoords](http://thebuildingcoder.typepad.com/blog/2011/09/filledregion-corrdinates.html) that
I presented last week and the new command CmdCopyWallType.