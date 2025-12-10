---
post_number: "1478"
title: "The Building Coder"
slug: "roomedit3d_broadcast"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'revit-api', 'rooms', 'selection', 'sheets', 'views', 'walls', 'windows']
source_file: "1478_roomedit3d_broadcast.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1478_roomedit3d_broadcast.html"
---

### Retrieving and Broadcasting the Roomedit3dv3 Translation
As I discussed last week,
[the translation tool emits info on its activity](http://thebuildingcoder.typepad.com/blog/2016/09/warning-swallower-and-roomedit3d-viewer-extension.html#3) via
`socket.io`.
Now I want to pick up that information higher up, in the viewer transform extension, and relay it to the node.js web server or broadcast it to the rest of the world.
The existing sample call to `emit` is inside `onTxChange`, which is called on each mouse move.
We do not need to track each mouse movement in such fine detail, just the beginning and end of the translation.
![Roomedit3dv3 Forge extension in action](img/roomedit3dv3_broadcast.png)
Luckily, I can take a look back at the previous incarnation [roomedit3d](https://github.com/jeremytammik/roomedit3d),
in the module [Roomedit3dTranslationTool.js](https://github.com/jeremytammik/roomedit3d/blob/master/www/js/extensions/Roomedit3dTranslationTool.js#L333-L366),
where this is handled by the `handleButtonUp` function:

```
  this.handleButtonUp = function(event, button) {

    if( _isDirty && _externalId && _initialHitPoint ) {
      var offset = subtract_point(
        _transformControlTx.position,
        _initialHitPoint );

      _initialHitPoint = new THREE.Vector3(
        _transformControlTx.position.x,
        _transformControlTx.position.y,
        _transformControlTx.position.z );

      console.log( 'button up: external id '
        + _externalId + ' offset by '
        + pointString( offset ) );

      var data = {
        externalId : _externalId,
        offset : offset
      }

      options.roomedit3dApi.postTransform(data);

      _isDirty = false;
    }

    _isDragging = false;

    if (_transformControlTx.onPointerUp(event))
      return true;

    return false;
  };
```

Using that as a starting point for my exploration how to reproduce the same functionality in the new version, I found that this is all I need for the internal `emit` call from the translate tool to the transform extension:

```
  handleButtonUp(event, button) {

    console.log( 'transform.translate complete' );

    if (this._selection) {
      if (this._selection.dbIdArray) {
        var dbId = this._selection.dbIdArray[0]

        if(dbId) {

          var translation = new THREE.Vector3(
            this._transformMesh.position.x - this._selection.model.offset.x,
            this._transformMesh.position.y - this._selection.model.offset.y,
            this._transformMesh.position.z - this._selection.model.offset.z)

          this._viewer.getProperties(dbId, (result) => {

            var externalId = result.externalId;

            console.log( 'transform.translate complete for '
              + externalId
              + ': ' + translation.x.toFixed( 2 )
              + ','+ translation.y.toFixed( 2 )
              + ','+ translation.z.toFixed( 2 ) );

            this.emit('transform.translate.complete', {
              externalId: externalId,
              translation: translation
            })
          });
        }
      }
    }

    this._isDragging = false

    if (this._transformControlTx.onPointerUp(event))
      return true

    return false
  }
```

This is much nicer and cleaner than my previous implementation, because I have not touched any of the other functionality or functions of the translation tool at all.
To pass on the information from the tool to the wide outside world via a `socket.io` broadcast in the viewer extension, I just added a couple of lines to its constructor:

```
class TransformExtension extends ExtensionBase {

  /////////////////////////////////////////////////////////////////
  // Class constructor
  //
  /////////////////////////////////////////////////////////////////
  constructor (viewer, options) {

    super (viewer, options)

    this.translateTool = new TranslateTool(viewer)

    this._viewer.toolController.registerTool(
      this.translateTool)

    this.rotateTool = new RotateTool(viewer)

    this._viewer.toolController.registerTool(
      this.rotateTool)

    this.translateTool.on('transform.translate.complete', (data) => {

      console.log( 'broadcast transform of '
        + data.externalId
        + ': ' + data.translation.x.toFixed( 2 )
        + ','+ data.translation.y.toFixed( 2 )
        + ','+ data.translation.z.toFixed( 2 ) );

      var socketSvc = ServiceManager.getService('SocketSvc')

      // external id == Revit UniqueId
      // THREE.Vector3 offset x y z

      socketSvc.broadcast('transform', data)

    })
  }

  // . . .
```

So things are going swimmingly on this front.
I still have a bunch of other stuff to implement, test and document, though, before I can go off on my vacation next week feeling well prepared for
the [RTC Revit Technology Conference Europe](http://www.rtcevents.com/rtc2016eur) in Porto the week after that:
In fact, I have to go over my whole suite of samples connecting the desktop and the cloud, each consisting of a C# .NET Revit API desktop add-in and a web server:
- [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) and the [roomeditdb](https://github.com/jeremytammik/roomedit) CouchDB
database and web server demonstrating real-time round-trip graphical editing of furniture family instance location and rotation plus textual editing of element properties in a simplified 2D representation of the 3D BIM.
- [FireRatingCloud](https://github.com/jeremytammik/FireRatingCloud) and
the [fireratingdb](https://github.com/jeremytammik/firerating) node.js
MongoDB web server demonstrating real-time round-trip editing of Revit element shared parameter values
– [Heroku](https://heroku.com) requires an update to MongoDB 3.2.
- Update the [Roomedit3dApp](https://github.com/jeremytammik/Roomedit3dApp) Revit add-in to work with the
new [roomedit3dv3](https://github.com/Autodesk-Forge/forge-boilers.nodejs/tree/roomedit3d) Forge Viewer extension demonstrating translation of BIM element instances in the viewer and updating the Revit model in real time via a `socket.io` broadcast.
For the latter, I need to set up a production environment and deploy to Heroku.
So far, I have just been developing and testing in a `DEV` environment on the local machine.
It is rather complicated to connect to that from the virtual Windows machine running Revit and the add-in, though, so better to move to the real Internet and `PROD` first.
Wish me luck with the next steps.