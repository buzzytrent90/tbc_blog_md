---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.188256'
original_url: https://thebuildingcoder.typepad.com/blog/0567_extensible_storage.html
post_number: '0567'
reading_time_minutes: 4
series: general
slug: extensible_storage
source_file: 0567_extensible_storage.htm
tags:
- csharp
- elements
- levels
- parameters
- revit-api
- transactions
- walls
title: Extensible Storage
word_count: 780
---

### Extensible Storage

I mentioned the main new features of the
[Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/03/revit-2012-api-features.html) and
also pointed out that many important developer
[wishes have been resolved](http://thebuildingcoder.typepad.com/blog/2011/03/many-issues-resolved.html).
Here is another very fundamental and longstanding wish that has been addressed, more completely than anyone might have expected:

**Question:** How can I store my custom application data inside the Revit file using the Revit API?

**Answer:** One possibility is to use a shared parameter. This is a slightly convoluted operation and involves setting up the shared parameter text file appropriately. You have the option of whether a shared parameter should or should not be visible to the user. The whole procedure is demonstrated by the FireRating SDK sample and discussed in numerous blog posts:

- [Creating and using a shared parameter on various elements](http://thebuildingcoder.typepad.com/blog/2009/06/model-group-shared-parameter.html).- [Storing structured data in a Revit shared parameter](http://thebuildingcoder.typepad.com/blog/2009/07/store-structured-data.html).- [Storing project data](http://thebuildingcoder.typepad.com/blog/2009/07/store-project-data.html).

In Revit 2012, one of the major API enhancements is the Extensible Storage mechanism. This offers a completely new storage facility for applications which is completely invisible to the user. It is the very first item described in the What's New section of the documentation on major enhancements:

#### Extensible Storage

The Revit API now allows you to create your own class-like Schema data structures and attach instances of them to any Element in a Revit model. This functionality can be used to replace the technique of storing data in hidden shared parameters. Schema-based data is saved with the Revit model and allows for higher-level, metadata-enhanced, object-oriented data structures. Schema data can be configured to be readable and/or writable to all users, just a specific application vendor, or just a specific application from a vendor. The extensible storage classes are all found in the Autodesk.Revit.DB.ExtensibleStorage namespace:

- Schema – contains a unique schema identifier, read/write permissions, and a collection of data Field objects.- Entity – an object containing data corresponding to a Schema that can be inserted into a Revit Element.- Field – contains data name, type, and unit information and is used as the key to access corresponding data in an Entity.- SchemaBuilder – create Schema definitions.- FieldBuilder – a helper class used with SchemaBuilder when creating a new field.

The following data types are currently supported:

- int- short- double- float- bool- string- Guid- ElementId- Autodesk.Revit.DB.UV- Autodesk.Revit.DB.XYZ- Array (as a System.Collections.Generic.IList<T>)- Map (as a System.Collections.Generic.IDictionary<TKey, TValue> – all types are supported for keys except double, float, XYZ, and UV)- Autodesk.Revit.DB.ExtensibleStorage.Entity (an instance of another Schema, also known as a SubSchema)

Simple usage of ExtensibleStorage:
```csharp
/// <summary>
/// Create a data structure, attach it to a wall,
/// populate it with data, and retrieve the data
/// back from the wall
/// </summary>
public void StoreDataInWall(
  Wall wall,
  XYZ dataToStore )
{
  Transaction createSchemaAndStoreData
    = new Transaction( wall.Document, "tCreateAndStore" );

  createSchemaAndStoreData.Start();
  SchemaBuilder schemaBuilder = new SchemaBuilder(
    new Guid( "720080CB-DA99-40DC-9415-E53F280AA1F0" ) );

  // allow anyone to read the object
  schemaBuilder.SetReadAccessLevel(
    AccessLevel.Public );

  // restrict writing to this vendor only
  schemaBuilder.SetWriteAccessLevel(
    AccessLevel.Vendor );

  // required because of restricted write-access
  schemaBuilder.SetVendorId( "ADSK" );

  // create a field to store an XYZ
  FieldBuilder fieldBuilder = schemaBuilder
    .AddSimpleField( "WireSpliceLocation",
    typeof( XYZ ) );

  fieldBuilder.SetUnitType( UnitType.UT\_Length );

  fieldBuilder.SetDocumentation( "A stored "
    + "location value representing a wiring "
    + "splice in a wall." );

  schemaBuilder.SetSchemaName( "WireSpliceLocation" );

  Schema schema = schemaBuilder.Finish(); // register the Schema object

  // create an entity (object) for this schema (class)
  Entity entity = new Entity( schema );

  // get the field from the schema
  Field fieldSpliceLocation = schema.GetField(
    "WireSpliceLocation" );

  entity.Set<XYZ>( fieldSpliceLocation, dataToStore,
    DisplayUnitType.DUT\_METERS ); // set the value for this entity

  wall.SetEntity( entity ); // store the entity in the element

  // get the data back from the wall
  Entity retrievedEntity = wall.GetEntity( schema );

  XYZ retrievedData = retrievedEntity.Get<XYZ>(
    schema.GetField( "WireSpliceLocation" ),
    DisplayUnitType.DUT\_METERS );

  createSchemaAndStoreData.Commit();
}
```

The Revit 2012 SDK includes a much more full-fledged example of making use of this technology, the ExtensibleStorageManager sample.

As Arnošt Löbel kindly pointed out, the VendorId specified above is the same as the
[new required VendorId tag in the add-in manifest](http://thebuildingcoder.typepad.com/blog/2011/03/revit-2012-api-features.html).
ADSK is a vendor id used by Autodesk, and you should replace it by your own
[Autodesk Registered Developer Symbol RDS](http://www.autodesk.com/symbreg).

That should give you ample material to explore. Have fun!