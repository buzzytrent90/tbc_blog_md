---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.911988'
original_url: https://thebuildingcoder.typepad.com/blog/0927_dreamseat_couchdb.html
post_number: 0927
reading_time_minutes: 10
series: general
slug: dreamseat_couchdb
source_file: 0927_dreamseat_couchdb.htm
tags:
- csharp
- elements
- family
- levels
- references
- revit-api
- rooms
- schedules
- views
title: Desktop to Cloud via DreamSeat CouchDB Client
word_count: 1969
---

### Desktop to Cloud via DreamSeat CouchDB Client

Continuing the research and development for my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3),
I now started looking at storing the plan view room and furniture boundary loop data in the cloud.

I wound up the main Revit add-in functionality using the
[ExtrusionAnalyzer](http://thebuildingcoder.typepad.com/blog/2013/04/extrusion-analyser-and-plan-view-boundaries.html)
to create a plan view boundary profile for the furniture and equipment family instances,
[sort and orient its output curves](http://thebuildingcoder.typepad.com/blog/2013/03/sort-and-orient-curves-to-form-a-contiguous-loop.html),
[determine their bounding box](http://thebuildingcoder.typepad.com/blog/2013/04/curve-following-face-and-bounding-box-implementation.html) and
[visualise the results](http://thebuildingcoder.typepad.com/blog/2013/04/geosnoop-net-boundary-curve-loop-visualisation.html) in
a dynamically generated GeoSnoop .NET form for verification.
The plan view boundary loop data is handily managed in
[integer-based 2D points](http://thebuildingcoder.typepad.com/blog/2013/04/geosnoop-net-boundary-curve-loop-visualisation.html#2) packaged
by the
[JtLoops](http://thebuildingcoder.typepad.com/blog/2013/04/geosnoop-net-boundary-curve-loop-visualisation.html#4) class.

That wraps up the initial add-in implementation.
We will certainly return to it and add more functionality when the time comes to update the model with the edits applied on the mobile device.

#### Moving to the Cloud

The next step is to implement a cloud-based data repository to make this information ubiquitously available for consumption on mobile devices and elsewhere.

I mentioned looking at the
[CouchDB](http://thebuildingcoder.typepad.com/blog/2013/03/relax-simple-free-cloud-based-data-repository-with-nosql-couchdb-and-iriscouch.html#4) database
and the free
[IrisCouch](http://thebuildingcoder.typepad.com/blog/2013/03/relax-simple-free-cloud-based-data-repository-with-nosql-couchdb-and-iriscouch.html#5) hosting
service as a candidate repository implementation.

My colleague
[Philippe Leefsma](http://adndevblog.typepad.com/cloud_and_mobile/philippe-leefsma.html)
took that same thought and very quickly and easily implemented a
[simple and efficient restful database](http://adndevblog.typepad.com/cloud_and_mobile/2013/03/experimenting-couchdb-a-simple-and-efficient-restful-database.html) using
those components to host and consume images of CAD model data.

Philippe used the
[DreamSeat](http://thebuildingcoder.typepad.com/blog/2013/03/relax-simple-free-cloud-based-data-repository-with-nosql-couchdb-and-iriscouch.html#6) .NET
CouchDB wrapper library to easily access the functionality from within a .NET add-in.

He added his own asynchronous support, in addition to the asynchronous support already provided by DreamSeat itself, and attached the complete implementation of a .NET client and console allowing for upload and viewing of pictures.

I added DreamSeat support to feed a CouchDB database with my plan view room and equipment boundary loop data in just a very few simple steps:

- [Implement SVG path properties](#3)
- [Add references to the DreamSeat libraries](#4)
- [Implement couch document wrappers](#5)
- [Instantiate, populate and save to cloud](#6)
- [Looking at the cloud-based data repository](#7)

#### Implement SVG Path Properties

First of all, I need to decide how to store the boundary loops in my data repository.

I initially thought I would save the integer values, and may still do so.

For the moment, though, I implemented a much simpler placeholder by adding methods to the point and loop classes to serialise the data to a single complete
[SVG path](http://www.w3.org/TR/SVG/paths.html#PathData) string
representation.

A single string is easy to add to the database and can be reused directly for the SVG visualisation.
I am not planning to modify the boundary loops in any way, so I might as well manage them in static strings.
The only disadvantage would be if I wanted to scale the coordinates differently.
If that need arises, I may switch to uploading the structured integer-based point and loop data instead.
We'll cross that bridge when we come to it.

An SVG path element contains a path data attribute that can include moveto, line, curve, arc and closepath instructions.

For an individual point, I simply output its two coordinate values.
For the first loop vertex, I prepend an 'M' for moveto.
For the second one, I prepend an 'L' for line-to.
For all following vertices, the 'L' can be omitted.
At the end, a 'Z' is appended to close the path.

For an individual point,
[Point2dInt](http://thebuildingcoder.typepad.com/blog/2013/04/geosnoop-net-boundary-curve-loop-visualisation.html#2),
the SvgPath property is therefore implemented like this:

```csharp
  /// <summary>
  /// Return a string suitable for use in an SVG
  /// path. For index i == 0, prefix with 'M', for
  /// i == 1 with 'L', and otherwise with nothing.
  /// </summary>
  public string SvgPath( int i )
  {
    return string.Format( "{0}{1} {2}",
      ( 0 == i ? "M" : ( 1 == i ? "L" : "" ) ),
      X, Y );
  }
```

For an individual boundary loop,
[JtLoop](http://thebuildingcoder.typepad.com/blog/2013/04/geosnoop-net-boundary-curve-loop-visualisation.html#3),
the member vertices are converted to their SVG string representation using LINQ and concatenated using the string Join method:

```csharp
  /// <summary>
  /// Return an SVG path specification, c.f.
  /// http://www.w3.org/TR/SVG/paths.html
  /// M [0] L [1] [2] ... [n-1] Z
  /// </summary>
  public string SvgPath
  {
    get
    {
      return string.Join( " ",
        this.Select<Point2dInt,string>(
          (p,i) => p.SvgPath( i ) ) )
        + "Z";
    }
  }
```

For a collection of loops,
[JtLoops](http://thebuildingcoder.typepad.com/blog/2013/04/geosnoop-net-boundary-curve-loop-visualisation.html#4),
concatenate the individual loop paths:

```csharp
  /// <summary>
  /// Return the concatenated SVG path
  /// specifications for all the loops.
  /// </summary>
  public string SvgPath
  {
    get
    {
      return string.Join( " ",
        this.Select<JtLoop,string>(
          a => a.SvgPath ) );
    }
  }
```

With these properties in place, I can finally set about uploading the data to the cloud.

#### Add References to the DreamSeat Libraries

Just like Philippe, I use the DreamSeat wrapper to access the CouchDB functionality from my .NET Revit add-in.

I simply added this functionality to the existing GetLoops external command.

First, I need to reference the DreamSeat .NET assemblies to access it classes and methods.

I grabbed these from the
[DreamSeat sample solution](https://github.com/vdaron/DreamSeat) on
[github](https://github.com).

For my initial minimal upload usage, all I need are DreamSeat.dll and mindtouch.dream.dll:

![DreamSeat library .NET assembly references](img/dreamseat_references.png)

#### Implement Couch Document Wrappers for my Data

Data in CouchDB is stored in documents.

DreamSeat provides a CouchDocument base class for deriving your own document wrappers.

I implemented derived classes to store some minimal model, level, room and furniture data as follows:

```csharp
class DbObj : CouchDocument
{
  protected DbObj()
  {
    Type = "obj";
  }
  public string Type { get; protected set; }
  //public string UniqueId { get; set; }
  public string Description { get; set; }
  public string Name { get; set; }
}

class DbModel : DbObj
{
  public DbModel()
  {
    Type = "model";
  }
}

class DbLevel : DbObj
{
  public DbLevel()
  {
    Type = "level";
  }
  public string ModelId { get; set; }

}

class DbRoom : DbObj
{
  public DbRoom()
  {
    Type = "room";
  }
  public string LevelId { get; set; }
  public string Loops { get; set; }
}

class DbFurniture : DbObj
{
  public DbFurniture()
  {
    Type = "furniture";
  }
  public string RoomId { get; set; }
  public string Loop { get; set; }
  public string Transform { get; set; }
}
```

I initially thought of identifying documents with the auto-generated CouchDB identifiers, but then chose to use the UniqueId already provided by Revit to identify the objects in CouchDB as well.

Type, description and name are fields added to all my objects.

Type identifies the object type and can be one of model, level, room or furniture.

The only information of interest to me is:

- Mutual relationships, i.e. which room belongs to what level, where does the furniture go, etc., handled by the ModelId, LevelId and RoomId properties.
- Boundary loop data for display purposes, currently stored as strings representing the SVG path information produced by the properties described above.
- The transform applied to the furniture, which can be moved and rotated in the mobile device room editor and then needs to be updated back to the model again.

Pretty minimal and readable, isn't it?

#### Instantiate, Populate and Save to Cloud

Here is the entire code to connect to the cloud database host, open the 'rooms' database, and populate the room and furniture data for a given room, either creating new records or updating existing ones:

```csharp
void UploadRoom(
  Room room,
  List<Element> furniture,
  JtLoops roomLoops,
  JtLoops furnitureLoops )
{
  CouchClient client = new CouchClient(
    "jt.iriscouch.com", 5984 );

  CouchDatabase db = client.GetDatabase(
    "rooms", true );

  string uid = room.UniqueId;

  DbRoom dbRoom;

  if( db.DocumentExists( uid ) )
  {
    dbRoom = db.GetDocument<DbRoom>( uid );

    Debug.Assert(
      dbRoom.Id.Equals( room.UniqueId ),
      "expected equal ids" );

    dbRoom.Description = Util.ElementDescription(
      room );

    dbRoom.Name = room.Name;
    dbRoom.LevelId = room.Level.UniqueId;
    dbRoom.Loops = roomLoops.SvgPath;

    dbRoom = db.UpdateDocument<DbRoom>( dbRoom );
  }
  else
  {
    dbRoom = new DbRoom();

    dbRoom.Id = uid;
    dbRoom.Description = Util.ElementDescription(
      room );

    dbRoom.Name = room.Name;
    dbRoom.LevelId = room.Level.UniqueId;
    dbRoom.Loops = roomLoops.SvgPath;
    dbRoom = db.CreateDocument<DbRoom>( dbRoom );
  }

  int i = 0;

  foreach( Element f in furniture )
  {
    uid = f.UniqueId;
    if( db.DocumentExists( uid ) )
    {
      DbFurniture dbf = db.GetDocument<DbFurniture>(
        uid );

      dbf.Description = Util.ElementDescription( f );
      dbf.Name = f.Name;
      dbf.RoomId = room.UniqueId;
      dbf.Loop = furnitureLoops[i++].SvgPath;
      dbf = db.UpdateDocument<DbFurniture>( dbf );
    }
    else
    {
      DbFurniture dbf = new DbFurniture();
      dbf.Id = f.UniqueId;
      dbf.Description = Util.ElementDescription( f );
      dbf.Name = f.Name;
      dbf.RoomId = room.UniqueId;
      dbf.Loop = furnitureLoops[i++].SvgPath;
      dbf = db.CreateDocument<DbFurniture>( dbf );
    }
  }
}
```

As you can see, I am storing the data in my IrisCouch hosted database in the cloud.

#### Looking at the Cloud-based Data Repository

After running this command, I can retrieve the top-level database information in JSON format:

```
{"db_name":"rooms", "doc_count":10, "doc_del_count":11,
"update_seq":54, "purge_seq":0, "compact_running":false,
"disk_size":102511, "data_size":6921,
"instance_start_time":"1365674085202773",
"disk_format_version":6, "committed_update_seq":54}
```

This is what an individual room looks like in futon, the CouchDB management console:

![View of a room in futon](img/room_in_futon.png)

Here is the raw JSON representation of the same data:

```
{
  "_id": "4d2e6fd6-eb44-4e2a-98c5-271decaa9225-00033e9f",
  "_rev": "2-a863fbc7b2294ef4d8630bc961221b24",
  "levelId": "e3e052f9-0156-11d5-9301-0000863f27ad-00000137",
  "loops": "M2753 3087 L-4446 3087 -4446 587 -746 587 -746 -1212 2753 -1212Z M298 -112 L298 587 1698 587 1698 -112Z",
  "type": "room",
  "description": "Room Rooms <212639 Room 1>",
  "name": "Room 1"
}
```

So far, so good.

Once I got to here, I wanted to start working on the visualisation in SVG and had a very nasty surprise: the strict
[same origin policy](http://www.w3.org/Security/wiki/Same_Origin_Policy) prevents
my simple JavaScript application from accessing my IrisCouch domain, so I cannot easily read the data.

The discovery robbed me of a full night's sleep, but I think I have found a good way out of that dilemma as well.

Wish me luck!

---

# Cloud and Mobile

### Desktop to Cloud via DreamSeat CouchDB Client

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

I mentioned the idea of
[using CouchDB and IrisCouch as a simple cloud-based data repository](http://thebuildingcoder.typepad.com/blog/2013/03/relax-simple-free-cloud-based-data-repository-with-nosql-couchdb-and-iriscouch.html) option,
and Philippe picked up that idea to demonstrate a
[simple and efficient restful database](http://adndevblog.typepad.com/cloud_and_mobile/2013/03/experimenting-couchdb-a-simple-and-efficient-restful-database.html) using
those components to host and consume images of CAD model data.

Now I presented my own example of
[pushing desktop data to the cloud via a DreamSeat CouchDB client](http://thebuildingcoder.typepad.com/blog/2013/04/desktop-to-cloud-via-dreamseat-couchdb-client.html) to
store 2D plan view room and furniture boundaries in a cloud-hosted data repository.

Check it out, and please let us know what you think of it!