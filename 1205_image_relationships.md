---
post_number: "1205"
title: "Debugging and Maintaining the Image Relationship"
slug: "image_relationships"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'revit-api', 'rooms', 'sheets', 'transactions', 'views', 'walls']
source_file: "1205_image_relationships.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1205_image_relationships.html"
---

### Debugging and Maintaining the Image Relationship

The Revit database contains a number of
[undocumented relationships](http://thebuildingcoder.typepad.com/blog/2011/11/undocumented-elementid-relationships.html),
and it can be challenging and useful to discover them.

Christian Tonn of [kubit](http://www.kubit-software.com) presents a powerful method to determine and maintain such a relationship using an officially supported approach instead:

**Question:** Is it possible to get the ImageType object from an image you inserted in the Revit document with Document.Import?

**Answer:** Not directly!

I am using the Import method:

```csharp
  bool Import(
    string file,
    ImageImportOptions options,
    View view,
    out Element element);
```

It only returns an Element object, which has no parameters attached – I used RevitLookup to check – nor can it be cast to anything useful, e.g. ImageType.

However, I can query Revit for all ImageType classes, and it really is possible to read the right Bitmap object out of the appropriate instance.

But how to connect the element returned by the Import method with the appropriate ImageType instance?

The Element holds the image position in the view and the ImageType holds the Bitmap without position.

I need both.

Luckily, both items have the same Element.Name in the database, the image filename.

I implemented a solution to determine and maintain the missing relationship like this:

1. Start a TransactionGroup.
2. Start a first transaction.
3. Import the image with a unique filename:
```csharp
String tmpFilename = System.IO.Path.GetTempPath()
  + "kubit\_" + Guid.NewGuid().ToString() + ".png";

mDoc.Import( tmpFilename, options, newView,
  out element );
```4. Attach an extensible storage Entity to the returned image element:
```csharp
Entity additionalData = newEntity(
  MyImageSchema.GetSchema() );

element.SetEntity( additionalData );
```5. Commit the first transaction.
6. Search the Revit database for all ImageType objects.
7. Use them to populate a dictionary mapping the Name property value to the ImageType instance: Dictionary<string, ImageType> nameImageTypeMap.
8. Retrieve all Element objects returned by the Import method in step 3 using an ExtensibleStorageFilter.
9. Start a second transaction.
10. Loop through all the elements and add the ImageType information to them using the dictionary:
```csharp
Entity additionalData = elem.GetEntity(
  MyImageSchema.GetSchema() );

if( additionalData.IsValid() )
{
  // Find correspondence between ImageType and
  // Element (first has image, second has additionalData)

  if( !nameImageTypeMap.ContainsKey( elem.Name ) )
    continue;

  ImageType image = nameImageTypeMap[elem.Name];
  additionalData.Set<ElementId>( "ImageTypeId", image.Id );
  elem.SetEntity( additionalData );
}
```11. Commit the second transaction.
12. Commit the TransactionGroup.

After this you can easily retrieve the image elements using an ExtensibleStorageFilter and also determine their associated ImageType instanced using the element id stored in ImageTypeId.

I am so happy I could figure this out.

I really hope this will be improved in some future release of the Revit API.

#### Element Lister Description and Questions

Very many thanks to Christian for his research and solution!

An easy way to discover certain element relationships between related objects that are added to Revit by certain operations is provided by the element lister:

1. List all elements to a file A.
2. Perform an operation in Revit.
3. List all elements to a file B.
4. Determine the difference between A and B.

That gives me the newly added or deleted elements and their ids.

I used it any number of times in the past to explore:

- Stacked walls
- Constraints
- Dimensioning
- Revision clouds
- [Element parameters](http://thebuildingcoder.typepad.com/blog/2008/11/exploring-element-parameters.html)
- [Title block of sheet](http://thebuildingcoder.typepad.com/blog/2009/11/title-block-of-sheet.html)
- [Pre-, post- and pick select](http://thebuildingcoder.typepad.com/blog/2010/05/pre-post-and-pick-select.html)
- [Curtain wall geometry](http://thebuildingcoder.typepad.com/blog/2010/05/curtain-wall-geometry.html)
- [Removing DWF links](http://thebuildingcoder.typepad.com/blog/2012/03/remove-dwf-links.html)
- [Purging zero-area rooms and spaces](http://thebuildingcoder.typepad.com/blog/2013/07/sydney-revit-api-training-and-vacation.html#5)

Possibly something similar could be implemented using a more API oriented approach by subscribing to the DocumentChanged event, which returns the same information in the added and deleted element id collections.

The element lister and other basic tools were used to establish certain
[undocumented element id relationships](http://thebuildingcoder.typepad.com/blog/2011/11/undocumented-elementid-relationships.html).

Use of these element id relationships is
[totally unsupported](http://thebuildingcoder.typepad.com/blog/about-the-author.html#4),
of course, and they can obviously change at any time.

Your approach sounds much more robust.

How does yours compare with these?

Do you see any similarity at all?

Are the element ids of the related elements also related?

#### Confirmation

Yes, the ElementId of the ImageType is one less than the ElementId of the returned image Element from doc.Import, so they should obviously belong together.

I was not sure about this, however, and chose the Element.Name approach instead.

It is kind of similar, and I agree that it ought to be more robust.

#### Conclusion

The relationship discussed here between an image and the associated image type exists analogously for other pairs of Revit elements as well, so the approach described above is probably also useful for other similar situations.

#### Welcome Back, Jaime!

Six weeks ago, we
[welcomed our new colleague Jaime](http://thebuildingcoder.typepad.com/blog/2014/07/upgrading-family-files-silently-part-2.html#4) and
enjoyed his first blog post.
Immediately afterwards, he went off for a long holiday and begin a much longer marriage.
Now he returned and immediately answered a
[Revit API forum issue](http://forums.autodesk.com/t5/revit-api/get-value-from-app-config-file/m-p/5244679) with
a new blog post on
[reading a value from an app.config file](http://adndevblog.typepad.com/aec/2014/09/get-value-from-appconfig-file-and-sur-la-france.html) and
the good times he had while away.

Welcome back, Jaime!