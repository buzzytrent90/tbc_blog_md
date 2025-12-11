---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: news
optimization_date: '2025-12-11T11:44:16.014087'
original_url: https://thebuildingcoder.typepad.com/blog/1432_fireratingclient.html
post_number: '1432'
reading_time_minutes: 6
series: general
slug: fireratingclient
source_file: 1432_fireratingclient.md
tags:
- csharp
- doors
- elements
- family
- filtering
- levels
- parameters
- revit-api
- rooms
- sheets
- views
- windows
title: Fireratingclient
word_count: 1127
---

### Real-Time BIM Update via FireRatingCloud Windows Client
Today, I address the first item in [the to do list that I published yesterday](http://thebuildingcoder.typepad.com/blog/2016/04/real-time-bim-update-with-fireratingcloud-2017.html#11):
- Document and improve [FireRatingClient](https://github.com/jeremytammik/FireRatingCloud/tree/master/FireRatingClient),
the stand-alone Windows client – we will need this to demonstrate the real-time BIM update from arbitrary sources.
Let's look at this task in context and then address it right away:
- [Context](#2)
- [FireRatingClient](#3)
- [Adding the `modified` Field](#4)
- [Updating the `modified` Field on Edit](#5)
- [FireRatingClient Live BIM Update Demo Recording](#6)
- [Download](#7)
- [To Do](#8)
#### Context
In the past few days, I worked on updating and migrating my samples connecting BIM and the cloud:
- [RoomEditorApp for Revit 2017](http://thebuildingcoder.typepad.com/blog/2016/04/room-editor-first-revit-2017-addin-migration.html#3)
- [Real-Time BIM Update with FireRatingCloud 2017](http://thebuildingcoder.typepad.com/blog/2016/04/real-time-bim-update-with-fireratingcloud-2017.html)
The [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) demonstrates
a full round-trip real-time connection between the Revit BIM and a simplified 2D view that can be graphically edited on any mobile device.
As a side effect, it also includes support for non-graphical editing of parameter values on family instances.
The [FireRatingCloud](https://github.com/jeremytammik/FireRatingCloud) sample is somewhat simpler.
It also demonstrates a full round-trip real-time connection between the Revit BIM and an external NoSQL database, in this case connecting to a MongoDB database via a node.js web server.
It supports non-graphical editing of a specific shared parameter value representing the fire rating values on door family instance elements.
For both these applications, the database and web server can be hosted either in the cloud or locally on the same system running Revit.
Of course, global accessibility is easier to implement by hosting them on the web.
The stand-alone Windows client presented here shows that you can make selected BIM data available to absolutely anybody, absolutely anywhere, with a truly minimal amount of effort, providing full real-time round-trip editing capability.
#### FireRatingClient
This stand-alone Windows client to edit the FireRatingCloud database values was originally implemented by
Jose Ignacio Montes, [@montesherraiz](https://github.com/Montesherraiz), of [Avatar BIM](http://avatarbim.com) in
Madrid, during the [BIM Programming Workshop](http://www.bimprogramming.com) in January.
It is extremely simple and obviously not fit for real-world use in a production environment in its current state.
For instance, the project name is not displayed, and elements cannot be filtered by project – or anything else either, for that matter – so currently all database records are loaded.
That will no longer work once there are enough records in the database.
It is totally inacceptable in the long run, but perfectly fine for my demonstration purposes here.
Also, simplicity is important for easier understanding.
I will now add the newly implemented `modified` timestamp field to each door data record that it displays, and update that field automatically when the `firerating` value is edited.
The modified values are immediately written back to the mongo database.
With the new database polling functionality added yesterday to the Revit add-in, these modifications will be detected and immediately and automatically reflected in the BIM, provided we have subscribed to that notification.
So let's get going.
#### Adding the `modified` Field
In order to support the real-time BIM update from the mongo database,
I [implemented my own timestamp in C#](http://the3dwebcoder.typepad.com/blog/2016/04/fireratingcloud-document-modification-timestamp.html#5).
For simplicity, I use
a [Unix epoch timestamp](https://en.wikipedia.org/wiki/Unix_time),
i.e., the count of seconds since January 1, 1970, and store it as a simple number in MongoDB.
In C#, it is implemented by the `DoorData` class:
```csharp
public class DoorData
{
public string _id { get; set; }
public string project_id { get; set; }
public string level { get; set; }
public string tag { get; set; }
public double firerating { get; set; }
public uint modified { get; set; }
///
/// Constructor to populate instance by
/// deserialising the REST GET response.
/// summary>
public DoorData()
{
}
```
The corresponding [mongoose](http://mongoosejs.com) schema definition to hold it looks like this:

```
var doorSchema = new Schema(
  { _id          : RvtUniqueId // suppress automatic generation
    , project_id : String
    , level      : String
    , tag        : String
    , firerating : Number
    , modified   : Number },
  { _id          : false } // suppress automatic generation
);
```

I updated the FireRatingClient to display the modified field and published the result
as [release 2017.0.0.13](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.13).
You can see exactly what I changed to add the new column by looking at
the [diffs](https://github.com/jeremytammik/FireRatingCloud/compare/2017.0.0.12...2017.0.0.13).
Here is the new form in all its glory:
![FireRatingClient with modified field](img/fireratingclient_with_modified_field.png)
As I mentioned directly when initially implementing the `modified` timestamp, anyone who updates a database entry is now responsible for explicitly setting the correct modified value as well.
That is exactly what we are about to do right now for FireRatingClient.
#### Updating the `modified` Field on Edit
Updating the `modified` field each time a door data record is edited turned out to be incredibly easy.
Jose already set up the Windows client to transmit each and every interactive edit immediately to the mongo db via REST in the `ExportData` method called by the `OnDoorsCellEditFinished` event handler.
I added the timestamp update to it like this:
```csharp
void ExportData( DoorData dd )
{
uint timestamp = Util.UnixTimestamp();
dd.modified = timestamp;
string jsonResponse, errorMessage;
HttpStatusCode sc = Util.Put(
out jsonResponse, out errorMessage,
"doors/" + dd._id, dd );
}
void OnDoorsCellEditFinished(
object sender,
BrightIdeasSoftware.CellEditEventArgs e )
{
ExportData( e.RowObject as DoorData );
}
```
Now all I want to do further is set up a nice demo for you and record it.
#### FireRatingClient Live BIM Update Demo Recording
Here is an four-minute recording demonstrating the final result,
[real-time Revit BIM update via Windows Forms FireRatingClient](https://youtu.be/vJyXmxHgu9g):

#### Download
The versions of FireRatingCloud and FireRatingClient presented above
are included in [release 2017.0.0.14](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.14),
available from the [FireRatingCloud GitHub repository](https://github.com/jeremytammik/FireRatingCloud).
#### To Do
Let's update [the to do list that I published yesterday](http://thebuildingcoder.typepad.com/blog/2016/04/real-time-bim-update-with-fireratingcloud-2017.html#11).
I have several more exciting tasks lined up, all related to connecting BIM and the cloud:
- Completely rewrite the existing [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) sample to move it from CouchDB to node.js plus MongoDB.
- Implement the database portion of the [TrackChangesCloud](https://github.com/jeremytammik/TrackChangesCloud) sample.
I think I'll tackle the latter first...