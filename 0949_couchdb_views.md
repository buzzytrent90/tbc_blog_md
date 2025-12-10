---
post_number: "0949"
title: "CouchDB Implementation and GitHub Repository"
slug: "couchdb_views"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'levels', 'parameters', 'references', 'revit-api', 'rooms', 'views', 'walls', 'windows']
source_file: "0949_couchdb_views.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0949_couchdb_views.html"
---

### CouchDB Implementation and GitHub Repository

I more or less completed the research for my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/05/my-cloud-based-2d-editor-implementation-status.html#2).

I'll describe some parts of that, and also address

- [Three aspects of life and programming](#2)
- [Room editor navigation](#3)
- [Nested asynchronous database call-back functions](#4)
- [Roomedit – my first GitHub project](#5)
- [Debugging JavaScript with Firebug](#6)
- [Screen snapshots on mobile devices](#7a)
- [Open source components used by Autodesk](#7)
- [Retrieve slabs above specific walls](#8)

#### Lazy, Simple and Perfect

You will like the first slide of my upcoming tech summit presentation.

I start off with a couple of quotes on three fundamental aspects of software engineering and life in general:

![Quotes on three fundamental aspects of software engineering and life](img/quotes_on_three_aspects_slide.png)

- Lazy

- ... develop the three great virtues of a programmer:
  [laziness, impatience, and hubris](http://c2.com/cgi/wiki?LazinessImpatienceHubris) – *Larry Wall*

- Simple

- Simplicity is the ultimate sophistication – *Leonardo da Vinci*
- There is no greatness where there is no simplicity – *Leo Tolstoy*
- KISS

- Perfect

- Perfection is achieved, not when there is nothing more to add, but when there is nothing left to take away – *Antoine de Saint-Exupéry*

Basically, I demonstrate how easy it is to implement simple model navigation and 2D graphical editing using server-side scripting so that it works on all mobile devices.

So it is all about keeping it simple and easy.

#### Room Editor Navigation

The last instalment on this topic included a two-minute minimal demo recording illustrating its use and started describing the simplest aspect of the implementation, the server-side generated index.html
[home page](http://thebuildingcoder.typepad.com/blog/2013/05/my-cloud-based-2d-editor-implementation-status.html#5) that
lists all models and prompts to select one of them.

From there, one navigates through the hierarchy of model > level > room > furniture and equipment.
The latter finally reference the symbol objects that determine their geometry.

Now lets move on to the more interesting parts where the inter-related objects need to execute more complex queries to establish their relationships.
I still have not covered the juicy part of rendering the SVG model, and the even juicier part of interacting with it, editing it, and updating the database with the changes made, either.

As said, the navigation logic is pretty trivial:

- Home page: list all models and prompt to select one of them.
- Model selected: list all levels in model and prompt to select one of them.
- Level selected: list all rooms on the level and prompt to select one of them.
- Room selected: display the room boundary and the furniture and equipment instances graphically and enable editing their location and rotation.

Basically, the entire implementation is all squeezed into one single overgrown index.html file.

Moving on from the home page to the second simplest part of it, we have selected a specific model and list all the levels it contains.

This navigation takes place by invoking the same index.html URL and adding a modelId parameter to it.

This parameter is picked up and used to query the database for all levels in the specified model.

To retrieve only the required ones, I use a CouchDB view mapping model id keys to level documents like this:

```
  map_model_to_level = {
    map: function (doc) {
      if( 'level' == doc.type ) {
        emit(doc.modelId, doc);
      }
    }
  };
```

I can specify a key to CouchDB when querying this view, i.e. make a REST API call with a parameter like this:

```
  .../roomedit/_design/roomedit/_views?key=12345
```

This will retrieve only those level documents from the database whose modelId value matches the given key.

Here is the part of index.html handling that interaction:

```
. . .

else if ( paramdict.hasOwnProperty( 'modelid' ) ) {

  //======================================================
  // display a menu of all levels in selected model
  //======================================================

  var mid = paramdict['modelid'];
  db.getDoc( mid,
    function(err,resp) {
      if (err) {
        return alert(JSON.stringify(err));
      }
      var modeldoc = resp;
      db.getView('roomedit',
        'map_model_to_level',
        { key: JSON.stringify(mid) },

        function (err, data) {
          if (err) {
            return alert(err);
          }
          var n = data.rows.length;
          for (var i = 0; i < n; ++i) {
            var doc = data.rows[i].value;
            if( doc.modelId != mid ) {
              alert( 'model ids differ: doc '
                + doc.modelId + ", url " + mid );
            }
            var s = url + '?levelid=' + doc._id;
            $('<li/>')
              .append($('<a>').attr('href',s).text(doc.name))
              .appendTo('#navigatorlist');
          }
          var p = $('<p/>').appendTo('#content');
          p.append( $('<a/>').text('Home').attr('href',url) );
          p.append( document.createTextNode( three_spaces ) );
          p.append( $('<a/>').text('Back').attr('href',url) );

          $('<p/>')
            .text( 'Please select a level in the list below.' )
            .appendTo('#content');

          var table = $('<table/>').appendTo('#content');
          table.append( '<tr><td>' + 'Model:'
            + '</td><td>' + modeldoc.name + '</td></tr>' );

          var prompt = n.toString() + ' level'
            + pluralSuffix( n ) + ' in model ';

          var p = $('<p/>').text( prompt ).appendTo('#content');
          p.append( $('<i/>').text( modeldoc.name ) );
          p.append( document.createTextNode( dotOrColon( n ) ) );
        }
      );
    }
  );
}
```

As I already mentioned, this is populating an super simple HTML skeleton containing the following nodes:

```
  <h1>Room Editor</h1>

  <div id="content"></div>

  <ul id="navigatorlist"></ul>
```

Here is the result in a model named 'Roombook' containing a single level 'Fondations':

![Room editor list of levels](img/roomedit_list_levels.png)

#### Nested Asynchronous Database Call-back Functions

One challenge for me in the room editor CouchDB implementation was understanding how the asynchronous database call-back functions work, and how I can nest them to retrieve all the results I need.

In the case above, the situation is still very simple: I need to perform two database queries:

- Query a specific document, the model, using the db.getDoc function.
- Query all the levels it contains, which are returned by the map\_model\_to\_level view with the model id key, using the getView function.

Both the getDoc and getView functions take an asynchronous call-back function as an argument.

I found out the hard way that I can nest these within each other to ensure that I have all up-to-date results before I display my page.

This gets a bit more hairy as the number of queries and thus the number of nested functions grows...

#### Roomedit – My First GitHub Project

Anyway, I have it all happily running now, and published the
[roomedit CouchDB application](https://github.com/jeremytammik/roomedit.git) implementing
this as my first ever GitHub project.

#### Debugging JavaScript with FireFox FireBug

Everybody makes errors.
In my HTML and JavaScript code, these sometimes caused my index.html not to display anything at all.

Often this was due to some trivial JavaScript typo that would have been really hard to chase down.

Enter Firebug, thank God.

This is a great tool that I have started to rely on greatly to help me out of tight corners.

The Firebug debugging console is integrated in FireFox and includes support for examining and debugging all kinds of aspects of a web page.

You can open the Firebug console on any web page through Tools > Web Developer > Firebug > Open Firebug.
That displays a new panel at the bottom of the browser with tabs for

- Console
- HTML
- CSS
- Script
- DOM
- Net
- Cookies

I use Console and Script, mostly.

The former displays the location of a JavaScript error and jumps to it when loading the file.
It also lists the URLs visited and stuff like that, which is very useful for analysing the REST API calls being made.

The script tab provides a JavaScript debugger allowing you to set breakpoints and analyse variable values etc.

There are lots of tutorials and documentation available on this feature, such as these ones on
[Firebug](http://www.w3resource.com/web-development-tools/firebug-tutorials.php) and
[JavaScript debugging](http://www.developerfusion.com/article/139949/debugging-javascript-with-firebug),
so I will not trouble you any further with my description.

To tell the truth, though, I never looked at any of them.

The thing just worked for me, is incredibly valuable, and I am grateful.

#### Screen Snapshots on Mobile Devices

Here is a quick guide how to create a screen snapshot on various types of mobile devices:

- iPhone – Press the Sleep/Wake and Home buttons together; the screen will fade temporarily, signalling that the screen shot was saved in your Camera Roll:
  ![Screen snapshot on iPhone](img/screen_snapshot_iphone.jpg)
- Android – Press the Power and Home buttons together, or hold down the Lock/Power and Volume Down buttons together:
  ![Screen snapshot on Android](img/screen_snapshot_android.jpg)
- Windows 8 – Press the Lock/Power and Home/Windows buttons together:
  ![Screen snapshot on Windows 8](img/screen_snapshot_win8.jpg)

#### Open Source Components used by Autodesk

A colleague pointed out this
[open source repository](http://www.autodesk.com/lgplsource) containing
all open source components used in AutoCAD, Map 3D and other Autodesk products.

I was actually surprised how short the list is.
Anyway, it might be useful to know about.

#### Retrieve Slabs Above Certain Walls

Let's wrap this up with a standard Revit API question:

**Question:** I want to determine the slab above the walls of a particular room.

How can retrieve those using the Revit API, please?

**Answer:** As so often in Revit, there are a number of different possibilities, and it depends in part upon how the building has been modelled.

In a structural model, you may be able to query the analytical support data. I have no idea whether this information is included there, but my naive assumption would be that this should be possible.

Much more generally, you can analyse and query the geometry in a number of different ways.

The two possibilities that immediately spring to mind are:

- Use the ray casting functionality provided by the Document FindReferencesWithContextByDirection method and the ReferenceIntersector wrapper class.
- Use the geometric proximity functionality provided by some element filters, e.g.

The discussion on
[determining the columns supporting a beam](http://thebuildingcoder.typepad.com/blog/2013/03/supporting-columns.html#2) demonstrates
several different variations of the latter.
Earlier blog posts include examples of the former as well.

**Response:** I was able to use the functionality for finding the supporting columns for beams from your link to solve my issue as well.

In my case, beam.GetGeneratingElementIds returns the desired supported floors.

I also tested using an ElementIntersectsSolidFilter to retrieve all walls that intersect the floor using the floor solid geometry object:
```csharp
  Solid solid = null;
  ElementFilter slabIntersectFilter
    = new ElementIntersectsSolidFilter( solid );

  FilteredElementCollector walls
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .OfClass( typeof( Wall ) )
      .WherePasses( slabIntersectFilter );
```

This also solved my issue.
It can be the alternative when beams are not there.

---

# Cloud and Mobile

### Debugging Asynchronous CouchDB Callbacks & Snapshots

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

I more or less completed the research for my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/05/my-cloud-based-2d-editor-implementation-status.html#2).

Here are the main items in my discussion of that and one or two other cloud and mobile related topics:

- [Three aspects of life and programming](http://thebuildingcoder.typepad.com/blog/2013/05/couchdb-implementation-and-github-repository.html#2)
- [Room editor navigation](http://thebuildingcoder.typepad.com/blog/2013/05/couchdb-implementation-and-github-repository.html#3)
- [Nested asynchronous database call-back functions](http://thebuildingcoder.typepad.com/blog/2013/05/couchdb-implementation-and-github-repository.html#4)
- [Roomedit – my first GitHub project](http://thebuildingcoder.typepad.com/blog/2013/05/couchdb-implementation-and-github-repository.html#5)
- [Debugging JavaScript with Firebug](http://thebuildingcoder.typepad.com/blog/2013/05/couchdb-implementation-and-github-repository.html#6)
- [Screen snapshots on mobile devices](http://thebuildingcoder.typepad.com/blog/2013/05/couchdb-implementation-and-github-repository.html#7a)
- [Open source components used by Autodesk](http://thebuildingcoder.typepad.com/blog/2013/05/couchdb-implementation-and-github-repository.html#7)