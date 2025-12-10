---
post_number: "0914"
title: "Relax – Simple Free Cloud Based Data Repository with NoSQL, CouchDB, and IrisCouch"
slug: "nosql_couchdb_iris"
author: "Jeremy Tammik"
tags: ['elements', 'geometry', 'revit-api', 'schedules']
source_file: "0914_nosql_couchdb_iris.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0914_nosql_couchdb_iris.html"
---

### Relax – Simple Free Cloud Based Data Repository with NoSQL, CouchDB, and IrisCouch

Last week, I mentioned my proposal for the Autodesk Technical Summit in June, to present a
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3).

That will obviously required a cloud-based data repository.
Talking with my colleagues, all of their suggestions were oriented towards the Microsoft Azure and Amazon solutions discussed so far on the
[cloud and mobile DevBlog](http://adndevblog.typepad.com/cloud_and_mobile).

I was searching for something freer, more open source, and less proprietary, and was rather surprised to discover an utterly relaxing solution that so far looks absolutely perfect for me and my needs.

I first stumbled upon IrisCouch, which led me to CouchDB and NoSQL, which all make me very happy.
For didactical purposes, I will discuss them in inverted order, though.

Before I get to that, I would also like to mention my first ski tour of the year:

- [Ski Tour Season Opening](#2)
- [NoSQL](#3)
- [CouchDB](#4)
- [IrisCouch](#5)
- [ID and GUID in Revit Documents](#6)
- [Visual Revit Programming with Dynamo](#7)

#### My Ski Tour Season Opening

![Jeremy on Clariden](file:///j/photo/jeremy/2013/2013-03-23_clariden/p1010311.jpg)

I went on a ski crossing of the Glarner Alps last weekend.

We took the cable lift up to the Fisetenpass and climbed the Gemsfairenstock, Clariden and Piz Cazarauls.
We were hoping to get on to the Schärhorn as well, but had to give that up due to bad weather conditions.
Instead we headed over the glacier to the Chammlijoch und down the Iiswaendli back to the Klausenpass.

This was my second ascent of the Clariden Mountain, following the
[last one](http://thebuildingcoder.typepad.com/blog/2011/09/revit-ifc-exporter-released-as-open-source.html) in
September 2011, from the opposite direction and obviously under very different conditions.

A great way to get away from computers for a day or two and face the natural elements instead, at their wildest and fiercest.

#### Relax – with NoSQL

As said, researching a simple efficient cloud-based data repository for my
[SVG editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3),
I discovered
[NoSQL](http://en.wikipedia.org/wiki/NoSQL),
'not only SQL', a whole new class of database model, hitherto unknown to me.

A NoSQL database provides a simple, lightweight mechanism for storage and retrieval of data that provides higher scalability and availability than traditional relational databases.
The NoSQL data stores use looser consistency models to achieve horizontal scaling and higher availability.
Some NoSQL systems do allow SQL-like query language to be used.

NoSQL database systems are often highly optimized for retrieval and appending operations and often offer little functionality beyond record storage, e.g. key–value stores.
The reduced run-time flexibility compared to full SQL RDBM systems is compensated by marked gains in scalability and performance for certain data models.

Since my application will only require storage of simple key-value pairs and should in theory be massively scalable, this sounds like an optimal choice.
In practice, I only expect it to be a toy, of course; the scalability is required only to make it a useful example toy.

#### Relax – with CouchDB

[Apache CouchDB](http://couchdb.apache.org) is
a NoSQL implementation, one of a new breed of database management systems.

As stated by its official by-line, the one word to describe CouchDB is relax.
When you start CouchDB, you see:

```
Apache CouchDB has started. Time to relax.
```

CouchDB is built not only for, but **of** the Web.
Its design borrows heavily from web architecture and the concepts of resources, methods, and representations.
Data is self-contained:

![Self-contained documents](img/couchdb_01.png)

Replication and backup is extremely simple, basically requiring no more than a modification of the base URL.

CouchDB is implemented in the
[Erlang](http://www.erlang.org) programming
language, which is used to build massively scalable soft real-time systems with requirements on high availability with uses in telecommunication, banking, e-commerce, computer telephony and instant messaging.
Erlang's runtime system has built-in support for concurrency, distribution and fault tolerance.

Here is a
['what is' Q&A](http://couchapp.org/page/what-is-couchapp),
[getting started tutorial](http://net.tutsplus.com/tutorials/getting-started-with-couchdb) and
full 270-page O'Reilly textbook
[CouchDB – The Definite Guide](http://guide.couchdb.org).

The textbook samples make use of the
[cURL](http://en.wikipedia.org/wiki/CURL) command line tool, e.g. the following two command line statements to create a new and list a database:

```
  curl -X PUT http://127.0.0.1:5984/baseball
  {"ok":true}

  curl -X GET http://127.0.0.1:5984/_all_dbs
  ["baseball"]
```

#### Relax – with IrisCouch

As I initially mentioned, the starting point of this exploration of mine was
[IrisCouch](http://www.iriscouch.com),
which terms itself 'easy CouchDB'.
It provides a free hosting service, 'a couch in the cloud'.
You can sign up in minutes to have your own Apache CouchDB server.

For my purposes, I can install CouchDB on my local machine, develop my data repository locally, and push it all to the cloud by replicating my local database to IrisCouch and changing the base URL to use that for all interaction.

#### Relax – with DreamSeat

While I explored the theory, my colleague
[Philippe Leefsma](http://adndevblog.typepad.com/autocad/philippe-leefsma.html) went
ahead and implemented a pair of .NET applications uploading and consuming cloud-based data from an IrisCouch-hosted CouchDB database using the
[DreamSeat](https://github.com/vdaron/DreamSeat) CouchDB .NET wrapper library.

Although the DreamSeat documentation boasts support for both complete synchronous and asynchronous API, Philippe used Visual Studio 2012 and its .NET 4.5 async support that automatically and transparently implements the entire support for async call-back.
'Async' methods can use the 'await' keyword, so he wrote a couple of anync wrapper methods for standard synchronous DreamSeat calls.

Well, that provides lots of material to play with in the immediate future, and gives me hope of easily implementing my cloud-based data repository requirements.

Here are a couple of other more directly Revit API related notes:

#### ID and GUID in Revit Documents

**Question:** A short question about the way unique ID’s are managed in Revit.
I want to export a large number of RVT models through DB link to a single database and I really need to have unique ID’s in my destination tables.
So far what I’ve seen is that the records pushed to my database have a field ID which seems to be unique within each Revit file, but not with my set of files.

So is there a way to have really unique identifier like a GUID instead of simple ID in my RVT models?
Or at least is there a way to rely on a kind of GUID when we push data through DB link?

**Answer:** Each Revit element has an ID property, its element id, which is unique within the RVT document, but not globally.

Each Revit element also has a UniqueId property, which is globally unique, unless you
[do something to sabotage that](http://thebuildingcoder.typepad.com/blog/2012/08/titbits-of-the-week.html#2).

Under normal circumstances, the UniqueId should fulfil your needs.

#### Visual Revit Programming with Dynamo

One last, final note before closing: if you are interested in a visual programming environment for Revit which allows non-programmers to create workflows to automate Revit functionality and geometry creation, be sure to check out
[Dynamo](https://github.com/ikeough/Dynamo), also featured at
[AutodeskVasari.com](http://www.AutodeskVasari.com).

---

# Cloud and Mobile

### Relax – Simple Free Cloud Based Data Repository with NoSQL, CouchDB, and IrisCouch

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

I searched for a simple cloud-based data repository option and found one that seems simpler and more transparent than the Microsoft Azure and Amazon solutions discussed here so far:
[NoSQL, CouchDB, and IrisCouch](http://thebuildingcoder.typepad.com/blog/2013/03/relax-simple-free-cloud-based-data-repository-with-nosql-couchdb-and-iriscouch.html).

Check it out, and please let us know what you think of this alternative!