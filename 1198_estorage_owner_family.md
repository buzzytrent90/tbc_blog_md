---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.6
content_type: code_example
optimization_date: '2025-12-11T11:44:15.492971'
original_url: https://thebuildingcoder.typepad.com/blog/1198_estorage_owner_family.html
post_number: '1198'
reading_time_minutes: 7
series: family
slug: estorage_owner_family
source_file: 1198_estorage_owner_family.htm
tags:
- csharp
- elements
- family
- parameters
- python
- revit-api
- transactions
- views
title: Accessing Extensible Storage on OwnerFamily in Project
word_count: 1423
---

### Accessing Extensible Storage on OwnerFamily in Project

A couple of developers reported a problem accessing extensible storage data on families when they are loaded into a project in Revit 2015.
This problem did not occur in previous versions.

Luckily, a workaround is possible right now, designed and implemented by Alexander Ignatovich, Александр Игнатович, of
[Investicionnaya Venchurnaya Companiya](http://www.iv-com.ru).

Alexander already made a number of other complex and powerful contributions to The Building Coder in the past:

- [Exporting Image and Setting a Default 3D View Orientation](http://thebuildingcoder.typepad.com/blog/2013/08/setting-a-default-3d-view-orientation.html)
- [Intimate Revit Database Exploration with the Python Shell](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html)
- [Multi-Version Visual Studio Revit Add-In Wizard](http://thebuildingcoder.typepad.com/blog/2013/11/multi-version-visual-studio-revit-add-in-wizard.html)
- [Another Balloon Tip Implementation](http://thebuildingcoder.typepad.com/blog/2014/03/another-balloon-tip-implementation.html)
- [Category Analysis with and without Python](http://thebuildingcoder.typepad.com/blog/2014/03/category-analysis-with-and-without-python.html)
- [WPF Fill Pattern Viewer Control](http://thebuildingcoder.typepad.com/blog/2014/04/wpf-fill-pattern-viewer-control.html)
- [Getting Serious Adding New Materials from List](http://thebuildingcoder.typepad.com/blog/2014/04/getting-serious-adding-new-materials-from-list.html)

Here is Alexander's description of the problem and workaround:

#### Revit 2015 Extensible Storage Access on Document OwnerFamily

The problem occurs when accessing extensible storage data on a family from a project environment via the Document.OwnerFamily property.

Let's start off with the source code referred to below: here is
[EstorageOnOwnerFamilyWorkaround.zip](zip/EstorageOnOwnerFamilyWorkaround.zip) containing
the complete source code, Visual Studio solution and add-in manifest of the projects discussed below.

I faced a big problem migrating my add-ins working with extensible storage tied to families to Revit 2015: extensible storage on an OwnerFamily object cannot be accessed from the project family it is loaded into. Vice versa, the same thing: extensible storage created in a project, cannot be read through doc.OwnerFamily in the family editor environment.

Let's take a closer look at this problem.
First of all, let's create a SchemaHelper – a piece of code to simplify our next investigations.
We define a simple extensible storage schema that contains one single field Id of type Guid.
See the project named "Common", SchemaHelper.cs, in the attached sample source code.

The method SchemaHelper.CreateSchema looks up or creates our extensible storage schema:

```csharp
  private static Schema CreateSchema()
  {
    var buider = new SchemaBuilder( GetSchemaId() );
    buider.SetSchemaName( "FamilyTestSchema" );
    buider.AddSimpleField( "Id", typeof( Guid ) );
    return buider.Finish();
  }
```

Now, let's implement a command that writes data to the extensible storage.
Please refer to the ExtStoragesFirstStep project.
It adds a new simple "Create test extensible storage data" command.
It writes data to the doc.OwnerFamily if the current document is a family, otherwise it writes data to all project families:

```csharp
  if( doc.IsFamilyDocument )
    SetEntityToOwnerFamily( doc );
  else
    SetEntityToDocumentFamilies( doc );
```

So let us see how this works and what is the source of problem: open any family in the family editor, invoke the command "Create test extensible storage data", ensure, that extensible storage entity exists in the family (for example, using RevitLookup, or just write another simple command), load the family to any other document (Project or family) and look at the loaded family in RevitLookup.
You will see no "FamilyTestSchema" extensible storage entities inside the family.

Now, invoke this command in any project document, look at the families in RevitLookup.
None of the families will contain "FamilyTestSchema" extensible storage entities.
Open any family from this project in the family editor.
There will be no entity of our schema on the document.OwnerFamily object.

The same code for Revit 2014 works fine; this is a regression in Revit 2015.
Another problem concerns getting extensible storage entity from family in dynamic updater; we also cannot read extensible storage data in families.
I suppose that when the transaction is "almost done" in the dynamic updater, Revit tries to read data from the family in the document in a similar way.

#### Workaround

I found a partial workaround for this problem.
In our add-ins, the most important case is when we set extensible storage entity in the family editor through the doc.Owner family and later retrieve this data in the Revit project.

The idea is to use the new FamilyLoadingIntoDocument and FamilyLoadedIntoDocument events provided by the ControlledApplication class.
I have investigated that the family being loaded always exists already in the application.Documents collection.
In the FamilyLoadingIntoDocument event we can retrieve the FamilyPath and FamilyName property values.
In the FamilyLoadedIntoDocument event handler, we are permitted to make modifications to the document.
So let me show you the code – for the full version, please refer to the module MoveExtensibleStorageDataApplication.cs in the ExtStorageSecondStep project in the attached source code:

```csharp
  private static string GetFamilyFileName(
    FamilyLoadingIntoDocumentEventArgs e )
  {
    return string.Format( "{0}.rfa", e.FamilyName );
  }

  public static Document GetFamilyDocument(
    FamilyLoadingIntoDocumentEventArgs e )
  {
    var document = e.Document;
    var application = document.Application;

    var documents = application.Documents
      .Cast<Document>();

    string familyname = GetFamilyFileName( e );

    // If e.FamilyPath is not empty it is simple:
    // just retrieve from the documents collection
    // by document PathName;
    // otherwise, get the document by title
    // (it can be with or without \*.rfa).

    return string.IsNullOrWhiteSpace( e.FamilyPath )

      ? documents.FirstOrDefault(
        x =>
          string.IsNullOrWhiteSpace( x.PathName )
          && ( x.Title == e.FamilyName
            || x.Title == familyname ) )

      : documents.FirstOrDefault(
        x =>
          x.PathName == Path.Combine(
            e.FamilyPath, familyname ) );
  }

  private void OnFamilyLoadingIntoDocument(
    object sender,
    FamilyLoadingIntoDocumentEventArgs e )
  {
    var familyDocument = GetFamilyDocument( e );
    var family = familyDocument.OwnerFamily;

    var schema = SchemaHelper.GetFamilySchema();
    var entity = family.GetEntity( schema );

    // If the entity is valid, store it in
    // the idToMove class property.

    idToMove = entity.IsValid()
      ? entity.Get<Guid>( "Id" )
      : (Guid?) null;
  }
```

So, if the family has extensible storage, we will read its Id from the entity and save it to the idToMove field.
Now let us write it back to the document:

```csharp
  private void OnFamilyLoadedIntoDocument(
    object sender,
    FamilyLoadedIntoDocumentEventArgs e )
  {
    // The event can be cancelled or loading family
    // can be without extensible storage entity.

    if( e.IsCancelled()
      || e.Status != RevitAPIEventStatus.Succeeded
      || !idToMove.HasValue )
    {
      return;
    }

    using( var transaction = new SubTransaction( e.Document ) )
    {
      transaction.Start();

      var familyId =
        e.NewFamilyId == ElementId.InvalidElementId
          ? e.OriginalFamilyId
          : e.NewFamilyId;

      var family = e.Document.GetElement( familyId );
      var schema = SchemaHelper.GetFamilySchema();
      var entity = new Entity( schema );
      entity.Set( "Id", idToMove.Value );
      family.SetEntity( entity );
      transaction.Commit();
    }
  }
```

To test this, you can manually copy the add-in manifest ExtStorageSecondStep.addin to the Revit Addins folder and restart Revit.
Repeat the load family sequence with our extensible storage entity to the document.
Now it has our entity.
We do not need to check nested families; they work fine.

In summary: the new family loading and loaded events allow you to retrieve the data from the family and write it back to the project; you may also need to rewrite your dynamic updaters that work with families – for example, in some cases, you can call e.Cancel in the FamilyLoadingIntoDocument event to prohibit a family from being loaded into the document.

Another case (when you set some data to project families) is more complex, and has no solution in the general case, but in some cases the algorithm can be adapted.
One of my add-in goals was to permit family loading to some projects, but excluding families that were already contained in the project.

The idea of the code was to calculate some "key" entity for the project – e.g., the central model path, or local path if the file is not workshared, and a couple of project information parameters – and write it to all families in the document.
In the dynamic updater, check if the key derived from the family is equal to the key derived from document.
The new approach is to get family and all nested shared families from the document in the FamilyLoadedIntoDocument event.
If they all exist, allow this action, otherwise cancel it.
Of course, it is not very good permanent solution, but it is an acceptable temporary workaround.
In this document, you can only load a family with a name that already exists; no new family can be loaded, unless it is nested.

Many thanks to Alexander for his in-depth research and crucial and creative stopgap workaround.

By the way, it also highlights the importance and usefulness of the new family loading events.

We are obviously working on an internal fix to remove the need for this as soon as possible.