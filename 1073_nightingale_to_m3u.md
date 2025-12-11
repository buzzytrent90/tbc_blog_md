---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: code_example
optimization_date: '2025-12-11T11:44:15.247792'
original_url: https://thebuildingcoder.typepad.com/blog/1073_nightingale_to_m3u.html
post_number: '1073'
reading_time_minutes: 3
series: general
slug: nightingale_to_m3u
source_file: 1073_nightingale_to_m3u.htm
tags:
- elements
- python
- revit-api
title: Nightingale M3U and Denormalized Filename Characters
word_count: 576
---

### Nightingale M3U and Denormalized Filename Characters

### Converting Nightingale Playlist to M3U and Handling Mac Denormalized Filename Characters

Here is a post addressing a Mac denormalized filename character issue, music players on Mac and Python file manipulation, so nothing to do with the Revit API.

It appears that Songbird is dead, Nightingale is its successor, and nothing much else is happening on that front, neither now nor ever.

I guess Nightingale will die a natural death too one of these days, but I am still using it so far anyway.

Just like Songbird, Nightingale does not provide any facility to export its playlists in M3U format, or any other way either, for that matter.

The only way that I am aware of to access the playlist data is to copy and paste it through the user interface.

That returns a list of (Artist, Album, Title) triples.

I presented a Python script to
[generate M3U from Songbird playlist copy and paste](http://thebuildingcoder.typepad.com/blog/2013/02/mp3-manipulation-using-python-mutagen-and-ffmpeg.html#3) data,
later updating it to
[support FLAC as well as MP3](http://thebuildingcoder.typepad.com/blog/2013/06/super-insane-mp3-and-songbird-playlist-exporter.html#6).

It works perfectly well for Nightingale also, completely unchanged.

However, it always had a problem on some filenames with simple non-ASCII characters that no amount of twiddling would fix.

I finally found an explanation for that in this thread discussing why
[Python's glob module and Unix' find command don't recognize non-ASCII](http://stackoverflow.com/questions/14185114/pythons-glob-module-and-unix-find-command-dont-recognize-non-ascii):
"Mac OS X uses denormalized characters always for filenames on HFS+. Use `unicodedata.normalize('NFD', pattern)` to denormalize the glob pattern."

I tried it out, and it does indeed fix the problem.

Here is my updated Python script to convert the (Artist, Album, Title) triples to valid local music track filenames to generate a valid M3U playlist:

```
#!/usr/bin/env python
#
# songbird_to_m3u.py - nightingale playlist to m3u
#
# Convert the file information copied and pasted from
# a songbird or nightingale playlist to an m3u playlist.
#
# Jeremy Tammik, Autodesk Inc., 2013-05-12
#
# Artist, Album, Title -->
# /m/Artist/Album/[Track]*Title.flac
# /m/Artist/Album/[Track]*Title.mp3
#
# cat nightingale_export.txt | songbird_to_m3u.py > playlist.m3u
#
import glob, os, sys, unicodedata

# filenames should be in the system encoding:
#fse = sys.getfilesystemencoding()

nOk = 0
nFailed = 0

while True:
  try: line = raw_input().decode("utf-8")
  except: break

  #print '>', line

  a = line.strip().split( ', ' )

  if 3 != len(a):

    print '#', line, ' - not 3 elements'
    sys.stderr.write( line + ' - not 3 elements\n' )
    nFailed += 1
    continue

  ok = False
  p = u'/m/' + a[0] + u'/' + a[1] + u'/*' + a[2]

  # Mac OS X uses denormalized characters always for
  # filenames on HFS+. Use
  #   unicodedata.normalize('NFD', pattern)
  # to denormalize the glob pattern.
  # http://stackoverflow.com/questions/14185114/pythons-glob-module-and-unix-find-command-dont-recognize-non-ascii

  p = unicodedata.normalize('NFD', p)

  a1 = glob.glob( p + '.flac' )
  if 1 == len(a1):
    print a1[0]
    ok = True
  else:
    a2 = glob.glob( p + '.mp3' )
    if 1 == len(a2):
      print a2[0]
      ok = True

  if ok:
    nOk += 1
  else:
    print '#', p, '?'
    sys.stderr.write(
      '\n%s\n%s\nglob returned %d (flac) and %d (mp3)\n\n'
      % (line, p, len(a1), len(a2)) )
    nFailed += 1

sys.stderr.write(
  '%s files passed, %s failed.\n'
  % (nOk, nFailed) )
```

Et voila!
That's all there is to it.