---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.4
content_type: documentation
optimization_date: '2025-12-11T11:44:16.692784'
original_url: https://thebuildingcoder.typepad.com/blog/1761_room_volume_gltf.html
post_number: '1761'
reading_time_minutes: 1
series: general
slug: room_volume_gltf
source_file: 1761_room_volume_gltf.md
tags:
- elements
- revit-api
- rooms
- sheets
- views
title: Room Volume Gltf
word_count: 273
---

### Room Volume glTF Generator
I travelled home last night from the Barcelona Forge accelerator and continued working on
the [room volume exporter](https://thebuildingcoder.typepad.com/blog/2019/06/improved-room-closed-shell-directshape-for-forge-viewer.html).
As suggested by [Michael Beale](https://forge.autodesk.com/author/michael-beale),
I now implemented support
for [glTF, the GL Transmission Format](https://en.wikipedia.org/wiki/GlTF).
As described yesterday, I generate generic model `DirectShape` elements to represent the room volume in the Forge viewer.
I initially generated them using solids returned by the Revit API `GetClosedShell` method.
However, these not work properly in the Forge viewer.
Therefore, I implemented a triangulation of those solids and generate new ones from that.
They work fine.
While I have the triangulation at hand, it is easy to also generate data for glTF.
That is now implemented in
[RoomVolumeDirectShape](https://github.com/jeremytammik/RoomVolumeDirectShape)
[release 2020.0.0.8](https://github.com/jeremytammik/RoomVolumeDirectShape/releases/tag/2020.0.0.8).
It still needs some final tweaks to feed it straight into a glTF viewer, e.g.,
[magicien's GLTFQuickLook](https://github.com/magicien/GLTFQuickLook) for Mac,
but we're getting there.
A few more little items to wrap up the Barcelona topic:
- Kean Walmsley's report on [this year's Forge Accelerator in Barcelona](https://www.keanw.com/2019/06/this-years-forge-accelerator-in-barcelona.html)
- My two favourite restaurants in Barcelona, both in Poblenou, 20 minutes' walk from the Autodesk office:
- [Fish – Restaurant Els Pescadors](http://www.elspescadors.com)
- [Vegetarian – Aguaribay](http://www.aguaribay-bcn.com)
![Forge team lunch on the beach](img/forge_team_lunch.jpg)