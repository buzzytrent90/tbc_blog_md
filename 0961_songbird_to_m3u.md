---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.3
content_type: code_example
optimization_date: '2025-12-11T11:44:14.997581'
original_url: https://thebuildingcoder.typepad.com/blog/0961_songbird_to_m3u.html
post_number: 0961
reading_time_minutes: 14
series: transactions
slug: songbird_to_m3u
source_file: 0961_songbird_to_m3u.htm
tags:
- elements
- levels
- python
- revit-api
- views
- transactions
title: Super Insane MP3 and Songbird Playlist Exporter
word_count: 2779
---

﻿

### Super Insane MP3 and Songbird Playlist Exporter

I am still in the USA, visiting my colleague Kevin Vandecar in Goffstown, New Hampshire, returning to Europe on Tuesday.

Today we worked in his huge garden, loading a bunch of tree stumps on a truck with a tractor, then went for a swim in the beautiful Uncanoonuc Lake:

![Uncanoonuc Lake](file:////j/photo/jeremy/2013/2013-06-08_goffstown/05.jpeg)

Back in February I mentioned using Songbird to play music on the Mac,
[extracting M3U playlists](http://thebuildingcoder.typepad.com/blog/2013/02/mp3-manipulation-using-python-mutagen-and-ffmpeg.html#3) from
it, and
[determining the playlist duration](http://thebuildingcoder.typepad.com/blog/2013/02/mp3-manipulation-using-python-mutagen-and-ffmpeg.html#4) using ffmpeg and mutagen.

I encountered lots of other digital music related issues since then, and here are some notes on a few of them:

- [MP3 versus FLAC](#2)
- [CBR versus VBR](#3)
- [Insanely high MP3 quality is insufficient](#4)
- [Converting M4A to MP3](#5)
- [Exporting a Songbird playlist to M3U](#6)
- [MP3 track and M3U playlist duration](#7)

### MP3 Versus FLAC

Here is a discussion of a topic important to me but completely unconnected to Revit and its API.
I hope it is of use to other audiophiles as well.
In fact, I would never dream of terming myself audiophile, and I seems to me that it should be of great interest to anybody using the MP3 format, and probably any other digital music format as well.
As far as I can tell, that incudes just about everybody I know, and definitely everybody under 25 years of age.

I mentioned that I dabble with DJ-ing and listen to music, which is vastly more handy to manage in digital format, of course.

Until quite recently, I was storing all my music in the lossy compressed
[MP3](http://en.wikipedia.org/wiki/MP3) format.

I recently had a bad shock connected with that, though.

My friend Markus gave me a nice jazzy guitar and percussion recording in the lossless
[FLAC](http://en.wikipedia.org/wiki/FLAC) format.

I converted it to MP3 using the standard settings, and was surprised and pretty devastated to find that I could hear a significant difference in quality, just listening to the first few seconds of the original FLAC and the compressed MP3 on the built-in Mac loudspeakers.

This led to a visit to Markus, who stores all his music in FLAC and has the appropriate equipment to listen to it in high quality as well.
Further experiments showed that I need to either abandon the MP3 format entirely for any high quality sound archival and listening, or pay a lot more attention to details that I had hitherto completely ignored.

I seriously considered giving up on the MP3 format altogether, which would be painful, since so far I have been converting everything I have to that format, and standardising all my tools to expect and handle that and nothing else, significantly simplifying scripts for creating and analysing playlists and such-like things.

On the other hand, the MP3 format does include options for handling levels of quality that are said to make the compressed version indistinguishable from the original, so I had hopes of making use of them to allow me to happily continue using this format after all.

It was harder than expected, though.

Below are a few things I learned on the way.

I would also like to recommend this old article from 2005 on
[variable bit rate and getting the best bang for your byte](http://www.codinghorror.com/blog/2005/12/variable-bit-rate-getting-the-best-bang-for-your-byte.html),
including the numerous comments providing some interesting suggestions and lots of background information.

#### CBR versus VBR

As far as I can tell, quite a number of MP3 files use constant bitrate encoding, CBR.

Many MP3 files are encoded using
[LAME](http://en.wikipedia.org/wiki/LAME).

Its documentation states that CBR is discouraged, and variable bitrate,
[VBR](http://lame.sourceforge.net/vbr.php), should be used instead.

I tried directly using lame for music conversion, but it does not preserve the metadata tags, so I returned to ffmpeg instead anyway.

Still, reading the lame documentation helps understand which of the multitude of ffmpeg tags to use to control the conversion quality, and how.

#### Insanely High MP3 Quality is Insufficient

Armed with these experiences, I tried converting from FLAC to MP3 using the lame -V 0 quality setting, which is termed 'insanely high quality' in the lame documentation.

I still hear a deterioration of clarity and brilliance, though.

This led me to look for further possibilities to force lame to globally maintain a specified minimum rate.

Happily, such a setting does exist using the -b option, and using it to request a minimum bitrate of 256 kbps finally succeeds in producing an output that I am unable to distinguish from the original.

#### Converting M4A to MP3

Later, I noticed a deterioration converting certain M4A files to MP3.

I still have no perfect solution for this.

Some suggestions for LAME encoding arguments that I tried include:

```
  -V9 --vbr-new -q0 -mj -b32 -F --lowpass 19.7
    --nspsytune --cwlimit 10.7 --athaa-sensitivity 1

  -V2 --vbr-new -q0 --lowpass 19.7 -b96

  -V2 --vbr-new -q0 --lowpass 19.7 --cwlimit 10.7
    --scale 0.99 -b96
```

My current solution includes comparing the original M4A file size with the resulting MP3 size, and looking at the bit rate and mode using
[mediainfo](http://mediainfo.sourceforge.net).

Here is the `m4a2mp3` script that I am currently using, and often re-adapting:

```
#!/bin/bash

# save and change IFS to allow spaces in filenames
OLDIFS=$IFS
IFS=$'\n'

#for f in *.m4a ; do ffmpeg -i "$f" "$f".mp3 ; done
#for f in *.m4a ; do ffmpeg -i "$f" -q:a 0 $(basename -s .m4a "$f").mp3 ; done
#for f in *.m4a ; do ffmpeg -i "$f" -b:a 320k $(basename -s .m4a "$f").mp3 ; done
#for f in *.m4a ; do ffmpeg -i "$f" -ab 300 $(basename -s .m4a "$f").mp3 ; done
#for f in *.m4a ; do ffmpeg -i "$f" -q 0 $(basename -s .m4a "$f").mp3 ; done

for f in *.m4a ; do ffmpeg -i "$f" -q 2 $(basename -s .m4a "$f").mp3 ; done

# this does not preserve the tags:
#for f in *.m4a ; do faad -o - "$f" | lame -V0 -b 256 - "${f%m4a}mp3" ; done

# restore IFS
IFS=$OLDIFS
```

Copy and paste the above to an editor or view the browser source code to see the truncated lines in full.

In the last few runs, I have simply changed the '-q 2' argument up or down a bit so that the resulting MP3 quality matches well with the original M4A.

#### Exporting a Songbird Playlist to M3U

I presented my Songbird
[M3U playlist extractor](http://thebuildingcoder.typepad.com/blog/2013/02/mp3-manipulation-using-python-mutagen-and-ffmpeg.html#3) when
I was still supporting only MP3 file format.
I enhanced it to support FLAC as well, and now present the updated version.

I sometime use
[VirtualDJ](http://www.virtualdj.com) to play music to dance to, since it provides the crossfade functionality lacking in Songbird.

I can feed it with an M3U playlist.

But...

The biggest gripe I have about Songbird is that it will not export its playlists to any sensible file format at all, in spite of being able to import M3U.

I searched for workarounds and saw many mentions of plugins for achieving this, but none of them worked for me.

There appears to be a lot of confusion among Songbird and its plugins regarding version compatibility.

I found a pretty easy workaround of my own, though, which I have been using successfully for some time now.

I can simply select all songs in a Songbird playlist and hit Cmd-C to copy them to the pasteboard.

If you select a single track in Songbird and copy and paste it to an editor, you will see that it places a comma delimited string containing the artist, album and track name on the pasteboard.
Selecting and copying several tracks generates a linefeed-delimited list of these records, i.e. each track information on a separate line.

This information can be used to generate an M3U playlist file, provided the actual file path structure exactly matches the artist, album and track information read from the MP3 or FLAC music file tags.

Once that is all set up, I can use the following Python script `songbird_to_m3u.py` to convert the information copied and pasted from Songbird to an M3U playlist referencing the files in my music library, which I can then import into VirtualDJ or any other player:

```
#!/usr/bin/env python
#
# songbird_to_m3u.py - convert the file tags exported
#                      by songbird to m3u playlist
#
# Jeremy Tammik, Autodesk Inc., 2013-05-12
#
# Artist, Album, Title -->
# /m/Artist/Album/Track*Title.flac
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

  #print '>', line

  a = line.strip().split( ', ' )

  if 3 != len(a):

    sys.stderr.write( line + ' - not 3 elements\n' )
    nFailed += 1
    continue

  ok = False
  p = '/m/' + a[0] + '/' + a[1] + '/*' + a[2]
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
    sys.stderr.write(
      '\n%s\n%s\nglob returned %d (flac) and %d (mp3)\n\n'
      % (line, p, len(a1), len(a2)) )
    nFailed += 1

sys.stderr.write(
  '%s files passed, %s failed.\n'
  % (nOk, nFailed) )
```

Here is the content of the Mac pasteboard after copying a playlist in Songbird:

```
  Gotan Project, La Revancha Del Tango, Chunga's Revenge
  Outback, Dance The Devil Away, Two In The Bush
  Marla Glen, Humanology, Fever
  Brent Lewis, Thunder Down Under, Outback Attack
  The Chemical Brothers, Hanna, Escape 700
  Shakti, Natural Elements, Come On Baby Dance With Me
  Gabrielle Roth, Tribe, Tsunami 3 - chaos
  Talvin Singh, Oriental Club Disk 2, OK
  Martin O, Der mit der Stimme tanzt, Japaner
  Jazzbit, Unknown, Swingin man
  Yma Sumac, Ultra Lounge - Leopard Skin Fuzzy Sampler, Taki Rari
  Folco Orselli, Generi di Conforto, La ballata di Piazzale Maciachini
  Beatles, White Album CD 2, Good Night
  Natalie Dessay, Vocalises, Vocalise
  Büdi Siebert, Namaste, Lotus Call 1 Varja Guru Mantra
  Jeremy Tammik, Meditation, Stillness One Minute Silence
  Erik Truffaz, Mantis, Saisir
```

Running the `songbird_to_m3u.py` script on that produces the following output, representing a valid M3U playlist on my file system:

```
  /m/Gotan Project/La Revancha Del Tango/03 - Chunga's Revenge.mp3
  /m/Outback/Dance The Devil Away/03 - Two In The Bush.mp3
  /m/Marla Glen/Humanology/13 Fever.flac
  /m/Brent Lewis/Thunder Down Under/01 Outback Attack.mp3
  /m/The Chemical Brothers/Hanna/02 Escape 700.mp3
  /m/Shakti/Natural Elements/03 - Come On Baby Dance With Me.mp3
  /m/Gabrielle Roth/Tribe/03 - Tsunami 3 - chaos.mp3
  /m/Talvin Singh/Oriental Club Disk 2/13 - OK.mp3
  /m/Martin O/Der mit der Stimme tanzt/07 - Japaner.mp3
  /m/Jazzbit/Unknown/00 Swingin man.mp3
  /m/Yma Sumac/Ultra Lounge - Leopard Skin Fuzzy Sampler/03 Taki Rari.mp3
  /m/Folco Orselli/Generi di Conforto/04 - La ballata di Piazzale Maciachini.mp3
  /m/Beatles/White Album CD 2/13 - Good Night.mp3
  /m/Natalie Dessay/Vocalises/01 Vocalise.flac
  /m/Büdi Siebert/Namaste/04 Lotus Call 1 Varja Guru Mantra.mp3
  /m/Jeremy Tammik/Meditation/Stillness One Minute Silence.mp3
  /m/Erik Truffaz/Mantis/03 Saisir.flac
```

Copy and paste the above to an editor or view the browser source code to see the truncated lines in full.

#### MP3 Track and M3U Playlist Duration

Once I have an M3U playlist, I would also like to determine the total duration of all the tracks.

I presented my
[mp3duration.py](http://thebuildingcoder.typepad.com/blog/2013/02/mp3-manipulation-using-python-mutagen-and-ffmpeg.html#4) Python
script achieving that, which I now enhanced to handle FLAC files as well.

Here is the updated version:

```
#!/usr/bin/python
#
# mp3duration.py - retrieve the length of the tracks in
#                  the playlist and calculate the total
#
# Jeremy Tammik, Autodesk Inc., 2013-05-12
#
# /j/sh/mp3duration.py playlist.m3u
#
# http://code.google.com/p/mutagen/wiki/Tutorial
# http://www.doughellmann.com/PyMOTW/subprocess
#
import glob, os, re, subprocess, sys
import mutagen.mp3, mutagen.flac

_find_duration = re.compile( '.*Duration: ([0-9:]+)', re.MULTILINE )

def min_sec_to_seconds( ms ):
  "Convert a minutes:seconds string representation to the appropriate time in seconds."
  a = ms.split(':')
  assert len( a ) in [1,2]
  if 1 == len(a):
    s = float(a[0])
  else:
    s = float(a[0]) * 60 + float(a[1])
  return s

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

  total_count = 0
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

    if( path.endswith( 'mp3' ) ):
      audio = mutagen.mp3.MP3( path )
    else:
      assert path.endswith( 'flac' )
      audio = mutagen.flac.FLAC( path )

    #print audio.info.length, audio.info.bitrate, path
    seconds = audio.info.length

    ffmpeg = subprocess.check_output(
      'ffmpeg -i "%s"; exit 0' % path,
      shell = True,
      stderr = subprocess.STDOUT )

    #>>> output = subprocess.check_output(
    #...   'echo to stdout; echo to stderr 1>&2; exit 0', shell=True)
    #to stderr
    #>>> print output
    #to stdout

    #print ffmpeg
    #exit( 1 )

    #lines = ffmpeg.split( '\n' )

    #ffmpeg -report -i path 2>&1 | awk '/Duration/{print $2}'

    match = _find_duration.search( ffmpeg )
    if match: ffmpeg = match.group( 1 )
    else: ffmpeg = '--'

    ffmpeg = ffmpeg.lstrip('0:')
    ffmpeg = min_sec_to_seconds( ffmpeg )

    print '%8.1f%8s%8s  %s' % (seconds, seconds_to_min_sec(seconds), seconds_to_min_sec(ffmpeg), path )

    total_count += 1
    total_mutagen += seconds
    total_ffmpeg += ffmpeg

  s = '-' * 6
  print '%8s%8s%8s  %s' % (s, s, s, s )
  print '%8.1f%8s%8s  total for %s tracks' % (total_mutagen, seconds_to_min_sec(total_mutagen), seconds_to_min_sec(total_ffmpeg), total_count )

def main():
  "Determine length of tracks listed in the given input files (e.g. playlists)."

  for pattern in sys.argv[1:]:
    filelist = glob.glob( pattern )
    for filename in filelist:
      retrieve_length( filename )

if __name__ == '__main__':
  main()
```

Here is the output generated by running it on a playlist containing both MP3 and FLAC files:

```
/j/audio/wave/ $ mp3duration.py LerchenWave-2013-05b.m3u
LerchenWave-2013-05b.m3u duration:
 mutagen     m:s  ffmpeg  track
   304.2    5:04    5:04  /m/Gotan Project/La Revancha Del Tango/03 - Chunga's Revenge.mp3
   314.9    5:14    5:14  /m/Outback/Dance The Devil Away/03 - Two In The Bush.mp3
   244.0    4:04    4:04  /m/Marla Glen/Humanology/13 Fever.flac
   315.7    5:15    5:18  /m/Brent Lewis/Thunder Down Under/01 Outback Attack.mp3
   599.5    9:59    4:21  /m/The Chemical Brothers/Hanna/02 Escape 700.mp3
   119.4    1:59    1:59  /m/Shakti/Natural Elements/03 - Come On Baby Dance With Me.mp3
   344.3    5:44    5:44  /m/Gabrielle Roth/Tribe/03 - Tsunami 3 - chaos.mp3
   255.5    4:15    4:15  /m/Talvin Singh/Oriental Club Disk 2/13 - OK.mp3
    56.1    0:56    0:56  /m/Martin O/Der mit der Stimme tanzt/07 - Japaner.mp3
   187.5    3:07    3:07  /m/Jazzbit/Unknown/00 Swingin man.mp3
   112.0    1:51    1:52  /m/Yma Sumac/Ultra Lounge - Leopard Skin Fuzzy Sampler/03 Taki Rari.mp3
   258.1    4:18    4:18  /m/Folco Orselli/Generi di Conforto/04 - La ballata di Piazzale Maciachini.mp3
   191.8    3:11    3:11  /m/Beatles/White Album CD 2/13 - Good Night.mp3
   323.9    5:23    5:23  /m/Natalie Dessay/Vocalises/01 Vocalise.flac
  1036.4   17:16    7:31  /m/Büdi Siebert/Namaste/04 Lotus Call 1 Varja Guru Mantra.mp3
    60.1    1:00    1:00  /m/Jeremy Tammik/Meditation/Stillness One Minute Silence.mp3
   392.3    6:32    6:32  /m/Erik Truffaz/Mantis/03 Saisir.flac
  ------  ------  ------  ------
  5115.7   85:15   69:49  total for 17 tracks
```

That about covers my current digital music status.

I hope you can find some useful snippets in here.

Happy Sunday!