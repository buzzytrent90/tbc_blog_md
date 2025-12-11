---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.1
content_type: code_example
optimization_date: '2025-12-11T11:44:14.815167'
original_url: https://thebuildingcoder.typepad.com/blog/0895_mutagen_ffmpeg.html
post_number: 0895
reading_time_minutes: 11
series: general
slug: mutagen_ffmpeg
source_file: 0895_mutagen_ffmpeg.htm
tags:
- elements
- family
- levels
- python
- revit-api
- rooms
- views
title: MP3 Manipulation Using Python, Mutagen and Ffmpeg
word_count: 2226
---

### MP3 Manipulation Using Python, Mutagen and Ffmpeg

Today is an education day, and I am taking another look at displaying a 2D view of a Revit model on mobile devices using SVG.

I started off doing so quite a while back, implementing a
[room polygon and furniture picker in SVG](http://thebuildingcoder.typepad.com/blog/2012/10/room-polygon-and-furniture-picker-in-svg.html).

That implementation displays a read-only view of the model, useful for picking and identifying elements, e.g. for querying or adding metadata to them.

Today I plan to go one step further, though, and enable translation and rotation of the furniture and other family instances.

To do so, I think it might be helpful to encapsulate the SVG interaction using a slightly higher-level toolkit such as the
[Raphaël JavaScript library](http://raphaeljs.com/).

I'll let you know how I do next week.

Right now, though, I'll first mention some completely different topics that I played with last week-end, completely unrelated to the Revit API, but still of a technical nature, related to MP3 music file tag manipulation and analysis:

- [Globally swapping MP3 artist and album artist tags](#2)
- [Extracting M3U playlist information from Songbird](#3)
- [Comparing erroneous Mutagen MP3 track duration with ffmpeg](#4)

The reason I mention them here is that I had to spend some time getting these scripts to work.
During my research, I saw that many others have encountered similar issues, and thought it worthwhile sharing my solutions with you.

#### Globally Swapping MP3 Artist and Album Artist Tags

I started DJ-ing in the past few years and put together a largish collection of music for that.

I obviously try to keep it clean and organised, storing all files in MP3, avoiding other strange partially unsupported formats, using open source tools to play an manipulate the tracks etc.

When I initially started, I was somewhat bewildered by the various tags, and ended up depending mainly on the artist, album and title triple to organise everything and also define the directory structure in which the physical files are stored.

A while back, I gradually discovered that actually, the album artist tag is more suited for this purpose.

Often, an album is produced with one single album artist, and each track on the album features additional guest artists.
I obviously do not want to create separate top-level folders for each of these tracks.

Therefore, I was faced with the problem of switching my entire tagging structure to use album artist instead of artist to define the directory structure.

In my collection, the contents of these two tags are identical in many cases, in which case the issue is trivial.

In other tracks, the album artist has not yet been defined, which also makes it easy to fix, since all it requires is copying the artist to the album artist.

If both have been defined and differ, however, I swap their contents.

I use
[Python](http://python.org) and
[Mutagen](http://code.google.com/p/mutagen).
The latter includes a helpful short
[tutorial](http://code.google.com/p/mutagen/wiki/Tutorial).
I also found the introduction on
[writing ID3 tags using Python and Mutagen for dummies](http://mydl.itweb.co.za/index.php?option=com_myblog&show=1046123&Itemid=29) very
useful.

Here is the resulting Python script to achieve this (copy to a text editor or view source to see the truncated lines in full):

```python
#!/usr/bin/python
#
# mp3swapartist.py
#
# swap mp3 artist and album artist tags
#
# I set up my whole music collection using a directory structure
# based on artist/album/trackno - title. Now I realise that
# album artist is the proper tag to use for that structure.
#
# Copyright (C) 2013 Jeremy Tammik
#
import glob, os, re, sys
from mutagen.mp3 import MP3
from mutagen.id3 import TPE1, TPE2
def swap\_artist( filepath ):
"Swap mp3 artist and album artist tags."
assert filepath.lower().endswith( '.mp3' )
try:
audio = MP3( filepath )
old\_artist = unicode( audio['TPE1'] )
assert 0 < len( old\_artist )
old\_album\_artist = ''
if audio.has\_key( 'TPE2' ):
old\_album\_artist = unicode( audio['TPE2'] )
s = "original artist '%s', album artist '%s'" % (old\_artist, old\_album\_artist)
if 0 == len( old\_album\_artist ):
# copy artist to album artist
s += ' - added album artist'
audio.tags.add( TPE2( encoding=3, text=old\_artist ) )
audio.tags.save()
elif old\_artist != old\_album\_artist:
# swap artist and album artist
s += ' swapped'
audio.tags.add( TPE2( encoding=3, text=old\_artist ) )
audio.tags.add( TPE1( encoding=3, text=old\_album\_artist ) )
audio.tags.save()
else:
s += ' retained'
print filepath + ':', s
except StandardError, err:
print 'Error:', str( err ), "in '%s'" % filepath
def main():
"Walk a directory tree and swap all mp3 artist and album artist tags."
dir = '.'
for root, dirs, files in os.walk( dir ):
for filename in files:
if filename.lower().endswith( '.mp3' ):
filepath = os.path.join( root, filename )
swap\_artist( filepath )
if \_\_name\_\_ == "\_\_main\_\_":
main()
```

It worked perfectly and instantaneously on a 40 GB collection of over 6000 tracks.

Note the use of the audio.tags.save method instead of audio.save, as recommended by
[mandibleclaw](http://mydl.itweb.co.za/index.php?option=com_myblog&blogger=mandibleclaw&Itemid=).

#### Extracting M3U Playlist Information from Songbird

One of the players I use to listen to music is
[Songbird](http://getsongbird.com).

It includes a number of pretty neat features.
One of the ones I like best is the search function, which picks up the target string from absolutely everywhere, including in comment fields etc.

It is also completely incomprehensibly utterly lacking in some other areas.
For example, although it will import an M3U playlist, there is no way to export one.
Purportedly, add-ins exist for doing so, but in spite of extensive searching I have not been able to find and install one that works for my up-to-date version 2.1.0.

I would like to extract the full path name and duration of tracks, as seen here in the Songbird user interface:

![Songbird Playlist](img/mp3_songbird.png)

The only way I found so far to access any of the playlist data at all is to select all files and copy to the clipboard.
That produces text containing a comma-separated list of the artist, album and track tag contents.
Of course, I implemented a python script to convert that to the actual file names :-)

It basically just puts together a slash-separated file path from the comma-separated data provided and uses the Python glob module to find out what the track number is, which may or may not be included in the path:

```
#!/usr/bin/env python
#
# songbird_to_m3u.py - convert the file tags exported by songbird to m3u playlist
#
# Artist, Album, Title -->
# /m/Artist/Album/Track*Title.mp3
#
# cat songbird_export.txt | songbird_to_m3u.py > songbird_export.m3u
#
import glob, os, sys

nOk = 0
nFailed = 0

while True:
  try: line = raw_input()
  except: break

  a = line.split( ', ' )

  if 3 != len(a):
    sys.stderr.write( line + '\n' )
    nFailed += 1
    continue

  p = '/m/' + a[0] + '/' + a[1] + '/*' + a[2] + '.mp3'
  a = glob.glob( p )

  if 1 == len(a):
    print a[0]
    nOk += 1
  else:
    sys.stderr.write( line + '\n' )
    nFailed += 1

sys.stderr.write( '%s files passed, %s failed.\n' % (nOk, nFailed) )
```

The '/m/' prefix is a link that I created in the file system root folder to my user-specific music folder:

```
lrwxr-xr-x 1 root wheel 21 Nov 14 19:42 m -> /Users/tammikj/Music/
```

Unix is so good.

#### Comparing Erroneous Mutagen MP3 Track Duration with Ffmpeg

As I mentioned above, I would like to determine the duration of a track as well as retrieving its absolute path to populate a M3U playlist.

Happily, Mutagen provides this feature; it is immediately available via

```
  audio = MP3( path )
  seconds = audio.info.length
```

Unfortunately, the time reported by Mutagen is sometimes blatantly wrong.

Researching this on the Internet, I discover that the problem is well known.

One suggestion was to use the incredibly powerful conversion package
[ffmpeg](http://www.ffmpeg.org) instead,
and that is what I ended up doing.

This involves launching the ffmpeg executable to analyse each track in turn and capture the output it produces, which is printed to the standard error stream stderr instead of the standard output stream stdout.

I can use the Python subprocess.check\_output method to launch the command and retrieve its output.
Since ffmpeg returns with a non-zero error code and the check\_output method traps that, I had to append a call to 'exit 0' to cancel the non-zero error code returned by ffmpeg.

I use a regular expression to extract the duration value from the ffmpeg stderr output, and the Python strip method to remove the leading zeroes and colons from that.

Here is a Python script to read a playlist, query and list both the Mutagen and ffmpeg duration values:

```
#!/usr/bin/python
#
# mp3duration.py - retrieve the length of the tracks in the playlist and calculate the total
#
import glob, os, re, subprocess, sys
from mutagen.mp3 import MP3

_find_duration = re.compile( '.*Duration: ([0-9:]+)', re.MULTILINE )

def min_sec_to_seconds( ms ):
  "Convert a minutes:seconds string representation to the appropriate time in seconds."
  a = ms.split(':')
  assert 2 == len( a )
  return float(a[0]) * 60 + float(a[1])

def seconds_to_min_sec( secs ):
  "Return a minutes:seconds string representation of the given number of seconds."
  mins = int(secs) / 60
  secs = int(secs - (mins * 60))
  return "%d:%02d" % (mins, secs)

def retrieve_length( playlist_filename ):
  "Determine length of tracks listed in the given input files (e.g. playlists)."

  print playlist_filename + ' duration:'

  if not os.path.exists( playlist_filename ):
    print "Error: specified playlist '%s' does not exist.\n" % playlist_filename
    raise SystemExit(1)

  f = open( playlist_filename )
  lines = f.readlines()
  f.close()

  total_mutagen = 0.0
  total_ffmpeg = 0.0

  print '%8s%8s%8s  %s' % ('mutagen', 'm:s', 'ffmpeg', 'track')

  for line in lines:
    path = line.strip()

    if not path or path[0] == '#':
      continue

    if not os.path.exists( path ):
      print "Error: specified music file '%s' does not exist.\n" % path
      raise SystemExit(2)

    audio = MP3( path )
    seconds = audio.info.length

    ffmpeg = subprocess.check_output(
      'ffmpeg -i "%s"; exit 0' % path,
      shell = True,
      stderr = subprocess.STDOUT )

    match = _find_duration.search( ffmpeg )
    if match: ffmpeg = match.group( 1 )
    else: ffmpeg = '--'

    ffmpeg = ffmpeg.lstrip('0:')

    print '%8.1f%8s%8s  %s' % (seconds, seconds_to_min_sec(seconds), ffmpeg, path )

    total_mutagen += seconds
    total_ffmpeg += min_sec_to_seconds( ffmpeg )

  s = '-' * 6
  print '%8s%8s%8s  %s' % (s, s, s, s )
  print '%8.1f%8s%8s  %s' % (total_mutagen, seconds_to_min_sec(total_mutagen), seconds_to_min_sec(total_ffmpeg), 'total' )

def main():
  "Determine length of tracks listed in the given input files (e.g. playlists)."

  for pattern in sys.argv[1:]:
    filelist = glob.glob( pattern )
    for filename in filelist:
      retrieve_length( filename )

if __name__ == '__main__':
  main()
```

Here is the result of running this script on the playlist shown above in Songbird:

```
LerchenWave-2013-02.m3u duration:
 mutagen     m:s  ffmpeg  track
   282.2    4:42    4:42  /m/Hildegard Von Bingen/Illumination/04 - Red River Falling.mp3
   305.8    5:05    5:05  /m/Beats Antique/Blind Threshold/02 Runaway.mp3
   129.5    2:09    2:09  /m/Hugues Le Bars/Musiques pour Versailles/12 - Lausann's Blues.mp3
  1121.4   18:41    8:08  /m/DJ Vish/Goa/09 Titel 09.mp3
   217.2    3:37    3:37  /m/Gontiti/In the Garden/07 - Kurt's Stroll Through Town.mp3
   272.0    4:32    4:36  /m/Beats Antique/Contraption/03 - Junktion.mp3
   356.8    5:56    5:56  /m/Red Fulka/We Are One/02 Disco Shamans.mp3
   447.1    7:27    3:14  /m/Stray Cats/The Very Best Of/03 Stray Cat Strut.mp3
   292.2    4:52    4:51  /m/Dunkelbunt/Morgenlandfahrt/14 Istanbul 1.26 AM ft. Orient Expressions (dunkelbunt remix).mp3
   218.6    3:38    3:38  /m/Wax Tailor/Electro Swing 2/01 - Say Yes.mp3
   806.2   13:26    5:51  /m/The Chemical Brothers/Further/05 Horse Power.mp3
   165.1    2:45    2:45  /m/Uri Caine/Rio/12 - Samba da Terra.mp3
   345.6    5:45    2:30  /m/Spike Jones/Spike Jones Goes Crazy/13 Black Bottom.mp3
   394.6    6:34    2:51  /m/Ländlerkapelle Carlo Brunner/Urichigi Tänzli/03 Grüezi Wohl Frau Stirnimaa.mp3
   583.7    9:43    4:14  /m/Giora Feidman/The Dance Of Joy/02 Rue Du Bac.mp3
   436.6    7:16    3:10  /m/Michael Nyman/Man on Wire/3 Gymnopédies Gymnopédie No 1.mp3
  1036.4   17:16    7:31  /m/Büdi Siebert/Namaste/04 Lotus Call 1 Varja Guru Mantra.mp3
  ------  ------  ------  ------
  7411.2  123:31   74:48  total
```

As you can see, out of a total of 17 tracks, the duration is reported identical for 7 and differs significantly for 8.
One track is off by just a single second, and another by four.
Ffmpeg produces the exact same times as Songbird does on all except the track that is off by four seconds, where Mutagen is in agreement with Songbird instead, surprisingly enough.

Rather strange, isn't it?
But interesting.

Anyway, I hope that this is of use to you if you ever run into similar issues, or even if you just want to see a couple of examples of using these tools for simple MP3 manipulation and analysis tasks.

And now back again to SVG, and ultimately, the Revit API...