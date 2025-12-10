---
post_number: "0950"
title: "Effortless Extensible Storage"
slug: "vc_estore_extension"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'references', 'revit-api', 'selection', 'transactions', 'views']
source_file: "0950_vc_estore_extension.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0950_vc_estore_extension.html"
---

### Effortless Extensible Storage

Today we proudly present an exciting and powerful extensible storage utility class library by
[Victor Chekalin](http://www.facebook.com/profile.php?id=100003616852588).
I'll also add some other assorted items at the end:

- [Extensible storage extension](#2)
- [Example 1 – save integer value to an element](#3)
- [Example 2 – map field and sub-entity value](#4)
- [Extensible storage extension conclusion](#5)
- [Zoom and pan in Revit 2013 preview control](#6)
- [Display real-time interactive element properties](#7)

#### Extensible Storage Extension

I think everyone of us has used shared parameters to save custom data in a Revit document.
If you created shared parameters programmatically, you know it was not easy.
Unfortunately, it was the only way to store own data in a Revit document.

When Revit 2012 was released I was very happy that Autodesk made one more way to store auxiliary data in a document.
It was a really exciting enhancement.

But when I started to use it in the real projects I noticed some inconveniences when using ExStorage in code.
I need to create a lot of code to create a schema, need to remember entity field names and their types, always check if schema or entity or field is null etc.
It was especially difficult to use complex schemas, with Sub-Schemas and with Array or Map fields.
It is even harder when using Sub-Schema in the Map field.
For example, here is the sample code snippet from a real project:

```csharp
  Family family = ( elementType as FamilySymbol )
    .Family;

  var familyEntity = family.GetEntity(
    \_familySchema );

  if( !familyEntity.IsValid() )
  {
    familyEntity = new Entity( \_familySchema );
  }

  var fsField = \_familySchema.GetField(
    "FamilySymbolsInfo" );

  var familySymbolsData =
    familyEntity.Get<IDictionary<string, Entity>>(
      fsField );

  Entity elementTypeEntity;

  if( !familySymbolsData.TryGetValue(
    elementType.Name, out elementTypeEntity ) )
  {
    elementTypeEntity = new Entity(
      fsField.SubSchemaGUID );
  }

  elementTypeEntity.Set( "Entity",
    serializedString );

  if( !familySymbolsData.ContainsKey(
    elementType.Name ) )
  {
    familySymbolsData.Add( elementType.Name,
      elementTypeEntity );
  }

  familyEntity.Set( fsField, familySymbolsData );

  family.SetEntity( familyEntity );
```

Pretty simple, isn't it? :-)

Of course not...

Very easy to make a mistake and quite difficult to understand what is written there.
The more I used ExStorage, the more I wanted to make it easier to use.

The Revit API help states the following about Extensible Storage: '... allows you to create your own class-like Schema data structures and attach instances of them...'
I thought – 'Why it is class-like and instances?
Why can't I use exactly a class to define Schema and use instance of a class to attach it to the Revit element?'

Thus was born the idea of the Extensible Storage Extension.
The idea is simple: define a special class describing the ExStorage schema, where class properties are schema fields.
To attach data to an element, create an instance of the class, set it properties and set to the element.
Sounds good.

As the result I implemented all what I wanted to do in my extension.

The extension supports all extensible storage features such as:

- Simple field
- Array field
- Map field
- Sub entities
- Documentations
- VendorID
- DisplayUnit

Let's look at the examples and compare it with the default using Extensible Storage.

#### Example 1 – Save Integer Value to an Element

To save an integer value to an element using standard Extensible storage methods, we can use the following code:

```csharp
  // 1. Looking for the schema in the memory

  Schema schema = Schema.Lookup( new Guid(
    "4E5B6F62-B8B3-4A2F-9B06-DDD953D4D4BC" ) );

  // 2. Check if schema exists in the memory or not

  if( schema == null )
  {
    // 3. Create it, if not

    schema = CreateSchema();
  }

  // 4. Create entity of the specific schema

  var entity = new Entity( schema );

  // 5. Set the value for the Field.
  // HERE WE HAVE TO REMEMEBER THE
  // NAME OF THE SCHEMA FIELD
  // It would be better to check if the field with
  // such name exists in the schema

  entity.Set( "SomeValue", 888 );

  // 6. Attach entity to the element

  element.SetEntity( entity );

  private Schema CreateSchema()
  {
    SchemaBuilder schemaBuilder =
      new SchemaBuilder( new Guid(
        "4E5B6F62-B8B3-4A2F-9B06-DDD953D4D4BC" ) );

    schemaBuilder.SetSchemaName( "SimpleIntSchema" );

    // Have to define the field name as string and
    // set the type using typeof method
    schemaBuilder.AddSimpleField( "SomeValue",
      typeof( int ) );

    return schemaBuilder.Finish();
  }
```

Here we achieve the same using the Extensible Storage Extension:

Define the special class at first:

```csharp
  // Set schema Id and Schema name as class attributes

  [Schema( "4E5B6F62-B8B3-4A2F-9B06-DDD953D4D4BB",
    "SimpleIntSchema" )]

  public class IntEntity : IRevitEntity
  {
    // Mark the property as Schema field using attributes

    [Field]
    public int SomeValue { get; set; }
  }
```

Attach an instance of this class to the element:

```csharp
  // Create a new instance of the class

  IntEntity intEntity =
      new IntEntity();

  // Set property value

  intEntity.SomeValue = 777;

  // Attach to the element

  element.SetEntity( intEntity );
```

That's it.

Amazing, isn't it?

Just compare the two approaches.

What is easier – to work with the ordinary .NET classes, or use a lot of methods to work with Schemas and Entities?

I think the answer is obvious.

Here is a similar comparison of the code to read a value.

Default approach:

```csharp
  var schema = Schema.Lookup( new Guid(
    "4E5B6F62-B8B3-4A2F-9B06-DDD953D4D4BC" ) );

  if( schema != null )
  {
    var entity = element.GetEntity( schema2 );
    var someValue = entity.Get<int>( "SomeValue" );
    TaskDialog.Show( "Entity value",
      someValue.ToString() );
  }
```

My extensible storage extension approach:

```csharp
  var intEntity2 = element.GetEntity<IntEntity>();

  if( intEntity2 != null )
  {
    TaskDialog.Show( "Entity value",
      intEntity2.SomeValue.ToString() );
  }
```

Let's look at another more complicated example.

#### Example 2 – Map Field and Sub-Entity Value

Here is some code to set up a complex entity using the default ExStorage methods:

```csharp
  // Check if schema exists in the memory.

  var schema3 = Schema.Lookup( new Guid(
    "1899FD3C-7046-4B53-945A-AA1370B8C577" ) );

  if( schema3 == null )
  {
    // Create if not

    schema3 = CreateComplexSchema();
  }

  var entity4 = new Entity( schema3 );

  // Map fields
  IDictionary<int, Entity> mapOfEntities
    = new Dictionary<int, Entity>();

  // Create sub-entity 1
  var entity7 = new Entity( new Guid(
    "4E5B6F62-B8B3-4A2F-9B06-DDD953D4D4BC" ) );

  entity7.Set( "SomeValue", 7 );

  // create sub-entity 2
  var entity8 = new Entity( new Guid(
    "4E5B6F62-B8B3-4A2F-9B06-DDD953D4D4BC" ) );

  entity8.Set( "SomeValue", 8 );

  mapOfEntities.Add( 7, entity7 );
  mapOfEntities.Add( 8, entity8 );

  entity4.Set( "MapField", mapOfEntities );

  element.SetEntity( entity4 );

  private Schema CreateComplexSchema()
  {
    SchemaBuilder schemaBuilder = new SchemaBuilder(
      new Guid( "1899FD3C-7046-4B53-945A-AA1370B8C577" ) );

    schemaBuilder.SetSchemaName( "ComplexSchema" );

    var mapField = schemaBuilder.AddMapField(
      "MapField", typeof( int ), typeof( Entity ) );

    mapField.SetSubSchemaGUID( new Guid(
      "4E5B6F62-B8B3-4A2F-9B06-DDD953D4D4BC" ) );

    mapField.SetDocumentation(
      "Map field documentation" );

    return schemaBuilder.Finish();
  }
```

We can use the following to change a value in the map field:

```csharp
  var entity10 = element.GetEntity( schema3 );

  var mapField =
    entity10.Get<IDictionary<int, Entity>>(
      "MapField" );

  if( mapField != null )
  {
    if( mapField.ContainsKey( 8 ) )
    {
      var entity11 = mapField[8];
      entity11.Set( "SomeValue", 999 );

      // Write changes

      entity10.Set( "MapField", mapField );

      element.SetEntity( entity10 );
    }
  }
```

Here is the code to achieve the same result using the extension:

```csharp
  [Schema( "93CC69FC-AB6F-451A-B075-A5D9467569C2",
    "ComplexSchema" )]
  public class ComplexEntity : IRevitEntity
  {
    [Field]
    public string SimpleField { get; set; }

    [Field]
    public List<IntEntity> ArrayField { get; set; }

    [Field]
    public Dictionary<int, IntEntity> MapField { get; set; }
  }

  ComplexEntity complexEntity = new ComplexEntity();

  complexEntity.MapField
    = new Dictionary<int, IntEntity>
    {
      {9, new IntEntity(){SomeValue = 9}},
      {10, new IntEntity(){SomeValue = 10}}
    };

  element.SetEntity( complexEntity );
```

Change the value like this:

```csharp
  var complexEntity2 = element
    .GetEntity<ComplexEntity>();

  if( complexEntity2 != null )
  {
    if( complexEntity.MapField.ContainsKey( 9 ) )
    {
      var entityInMapField = complexEntity.MapField[9];

      entityInMapField.SomeValue = 9898;

      element.SetEntity( complexEntity2 );
    }
  }
```

#### Extensible Storage Extension Conclusion

My extensible storage extension provides an easy and safe way to use extensible storage.
You use standard .NET classes instead a lot of RevitAPI classes and methods.

If you use extensible storage a lot, you should evaluate the benefits of my extension.
If you never used ExStorage, I hope that my extension will help you to begin using it.

You may download the source code and more samples from the
[GitHub repository](https://github.com/chekalin-v/VCExtensibleStorageExtension).
If you do not know how to use Git you can simply download it as a complete
[zip archive](https://github.com/chekalin-v/VCExtensibleStorageExtension/archive/master.zip).

If you do not need the source code, you may download the
[.NET assembly](https://raw.github.com/chekalin-v/VCExtensibleStorageExtension/master/bin/Release/VCExtensibleStorageExtension.dll) only,
reference it in your add-in project and immediately take advantage of all the benefits of my extension.

I hope this information is useful for all Revit developers.

Many thanks to Victor for this beautiful and powerful tool!

#### Zoom and Pan in Revit 2013 Preview Control

I myself ran into some confusion using the preview control in Revit 2013.
Others ran into similar issues, so here is a clarification:

**Question:** I notice varying behaviour of the preview control in different versions of Revit 2013.
If you compile and load the PreviewControl sample add-in from the Revit 2013
[UIAPI SDK sample](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4),
you can pan around in the model using the mouse wheel or the context menu.
This no longer works in the service packs.
How should we deal with this, please?

**Answer:** Yes, I noticed something similar myself when implementing my
minimal DevCamp
[PreviewControlSimple](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#23) sample
for the session on
[Revit 2013 UI API enhancements](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#2).

I would suggest trying to encapsulate the whole use of the preview control in a transaction.

The zoom and pan is actually affecting the real live Revit viewport, i.e., modifying the database, so it requires a transaction.

I believe that you can get all three kinds of behaviour by panning and zooming in a preview control:

- No transaction → exception on touching the view cube
- No transaction → nothing happens (like you describe)
- Transaction → should work afaik

Therefore, the simple solution is to simply encapsulate your use of the control in a transaction.

If you commit it afterwards, the Revit viewport will be affected by the panning and zooming. If not, not.

I'm happy to announce that this should no longer be an issue in Revit 2014.

#### Display Real-time Interactive Element Properties

**Question:** I have implemented a tool that displays some properties of a selected element on the add-in button click.

I would like to enable the user to just click my add-in button once, go on selecting elements, and display updated properties on each new selection.

How could that be achieved?

**Answer:** To achieve this, you have to work in a modeless context and make use of the
[Idling event](http://thebuildingcoder.typepad.com/blog/idling).

Here is an obsolete sample describing a
[modeless pressure drop tool](http://thebuildingcoder.typepad.com/blog/2009/10/modeless-pressure-drop-tool.html)
posted before the advent of Idling.
That might give you some bright ideas, but definitely do not use it as is!

Here is a more up-to-date sample making use of the Idling event to display the contents of the current selection set, which is pretty close to what you are searching for.

I strongly suggest basing anything making use of the Idling event, or an external event, which is a simplified wrapper around that, on the
[ModelessForm\_ExternalEvent and ModelessForm\_IdlingEvent](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#1) SDK
samples introduced in Revit 2013.