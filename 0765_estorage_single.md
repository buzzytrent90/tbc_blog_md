---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.2
content_type: code_example
optimization_date: '2025-12-11T11:44:14.547724'
original_url: https://thebuildingcoder.typepad.com/blog/0765_estorage_single.html
post_number: '0765'
reading_time_minutes: 10
series: general
slug: estorage_single
source_file: 0765_estorage_single.htm
tags:
- csharp
- elements
- filtering
- levels
- parameters
- python
- revit-api
- transactions
title: DevBlog, DevCamp, Element and Project Wide Data
word_count: 1994
---

﻿

### DevBlog, DevCamp, Element and Project Wide Data

Every once in a while, I like to start the week with a bang, so here goes, with a risk of information overload on the following topics:

- [ADN AEC DevBlog](#1)- [Autodesk AEC DevCamp](#2)- [Goodbye Kailash Kute](#3)- [Project Wide Data Storage](#4)- [DataStorage Element](#5)
          - [Simple DataStorage Sample](#6)- [Identifying DataStorage Elements](#7)

#### ADN AEC DevBlog

I bet you are not half as confused as we always are internally about DevDays, DevLabs, DevNotes, and last but not least DevCamps.

I
[recently mentioned](http://thebuildingcoder.typepad.com/blog/2012/04/failure-rollback.html) that
we are raising the bar one further with the introduction of the Autodesk Developer Network
[ADN](http://www.autodesk.com/joinadn) DevBlogs,
for AutoCAD and Infrastructure, marking the advent of a whole series and a significant shift from material available to ADN members, like technical solutions known as DevNotes on the ADN extranet, to material publicly available to everyone.
Following these two, we are now proud to present the AEC DevBlog!

![AEC DevBlog](img/aec_devblog_banner.png)

The AEC DevBlog covers Revit, Navisworks and other Autodesk AEC and BIM technologies and their APIs.

Here are links to the three DevBlogs live so far:

- [AutoCAD DevBlog](http://adndevblog.typepad.com/autocad)- [Infrastructure Modelling DevBlog](http://adndevblog.typepad.com/infrastructure)- [AEC DevBlog](http://adndevblog.typepad.com/AEC)

ADN DevBlogs for Manufacturing, Media & Entertainment are also expected.

For more information, please refer to
[Stephen Preston's welcome post](http://adndevblog.typepad.com/autocad/2012/02/welcome.html).

#### Only a month to go – Autodesk AEC DevCamp

I also
[recently mentioned](http://thebuildingcoder.typepad.com/blog/2012/04/devcamp-and-refresh-display-for-a-kinetic-facade.html#1) the
upcoming
[AEC DevCamp conference](http://tinyurl.com/83jzvov) being
held in Boston June 6-7, where customers, resellers and professional developers come together and learn about extending Autodesk technologies through software development.

The AEC DevCamp comprises several parallel tracks for

- Revit App Development for Beginners and Intermediates- Revit App Development for Intermediates and Experts- Infrastructure Technologies- Cloud/Mobile Technologies- Other Autodesk AEC Technologies- Business

The classes are presented by a managers, core developers, ADN experts and others and cover all levels from people just beginning to do software development with our technologies to experts looking to build sophisticated cloud and mobile apps.
You can also grab a chance to meet and talk with these interesting and influential people.

Besides welcoming new beginners to the Revit API, I would also like to bid farewell to an old-timer:

#### Goodbye Kailash Kute

I gratefully acknowledge this very sweet parting message from one of our most prolific commenters and autonomous Revit MPE API problem solvers,
[Kailash Kute](http://thebuildingcoder.typepad.com/blog/2011/02/create-a-pipe-cap.html?cid=6a00e553e1689788330168eb7ca24d970c#comment-6a00e553e1689788330168eb7ca24d970c):

Dear Jeremy, i would like to inform you that since i have changed my current company to another where there is no Revit, but till this whole time which i used to blog post on building coder, was very good and fantastic journey, your blog was most useful stuff for me, i liked the coding, the posts, the information provided on this.
i will miss but i also like to appreciate the work you are doing for revit stuff.
Really it was a wonderful journey till this day on BuildingCoder.

Very many thanks, Kailash, for your numerous contributions and great endurance in solving so many Revit MEP API issues!

I was always sorry not to be able to help you more, and very impressed that you struggled on and succeeded in solving every single issue in the end.

Funnily enough, I just last week published an
[updated pipe cap creation](http://thebuildingcoder.typepad.com/blog/2012/05/create-a-pipe-cap.html) post
superseding the
[original one](http://thebuildingcoder.typepad.com/blog/2011/02/create-a-pipe-cap.html) on which you submitted all your comments :-)

I wish you much success and all the best in your new role!

So long, and
[thanks for all the fish!](http://en.wikipedia.org/wiki/So_Long,_and_Thanks_for_All_the_Fish)

Back to the nuts and bolts of the Revit API, here is a case brought up and eventually solved by Mario Guttman of Perkins+Will:

#### Project Wide Data Storage

Here is a simple question that has been asked repeatedly, so I guess it is worthwhile pointing out:

**Question:** Is it possible to save some data in the Revit model that has no relation to any specific Revit model building element?

For instance, in AutoCAD I can store data in the named objects dictionary, and I can store xdata or xrecord to be saved with an entity.

I know that it is possible to store extensible storage data on individual Revit elements, but I need a place to save data applying to the entire model.

How can I achieve this, please?

**Answer:** Although extensible storage is indeed associated with individual elements, you can easily store global data in certain singleton elements.

There are several such elements that are always guaranteed to exist, and to exist only once, so-called singleton instances.
One such instance is the ProjectInformation element, whose category is OST\_ProjectInformation.

It was commonly used to store global shared parameters in the past, and the same of course applies for extensible storage as well:

- [Define a new parameter](http://thebuildingcoder.typepad.com/blog/2008/11/defining-a-new-parameter.html)- [Store project data](http://thebuildingcoder.typepad.com/blog/2009/07/store-project-data.html)- [Add a category to a parameter binding](http://thebuildingcoder.typepad.com/blog/2009/09/adding-a-category-to-a-parameter-binding.html)

In another case, an arbitrary element was required to
[format a parameter value](http://thebuildingcoder.typepad.com/blog/2011/12/unit-conversion-and-display-string-formatting.html),
and the ProjectInformation singleton was used for that also.

For complete background information on extensible storage in Revit 2012, I would suggest looking at my Autodesk University 2011
[class](http://au.autodesk.com/?nd=event_class&session_id=9263&jid=1725932) and
[lab](http://au.autodesk.com/?nd=event_class&session_id=9726&jid=1725932) on
this topic, which tell the complete story.
I recently published the
[handouts and sample material](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html#3) for
those here on the blog to provide more immediate access.

#### DataStorage Element

Better still for your case, one of the
[add-in integration features](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2) introduced
by the
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html)
is a new DataStorage element to enable add-ins to store extensible storage data completely independently of all BIM database elements.

The DataStorage element is only accessible through the API, not through the user interface. Here is the excerpt on it from the 'What's New' section of the Revit Quasar SDK help file RevitAPI.chm:

The new DataStorage class represents an element which API applications can create logically organize different sets of ExtensibleStorage entities into separate elements. This allows an application to update one set of data in a local workshared project without locking other elements.

To demonstrate the usage of the new DataStorage element, here is a very nice set of samples by Victor Chekalin, Виктор  Чекалин.

#### Simple DataStorage Sample

Storing data in a DataStorage element is very easy.

You can follow these steps to write your own data to a DataStorage element:

1. Create a new DataStorage using static method Create of DataStorage class.- Create a Schema.- Create an Entity which contains data you want to save.- Set entity to the created DataStorage.

To read data from DataStorage you need to:

1. Find DataStorage in the project (using FilteredElementCollector).- Get the entity from the retrieved DataStorage.- Read data from the Entity.

For example, assume we need to store in each document when a document was created and by whom.

On DocumentCreated event write information about the creating user:
```csharp
  using( Transaction t = new Transaction(
    doc, "Create created info" ) )
  {
    t.Start();

    // Create data storage in new document

    DataStorage createdInfoStorage
      = DataStorage.Create( doc );

    // Create entity which store created info

    Entity entity = new Entity(
      CreatedInfoSchema.GetSchema() );

    entity.Set( "CreatedUser",
      Environment.UserName );

    entity.Set( "CreatedDate",
      DateTime.Now.ToString() );

    // Set entity to the data storage element

    createdInfoStorage.SetEntity( entity );

    t.Commit();
  }
```

And read information later:
```python
  // Retrieve data storage

  FilteredElementCollector collector =
      new FilteredElementCollector( doc );

  var dataStorage =
      collector
      .OfClass( typeof( DataStorage ) )
      .FirstElement();

  if( dataStorage == null )
  {
    message = "No data storage found "
      + "in current project";

    return Result.Failed;
  }

  // Retrieve entity from the data storage

  Entity createdInfoEntity = dataStorage.GetEntity(
    CreatedInfoSchema.GetSchema() );

  if( !createdInfoEntity.IsValid() )
  {
    message = "Data storage doesn't "
      + "have CreatedInfoSchema";

    return Result.Failed;
  }

  var createdUser = createdInfoEntity.Get<string>(
    "CreatedUser" );

  var createdDate = createdInfoEntity.Get<string>(
    "CreatedDate" );

  StringBuilder sb = new StringBuilder();

  sb.AppendFormat( "Created user: {0}\r\n",
    createdUser );

  sb.AppendFormat( "Created date: {0}",
    createdDate );

  TaskDialog.Show( "Project created info",
    sb.ToString() );
```

#### Identifying DataStorage Elements

The simple DataStorage example above very simple indeed.
It has one big disadvantage, though: In a project there could be a lot of DataStorage elements including elements created by other developers.
So, we must implement a method to differentiate one DataStorage from another and retrieve only the DataStorage elements we really need.

The simplest way to achieve this is to search for a singleton DataStorage element of the expected Schema.

Let’s look at another example in which we store our own settings for the entire project which we could use in our add-in.

At first, create a Settings class
```csharp
  public class MyProjectSettings
  {
    public int Parameter1 { get; set; }
    public string Parameter2 { get; set; }
  }
```

Next, create a schema for it:
```python
  public static class MyProjectSettingsSchema
  {
    readonly static Guid schemaGuid = new Guid(
      "{9DBE0174-AA01-4CDD-BA86-96DE1FDCE041}" );

    public static Schema GetSchema()
    {
      Schema schema = Schema.Lookup( schemaGuid );

      if( schema != null ) return schema;

      SchemaBuilder schemaBuilder =
          new SchemaBuilder( schemaGuid );

      schemaBuilder.SetSchemaName(
        "MyProjectSettings" );

      schemaBuilder.AddSimpleField(
        "Parameter1", typeof( int ) );

      schemaBuilder.AddSimpleField(
        "Parameter2", typeof( string ) );

      return schemaBuilder.Finish();
    }
  }
```

For reading and writing settings to the DataStorage, we can implement a class like the following MyProjectSettingStorage:
```csharp
class MyProjectSettingStorage
{
  readonly Guid settingDsId = new Guid(
    "{A71F620F-BD0D-46DD-AECD-AFDEF0DFFD74}" );

  public MyProjectSettings ReadSettings(
    Document doc )
  {
    var settingsEntity = GetSettingsEntity( doc );

    if( settingsEntity == null
      || !settingsEntity.IsValid() )
    {
      return null;
    }

    MyProjectSettings settings =
        new MyProjectSettings();

    settings.Parameter1 = settingsEntity.Get<int>(
      "Parameter1" );

    settings.Parameter2 = settingsEntity.Get<string>(
      "Parameter2" );

    return settings;
  }

  public void WriteSettings(
    Document doc,
    MyProjectSettings settings )
  {
    var settingDs = GetSettingsDataStorage(
      doc );

    if( settingDs == null )
    {
      settingDs = DataStorage.Create( doc );
    }

    Entity settingsEntity = new Entity(
      MyProjectSettingsSchema.GetSchema() );

    settingsEntity.Set( "Parameter1",
      settings.Parameter1 );

    settingsEntity.Set( "Parameter2",
      settings.Parameter2 );

    // Identify settings data storage

    Entity idEntity = new Entity(
      DataStorageUniqueIdSchema.GetSchema() );

    idEntity.Set( "Id", settingDsId );

    settingDs.SetEntity( idEntity );
    settingDs.SetEntity( settingsEntity );
  }

  private Entity GetSettingsEntity(
    Document doc )
  {
    FilteredElementCollector collector =
        new FilteredElementCollector( doc );

    var dataStorages =
        collector.OfClass( typeof( DataStorage ) );

    // Find setting data storage

    foreach( DataStorage dataStorage in dataStorages )
    {
      Entity settingEntity =
        dataStorage.GetEntity( MyProjectSettingsSchema.GetSchema() );

      // If a DataStorage contains
      // setting entity, we found it

      if( !settingEntity.IsValid() ) continue;

      return settingEntity;
    }

    return null;
  }
}
```

Remember, the DataStorage element storing project-wide settings must be a singleton.
So, before writing settings we look for an existing DataStorage and if it doesn’t exist, create it.
For searching, we check each data storage element for the Setting entity.

Of course, you could also use another way to differentiate one DataStorage element from another.
For example, by creating an auxiliary identification schema and attaching an entity instance of it to each DataStorage element.
Using this entity, you can check whether a given DataStorage element is the one we need or not when retrieving it.
You might also be able to make use of the built-in Revit API Element.UniqueId property, or other means of identification.
It depends on your tasks.

Here is
[DataStorageSample.zip](zip/DataStorageSample.zip) containing
the full Visual Studio solution, project files, source code, and add-in manifests for Victor's two DataStorage samples.

Many thanks to Victor for providing and documenting this, providing such an easy entry point for everybody!