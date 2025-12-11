---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.436755'
original_url: https://thebuildingcoder.typepad.com/blog/1166_roomedit_handlebars.html
post_number: '1166'
reading_time_minutes: 3
series: general
slug: roomedit_handlebars
source_file: 1166_roomedit_handlebars.htm
tags:
- elements
- levels
- references
- revit-api
- rooms
- sheets
title: Room Editor with Handlebars and Refresh
word_count: 694
---

### Room Editor with Handlebars and Refresh

Somehow, I have a much harder time documenting my JavaScript exploits than my Revit API ones.

The Autodesk Tech Summit is taking place in Toronto next week, and I am making those potentially disastrous last minute changes that every sane person avoids at all costs.

#### Automatically Refresh on Save

One really important thing that I fixed now required just one single line of code:

The browser display of the model is automatically refreshed when you press 'Save'.

I remember wondering why this did not happen when I originally implemented it last year, and finally giving up, simply resorting to an additional manual button click to trigger the refresh.

Pondering my simple solution to the
[async trap](http://thebuildingcoder.typepad.com/blog/2014/05/room-editor-element-properties-and-the-async-trap.html#2) that
I fell into saving element properties, I discovered that the issue here is the exact same thing.

I need to do whatever needs to be done inside the database callback function and nowhere else.

If I add other code after initiating the database call, that code will interfere with and corrupt the database interaction.

So now my save method looks like this:

```
function save(a) {
  for( var i = 0; i < a.length; ++i ) {
    var f = a[i];
    var id = f.data("jid");

    var txy = f.transform().toString()
      .substring(1).split(/[,r]/);

    var trxy = 'R' + f.data('angle')
      + 'T' + txy[0] + "," + txy[1];

    var fdoc = f.data("doc");

    fdoc.transform = trxy;

    if( fdoc.hasOwnProperty('loop') ) {
      delete fdoc.loop;
    }
    db.saveDoc( fdoc,
      function (err, data) {
        if (err) {
          console.log(err);
          alert(JSON.stringify(err));
        }
        document.location.reload( true );
      }
    );
  }
}
```

Note the call to set the document location?

Last year, I tried to make that call on the last line of the save function instead of inside the anonymous database callback.

No-no.

#### Adding Handlebars

Today, I added a new package to my bundle: [handlebars](http://handlebarsjs.com).

It comes packaged for use with [kanso](http://kan.so), and yet I had to struggle a little bit to understand how to add it to my installation.

In the end, it was simple, just copying the package contents into my packages subfolder and adding a reference to it in my main kanso.json file, which now looks like this:

```
{
  "name": "roomedit",
  "version": "0.0.2",
  "description": "display and edit 2D room, furniture and equipment layout",
  "attachments": ["index.html","raphael-min-jt.js","roomedit.js","index2.html"],
  "modules": ["lib"],
  "load": "lib/app",
  "dependencies": {
    "attachments": null,
    "db": null,
    "handlebars":null,
    "modules": null,
    "jquery": null,
    "properties": null
  }
}
```

So far, though, I am only using it as a pretty stupid formatting tool by defining a template on the fly and stuffing it with values like this:

```
  var handlebars = require('handlebars');

  var htemplate = handlebars.compile(
    '<p>{{levels}} and '
    + '{{sheets}} in '
    + 'model <i>{{model}}</i>.</p>'
    + '<p>Please select a level or sheet:</p>' );

  var hresult = htemplate({
    levels: thingies( nLevel, 'level' ),
    sheets: thingies( nSheet, 'sheet' ),
    model: modeldoc.name});

  $('#content').append( hresult );
```

That generates two paragraph nodes that I append to the DOM, and nothing more.

I was previously achieving the exact same result using JavaScript and jquery like this:

```
  var prompt =
    nLevel.toString() + ' level' + pluralSuffix( nLevel ) + ' and ' +
    nSheet.toString() + ' sheet' + pluralSuffix( nSheet ) + ' in model ';

  $('#content')
    .append($('<p/>')
      .text( prompt )
      .append( $('<i/>').text( modeldoc.name ) )
      .append( document.createTextNode( '.' ) ) )
    .append($('<p/>')
      .text( 'Please select a level or sheet:' ) );
```

It's not really shorter.

I find it more readable, though.

And a useful exercise.

The rest of my efforts today went into refactoring my JavaScript, eliminating code duplication and ludicrously deep indenting.

#### Download

As always, the updated CouchDB database design and Kanso package definition is available from the
[roomedit GitHub repository](https://github.com/jeremytammik/roomedit),
and the version described above is
[release 2.0.0.12](https://github.com/jeremytammik/roomedit/releases/tag/2.0.0.12).

Its playmate, the RoomEditorApp Revit add-in, with its Visual Studio solution and add-in manifest remains virtually unchanged in its
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp),
and the current version is now
[2015.0.2.16](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2015.0.2.16).