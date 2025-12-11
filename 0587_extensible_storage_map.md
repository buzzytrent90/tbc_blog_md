---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.218578'
original_url: https://thebuildingcoder.typepad.com/blog/0587_extensible_storage_map.html
post_number: 0587
reading_time_minutes: 5
series: general
slug: extensible_storage_map
source_file: 0587_extensible_storage_map.htm
tags:
- csharp
- elements
- filtering
- levels
- python
- references
- revit-api
- selection
- transactions
- walls
title: Extensible Storage of a Map
word_count: 1077
---

### Extensible Storage of a Map

I already discussed the important new and powerful
[extensible storage](http://thebuildingcoder.typepad.com/blog/2011/04/extensible-storage.html) mechanism,
and we also explained and demonstrated it in both the
[DevHelp Online](http://thebuildingcoder.typepad.com/blog/2011/04/devdays-2010-online-with-revit-2012-api-news.html)
and
[Revit 2012 API webcast](http://thebuildingcoder.typepad.com/blog/2011/05/revit-2012-api-webcast.html) presentations,
both of which were recorded.

I updated the sample code for the latter webcast.
Here is the mainline of the Execute method:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  try
  {
    // pick an element and define the XYZ
    // data to store at the same time

    Reference r = uidoc.Selection.PickObject(
      ObjectType.Face,
      new WallFilter(),
      "Please pick a wall at a point on one of its faces" );

    Wall wall = doc.get\_Element( r.ElementId ) as Wall;
    XYZ dataToStore = r.GlobalPoint;

    Transaction t = new Transaction( doc,
      "Create Extensible Storage Schemata and Store Data" );

    t.Start();

    // store the data, and also
    // demonstrate reading it back

    StoreDataInWall( wall, dataToStore );

    t.Commit();

    // list all schemas in memory across all documents

    ListSchemas();

    return Result.Succeeded;
  }
  catch( Exception ex )
  {
    message = ex.Message;
    return Result.Failed;
  }
}
```

It prompts you to select a wall at a specific point.
The PickObject method can return both the selected wall and the picked point from one single user click.
It also takes a WallFilter instance, an implementation of the ISelectionFilter interface, to restrict the user selection to walls only:
```python
class WallFilter : ISelectionFilter
{
  public bool AllowElement( Element e )
  {
    return e is Wall;
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    return true;
  }
}
```

The selected point is stored on the wall in a newly created extensible storage schema by the following helper method:
```csharp
/// <summary>
/// Create an extensible storage schema,
/// attach it to a wall, populate it with data,
/// and retrieve the data back from the wall.
/// </summary>
void StoreDataInWall(
  Wall wall,
  XYZ dataToStore )
{
  SchemaBuilder schemaBuilder = new SchemaBuilder(
    new Guid( "720080CB-DA99-40DC-9415-E53F280AA1F0" ) );

  // allow anyone to read the object

  schemaBuilder.SetReadAccessLevel(
    AccessLevel.Public );

  // restrict writing to this vendor only

  schemaBuilder.SetWriteAccessLevel(
    AccessLevel.Vendor );

  // required because of restricted write-access

  schemaBuilder.SetVendorId( "ADNP" );

  // create a field to store an XYZ

  FieldBuilder fieldBuilder = schemaBuilder
    .AddSimpleField( "WireSpliceLocation",
    typeof( XYZ ) );

  fieldBuilder.SetUnitType( UnitType.UT\_Length );

  fieldBuilder.SetDocumentation( "A stored "
    + "location value representing a wiring "
    + "splice in a wall." );

  schemaBuilder.SetSchemaName( "WireSpliceLocation" );

  // register the schema

  Schema schema = schemaBuilder.Finish();

  // create an entity (object) for this schema (class)

  Entity entity = new Entity( schema );

  // get the field from the schema

  Field fieldSpliceLocation = schema.GetField(
    "WireSpliceLocation" );

  // set the value for this entity

  entity.Set<XYZ>( fieldSpliceLocation, dataToStore,
    DisplayUnitType.DUT\_METERS );

  // store the entity on the element

  wall.SetEntity( entity );

  // read back the data from the wall

  Entity retrievedEntity = wall.GetEntity( schema );

  XYZ retrievedData = retrievedEntity.Get<XYZ>(
    schema.GetField( "WireSpliceLocation" ),
    DisplayUnitType.DUT\_METERS );
}
```

The mainline also calls another little method ListSchemas which shows how all schemas currently loaded into Revit memory can be accessed and listed:
```csharp
/// <summary>
/// List all schemas in Revit memory across all documents.
/// </summary>
void ListSchemas()
{
  IList<Schema> schemas = Schema.ListSchemas();

  int n = schemas.Count;

  Debug.Print(
    string.Format( "{0} schema{1} defined:",
      n, PluralSuffix( n ) ) );

  foreach( Schema s in schemas )
  {
    IList<Field> fields = s.ListFields();

    n = fields.Count;

    Debug.Print(
      string.Format( "Schema '{0}' has {1} field{2}:",
        s.SchemaName, n, PluralSuffix( n ) ) );

    foreach( Field f in fields )
    {
      Debug.Print(
        string.Format(
          "Field '{0}' has value type {1}"
          + " and unit type {2}", f.FieldName,
          f.ValueType, f.UnitType ) );
    }
  }
}
```

#### Storing a Dictionary Mapping in Extensible Storage

In the sample so far, only a simple XYZ data field is added and stored.

Here is a new question on making use of extensible storage to store a more complex data type, a dictionary mapping keys to values, in this case both represented by strings.

**Question:** I have added simple data to Revit elements using Extensible Storage.
Now I need a schema containing a field which is a map of strings.
I added a field by using
```csharp
FieldBuilder.AddMapField( MyMappedField,
typeof( string ), typeof( string ) );
```

Now comes my problem: how can I set a mapped value to such a field by using Entity.Set<???>(???)?

Similarly, how can I retrieve a mapped value from such a field?

How can I remove a mapped value?

Please can you implement such a sample?

**Answer:** The key, no pun intended, is to use IDictionary<> as the type you pass to the Set<> method, even though your actual implementation concrete type is Dictionary<>.

This is document in the Revit API help file entry on the Entity.Set method, but I admit that it is tricky.

By the way, this is also demonstrated by the ExtensibleStorageManager SDK sample.
For instance, it includes the following code snippet:
```csharp
  // Note that we use IDictionary<> for
  // map types and IList<> for array types

  mySchemaWrapper
    .AddField<IDictionary<string, string>>(
      map0Name, UnitType.UT\_Undefined, null );

  mySchemaWrapper
    .AddField<IList<bool>>(
      array0Name, UnitType.UT\_Undefined, null );
```

Here is a new method StoreStringMapInElement that I added to the sample above to demonstrate this:
```csharp
/// <summary>
/// Create an extensible storage schema specifying
/// a dictionary mapping keys to values, both using
/// strings,  populate it with data, attach it to the
/// given element, and retrieve the data back again.
/// </summary>
void StoreStringMapInElement( Element e )
{
  SchemaBuilder schemaBuilder = new SchemaBuilder(
    new Guid( "F1697E22-9338-4A5C-8317-5B6EE088ECB4" ) );

  // allow anyone to read or write the object

  schemaBuilder.SetReadAccessLevel(
    AccessLevel.Public );

  schemaBuilder.SetWriteAccessLevel(
    AccessLevel.Public );

  // create a field to store a string map

  FieldBuilder fieldBuilder
    = schemaBuilder.AddMapField( "StringMap",
      typeof( string ), typeof( string ) );

  fieldBuilder.SetDocumentation(
    "A string map for Tobias." );

  schemaBuilder.SetSchemaName(
    "TobiasStringMap" );

  // register the schema

  Schema schema = schemaBuilder.Finish();

  // create an entity (object) for this schema (class)

  Entity entity = new Entity( schema );

  // get the field from the schema

  Field field = schema.GetField(
    "StringMap" );

  // set the value for this entity

  IDictionary<string, string> stringMap
    = new Dictionary<string, string>();

  stringMap.Add( "key1", "value1" );
  stringMap.Add( "key2", "value2" );

  entity.Set<IDictionary<string, string>>(
    field, stringMap );

  // store the entity on the element

  e.SetEntity( entity );

  // read back the data from the wall

  Entity retrievedEntity = e.GetEntity( schema );

  IDictionary<string, string> stringMap2
    = retrievedEntity
    .Get<IDictionary<string, string>>(
      schema.GetField( "StringMap" ) );
}
```

Here is the full updated sample application [ExtensibleStorage.zip](zip/ExtensibleStorage.zip) including the Visual Studio solution and full source.

Many thanks to our local Revit extensible storage expert Steven Mycynek for help resolving this issue!