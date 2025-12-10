---
post_number: "0604"
title: "Extensible Storage Features"
slug: "estorage_element_id"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'parameters', 'revit-api', 'transactions', 'walls']
source_file: "0604_estorage_element_id.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0604_estorage_element_id.html"
---

### Extensible Storage Features

Here is a nice hot topic to start off the week.
We already talked about the new Revit 2012 API
EStorage or
[extensible storage](http://thebuildingcoder.typepad.com/blog/2011/04/extensible-storage.html) functionality and presented example code to
[store a map or dictionary](http://thebuildingcoder.typepad.com/blog/2011/05/extensible-storage-of-a-map.html) in it.

Here are a few other interesting notes and issues related to this which have cropped up, contributed and pointed out by Steven Mycynek:

1. [Intended use of EStorage](#1).- [Handling large amounts of data in EStorage](#2).- [EStorage is self-documenting](#3).- [EStorage is object-oriented in two ways](#4).- [Read/Write permissions](#5).- [Modification of EStorage data on an element type](#6).- [Handling of ElementId data in EStorage](#7).- [Retrieving elements with a specific schema entity](#8).- [Checking for a valid entity on an element](#9).- [One entity per element](#10).- [Schemata remain in memory](#11).

The automatic translation of element ids is one of the absolute highlights of this new technology that I was previously not aware of.

#### 1. Intended Use of EStorage

**Question:** When should I use EStorage versus the existing technique of storing my data in text form in an XML-based hidden shared parameter?

**Answer:** EStorage is for storing a lot of complex bits of data that you want to organize into a class-like structure, complete with units, documentation, etc. If you already have an XML file that does all of that already,
and you don't find using a single hidden shared parameter to be a burden, you might want to go ahead and keep using it. On the other hand, if you didn't already have an XML schema in place and had a huge variety of data you
didn't want to convert to a text representation, I'd recommend starting out with EStorage.

#### 2. Handling Large Amounts of Data in EStorage

**Question:** I am thinking of storing a large amount of data on a number of BIM elements.
What approach would you recommend?

**Answer:** The intended and recommended use of EStorage does not include storing huge amounts of data in the model. For instance, we have seen issues with developers trying to store very heavy analysis data in hundreds of MB spread across thousands of elements, all loaded at start-up.
Such usage will degrade performance.
If you store large amounts of data across thousands of elements, do not expect an instant load time.

If you wish to store a large amount of data on individual elements, this should not be a problem. For instance, storing something like a .png file on a wall element is no issue at all, whereas it would be an issue to store a dozen .png files on **every** wall element and extract all of them at once, even if you only need the data for one at a given time.

One of the strengths of EStorage is its handling of arrays of objects or sub-entities and different schemas in general rather than one large segment of data that needs to be read in its entirety and deserialized all at once. I recommend taking advantage of this and only loading what you actually need for a given task. For small loads of data, this might not seem necessary, but when you get into many MB of data, it makes a difference.

#### 3. EStorage is Self-documenting

When you create a schema, you are creating documentation.
Be sure to fill out the documentation strings for each field – they will help you in your development process and others when they use your schema.
What's more, since you can look up a schema by Guid, if you want to share a document with a schema with someone else, all they need is the Guid to look it up and read your structure and documentation comments.
This might be a good opportunity to either use the SchemaWrapperTools included with the ExtensibleStorageManager SDK sample (or a simpler, similar tool) to print out a schema's field definitions and documentation strings from a single GUID input.

#### 4. EStorage is Object-oriented in Two Ways

Not only is a given schema entity structured into named fields, but each entity is placed on elements relative to the data itself, as opposed to one large blob that you must read in its entirety to unpack.
This goes along with what the recommendation above about not reading all storage at start-up.
Since you can choose which elements to process, you have the opportunity to only load what you need an automatically have an association between data and a specific element – shared parameters can't do this.

#### 5. Read/Write Permissions

This is another area that goes beyond shared parameters.
While your schema definition is public, you can restrict who reads and writes schema data based to a specific vendor or a specific application from that vendor.

#### 6. Modification of EStorage Data on an Element Type

**Question:** When transferring types from one project to another using Transfer Project Standards, it only copies across types that are different. If you change a parameter on a wall type that exists in both projects, then it gives you the
option of copying over "new" types or overwriting existing types. However, if the parameters are unchanged but the schema data is different, it treats the wall types as identical, so does not give the option of copying over the
wall type.

Are element types with different EStorage data attached to them treated as the same or not?

**Answer:** Any parameter based change, hidden or not, will trigger a new type to be recognized. EStorage, however, is not a parameter-based transaction, so those rules don't apply.
Therefore, element types with differing EStorage attached to them are still treated as the same in this case.

#### 7. Handling of ElementId Data in EStorage

**Question:** What happens to element ids stored in EStorage?

**Answer:** When you store an ElementId using EStorage and that ElementId gets remapped because the element is deleted or updated, e.g. because of a worksharing update, your stored ElementId is also automatically remapped to the new ElementId value. This is one strong advantage for using EStorage over text or raw numbers to store ElementIds – the tracking for element updates is handled automatically, so you can be sure that your ElementIds will remain valid.
If the element is deleted, your ElementId will be set to ElementId.InvalidElementId.

#### 8. Retrieving Elements with a Specific Schema Entity

**Question:** How can I retrieve all elements that have data from a certain schema attached to them?
Optimally, I think that should be a filtered element collector option.

**Answer:** Right now, you must do a manual iteration of all elements. There may be a project to add a filter as you describe in the future, but we make no promises about any future features whatsoever.

#### 9. Checking for a Valid Entity on an Element

**Question:** If I register a schema and then select an element that has no entity for that schema attached to it, I would expect the following call to return null:
```csharp
Entity ent = e.GetEntity( schema );
```

It does not.
Instead, it returns a valid entity pointing to a null schema.
To handle this, I expanded my check to this:
```csharp
if( null == ent || null == ent.Schema ) ...
```

**Answer:** Use the Entity.IsValid method to check to see if the entity you received from GetEntity actually has data of a given schema.

#### 10. One Entity per Element

**Question:** Is only one entity per schema possible per Revit element?

**Answer:** Yes and no.
There is no way to have more than one entity of a given schema per element via the GetEntity/SetEntity operations.
However, you could always create another schema with more than one sub-entities of a given schema type, including array fields and map fields.

#### 11. Schemata Remain in Memory

**Question:** Is it true that once a schema has been loaded into Revit memory, it never disappears again until the session ends?

**Answer:** That is true.
A schema is available per use on the session level, even if a new document is created.
Entities are associated with specific documents.

Many thanks to Steve for all of these tips!