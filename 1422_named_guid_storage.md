---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: code_example
optimization_date: '2025-12-11T11:44:15.982971'
original_url: https://thebuildingcoder.typepad.com/blog/1422_named_guid_storage.html
post_number: '1422'
reading_time_minutes: 5
series: general
slug: named_guid_storage
source_file: 1422_named_guid_storage.md
tags:
- elements
- filtering
- python
- revit-api
- sheets
- transactions
title: Named Guid Storage
word_count: 918
---

### Named Guid Storage for Project Identification
I want to continue working on
the [TrackChangesCloud](https://github.com/jeremytammik/TrackChangesCloud) project asap.
So far, it only consists of the Revit add-in to determine and list the changes made to the BIM.
The interesting part will be to store the results in a cloud database for analysis and reporting.
A prerequisite for that is a reliable way to identify Revit project documents.
I already explored that topic
when [starting to implement](http://thebuildingcoder.typepad.com/blog/2015/07/firerating-and-the-revit-python-shell-in-the-cloud-as-web-servers.html#4)
the [FireRatingCloud](https://github.com/jeremytammik/FireRatingCloud) sample, writing
about [implementing Mongo database relationships](http://the3dwebcoder.typepad.com/blog/2015/07/implementing-mongo-database-relationships.html)
and [identifying a RVT project](http://the3dwebcoder.typepad.com/blog/2015/07/implementing-mongo-database-relationships.html#2).
I was not completely happy with that solution, and have not heard of any really perfect way to achieve what I want, so I decided to try a different tack this time:
Simply create my own Guid for the current Revit project and use that to identify it globally forever after.
Actually, I decided for a slightly more generic approach, supporting 'named Guid storage'.
I define an extensible storage schema named `JtNamedGuidStorageSchema` that just stores one single Guid object.
To create a new project identifier, I create a GUID and store it in an extensible storage entity with that schema on a Revit `DataStorage` element with a specific element name, e.g., `TrackChanges_project_identifier`.
To search for an existing project identifier, I can filter for all data storage elements with extensible storage entities containing data matching our specific schema and with the given element name.
One danger in this approach is that an existing project that already defines its own identifier might be copied to one or more follow-up projects so that its identifier is retained and reused.
Ah well, I guess I will live with that.
#### JtNamedGuidStorage Implementation Class
Here is the new `JtNamedGuidStorage` class that I just implemented and added
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples):

```
class JtNamedGuidStorage
{
  ///
  /// The extensible storage schema,
  /// containing one single Guid field.
  ///
  public static class JtNamedGuidStorageSchema
  {
    public readonly static Guid SchemaGuid = new Guid(
      "{5F374308-9C59-42AE-ACC3-A77EF45EC146}" );

    ///
    /// Retrieve our extensible storage schema
    /// or optionally create a new one if it does
    /// not yet exist.
    ///
    public static Schema GetSchema(
      bool create = true )
    {
      Schema schema = Schema.Lookup( SchemaGuid );

      if( create && null == schema )
      {
        SchemaBuilder schemaBuilder =
          new SchemaBuilder( SchemaGuid );

        schemaBuilder.SetSchemaName(
          "JtNamedGuidStorage" );

        schemaBuilder.AddSimpleField(
          "Guid", typeof( Guid ) );

        schema = schemaBuilder.Finish();
      }
      return schema;
    }
  }

  ///
  /// Retrieve an existing named Guid
  /// in the specified Revit document or
  /// optionally create and return a new
  /// one if it does not yet exist.
  ///
  public static bool Get(
    Document doc,
    string name,
    out Guid guid,
    bool create = true )
  {
    bool rc = false;

    guid = Guid.Empty;

    // Retrieve a DataStorage element with our
    // extensible storage entity attached to it
    // and the specified element name.

    ExtensibleStorageFilter f
      = new ExtensibleStorageFilter(
        JtNamedGuidStorageSchema.SchemaGuid );

    DataStorage dataStorage
      = new FilteredElementCollector( doc )
        .OfClass( typeof( DataStorage ) )
        .WherePasses( f )
        .Where<Element>( e => name.Equals( e.Name ) )
        .FirstOrDefault<Element>() as DataStorage;

    if( dataStorage == null )
    {
      if( create )
      {
        using( Transaction t = new Transaction(
          doc, "Create named Guid storage" ) )
        {
          t.Start();

          // Create named data storage element

          dataStorage = DataStorage.Create( doc );
          dataStorage.Name = name;

          // Create entity to store the Guid data

          Entity entity = new Entity(
            JtNamedGuidStorageSchema.GetSchema() );

          entity.Set( "Guid", guid = Guid.NewGuid() );

          // Set entity to the data storage element

          dataStorage.SetEntity( entity );

          t.Commit();

          rc = true;
        }
      }
    }
    else
    {
      // Retrieve entity from the data storage element.

      Entity entity = dataStorage.GetEntity(
        JtNamedGuidStorageSchema.GetSchema( false ) );

      Debug.Assert( entity.IsValid(),
        "expected a valid extensible storage entity" );

      if( entity.IsValid() )
      {
        guid = entity.Get<Guid>( "Guid" );

        rc = true;
      }
    }
    return rc;
  }
}
```

#### CmdNamedGuidStorage Test Command
As you can see, the implementation defines one single public entry point `Get`.
By default, it retrieves an existing named Guid from the given Revit document, and creates a new one if none already exists.
The creation can be suppressed by passing a `false` value for the `create` argument.
I exercise it in the trivial external command `CmdNamedGuidStorage` as follows:

```
  Result rslt = Result.Failed;

  string name = "TrackChanges_project_identifier";
  Guid named_guid;

  bool rc = JtNamedGuidStorage.Get( doc,
    name, out named_guid, false );

  if( rc )
  {
    Util.InfoMsg( string.Format(
      "This document already has a project "
      + "identifier: {0} = {1}",
      name, named_guid.ToString() ) );

    rslt = Result.Succeeded;
  }
  else
  {
    rc = JtNamedGuidStorage.Get( doc,
      name, out named_guid, true );

    if( rc )
    {
      Util.InfoMsg( string.Format(
        "Created a new project identifier "
        + "for this document: {0} = {1}",
        name, named_guid.ToString() ) );

      rslt = Result.Succeeded;
    }
    else
    {
      Util.ErrorMsg( "Something went wrong" );
    }
  }
  return rslt;
```

Here is the message box displayed and logged on the Visual Studio debug output console by the test command on creating a new named Guid storage:
![Retrieving a named Guid from extensible storage on a DataStorage element](img/named_guid_storage_01.png)
A similar message is generated on retrieving an existing identifier:

```
This document already has a project identifier: TrackChanges_project_identifier = 4223cc56-e6ae-4ab9-92da-1da69c72bd10
```

The new functionality discussed above is included
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2016.0.127.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.127.1).
I look forward to hearing what you think of it.
Thank you in advance for any comments you may have.
Now I am excited to get going with the TrackChangesCloud project again, after spending a lot of time last week answering Revit cases
and [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) questions.