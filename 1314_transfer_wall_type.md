---
post_number: "1314"
title: "Transferring a Wall Type"
slug: "transfer_wall_type"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'parameters', 'python', 'revit-api', 'transactions', 'walls']
source_file: "1314_transfer_wall_type.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1314_transfer_wall_type.html"
---

### Transferring a Wall Type

Let's look at transferring a wall type from one document to another.

In fact, we already did so, way back in 2011, as an example about how to possibly approach the task of at least partially programmatically [transferring project standards](http://thebuildingcoder.typepad.com/blog/2011/09/transfer-project-standards.html).

Now Parley submitted a [comment](http://thebuildingcoder.typepad.com/blog/2011/09/transfer-project-standards.html?cid=6a00e553e16897883301bb08275f46970d#comment-6a00e553e16897883301bb08275f46970d) on that, saying:

**Question:** We could still really use this...
This post originally was from 2011.
Any update on API access for this tool?

**Answer:** Glad to hear it sounds useful to you.

Well, nothing that I present here is really a tool, just sample source code for you to create your own tools from.

This should be pretty straightforward to migrate to Revit 2016, though.

On second thoughts, looking more closely at the text, I notice that I included the code above in The Building Coder samples as an external command CmdCopyWallType.

Therefore, it has been continually migrated every year, and the Revit 2015 version is provided in
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples).

The code for the [external command CmdCopyWallType](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCopyWallType.cs) is available there, already migrated to Revit 2015, and all the intervening versions as well.

As an added bonus, though, I went and tested the command for you.

I discovered that the transaction was not nicely encapsulated in a using statement, as it should be, so I fixed that.

I also discovered that a bug was introduced during the migration from Revit 2013 to 2014.
Apparently, this command was never tested in Revit 2014 and the error remained undetected ever since.
So I fixed that as well.

The Revit 2015 implementation now looks like this:

```python
[Transaction( TransactionMode.Manual )]
class CmdCopyWallType : IExternalCommand
{
  /// <summary>
  /// Source project to copy system type from.
  /// </summary>
  const string \_source\_project\_path
    = "Z:/a/case/sfdc/06676034/test/NewWallType.rvt";

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

    Document docHasFamily = app.OpenDocumentFile(
      \_source\_project\_path );

    // Find system family to copy, e.g. using a named wall type

    WallType wallType = null;

    FilteredElementCollector wallTypes
      = new FilteredElementCollector( docHasFamily ) // 2014
        .OfClass( typeof( WallType ) );

    int i = 0;

    foreach( WallType wt in wallTypes )
    {
      string name = wt.Name;

      Debug.Print( "  {0} {1}", ++i, name );

      if( name.Equals( \_wall\_type\_name ) )
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
        \_wall\_type\_name, \_source\_project\_path );

      return Result.Failed;
    }

    // Create a new wall type in current document

    using( Transaction t = new Transaction( doc ) )
    {
      t.Start( "Transfer Wall Type" );

      WallType newWallType = null;

      wallTypes = new FilteredElementCollector( doc )
        .OfClass( typeof( WallType ) ); // 2014

      foreach( WallType wt in wallTypes )
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
    }
    return Result.Succeeded;
  }
}
```

The updated version of The Building Code samples is
[release 2015.0.120.10](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.120.10).

So there you are.

Have fun!

**Addendum:** As Matt Taylor kindly points out below, the Revit 2014 API introduced the powerful
[copy and paste API](http://thebuildingcoder.typepad.com/blog/2013/05/copy-and-paste-api-applications-and-modeless-assertion.html),
which can be used to easily implement a more complete solution that the one presented above.