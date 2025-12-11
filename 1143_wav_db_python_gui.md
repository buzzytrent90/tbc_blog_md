---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: code_example
optimization_date: '2025-12-11T11:44:15.393768'
original_url: https://thebuildingcoder.typepad.com/blog/1143_wav_db_python_gui.html
post_number: '1143'
reading_time_minutes: 4
series: general
slug: wav_db_python_gui
source_file: 1143_wav_db_python_gui.htm
tags:
- python
- revit-api
- views
title: WAV Database, Python and GUI Tutorials
word_count: 727
---

### WAV Database, Python and GUI Tutorials

My son Christopher is thinking of creating a music database for his personal use supporting some strict requirements that other databases do not fulfil.

That prompted me to jot down some advice and a couple of starting points.

The initial requirements are simple.

**Question:** I am wondering if I should build my own sample management software since it seems to be quite difficult to find anything useful...

My wishes:

- Build a database of sounds and tags, categories, comments etc. for WAV files
- Manage several collections (SoundEffects, Fieldrecordings, Music, Interviews)
- Search filenames, tags etc. in certain collection or across all collections
- Copy file from original location to other location
- Export database to file (text?)

One more (advanced) feature would be:

- Read embedded Metadata, e.g. INFO and BEXT Chunks in the wave files head

Appendix B in the guidelines on
[Embedding Metadata in Digital Audio Files](http://www.digitizationguidelines.gov/audio-visual/documents/Embed_Intro_090915.pdf)
might be interesting.

But I still have to figure out how this actually looks in the file itself.

I tested writing some info into a wave file and it shows up somwhere at the beginning (looking at it in notepad++).

**Answer:** This should not be too hard.

I love the NoSQL concept and have some small experience using CouchDB.

That would easily do everything you need, and on a web server, as well.

Mediainfo on my mac reads metadata from a WAV file, for instance the title in this case:

```
/downloads/mh/sailor/ $ mediainfo *wav
General
Complete name                            : 3-26 Sailor With the Navy Blue Eyes.wav
Format                                   : Wave
File size                                : 27.1 MiB
Duration                                 : 2mn 40s
Overall bit rate mode                    : Constant
Overall bit rate                         : 1 411 Kbps
Track name                               : Sailor With the Navy Blue Eyes

Audio
Format                                   : PCM
Format settings, Endianness              : Little
Format settings, Sign                    : Signed
Codec ID                                 : 1
Duration                                 : 2mn 40s
Bit rate mode                            : Constant
Bit rate                                 : 1 411.2 Kbps
Channel(s)                               : 2 channels
Sampling rate                            : 44.1 KHz
Bit depth                                : 16 bits
Stream size                              : 27.1 MiB (100%)

/downloads/mh/sailor/ $ mediainfo --Version
MediaInfo Command line,
MediaInfoLib - v0.7.62
```

I use
[mutagen](https://code.google.com/p/mutagen) for
all my
[music file analysis needs](http://thebuildingcoder.typepad.com/blog/2013/06/super-insane-mp3-and-songbird-playlist-exporter.html#6).

**Response:** Mutagen only supports id3 tags, which are not supported in WAV.
The Python
[chunk library](https://docs.python.org/2/library/chunk.html) might
be the answer.

I have to look into analysing wave chunks some more...

Here is what mediainfo tells me about my file:

```
Allgemein
Complete name                            : F:\Library\_TestFiles\2014_03_20_Nora-Tammik\ZOOM0001\ZOOM0001_LR.WAV
Format                                   : Wave
File size                                : 20,6 MiB
Duration                                 : 1min 15s
Overall bit rate mode                    : konstant
Overall bit rate                         : 2 309 Kbps
Producer                                 : ZOOM Handy Recorder H6
Description                              : sPROJECT= / sSCENE=1 / sTAKE=1 / sNOTE=nora tammik, first visit, 3 weeks old, moan, press, fart,
Encoded date                             : 2014-04-20 14:08:39
Encoding settings                        : A=PCM,F=48000,W=24,M=stereo,T=ZOOM Handy Recorder H6 MS S: +1

Audio
ID                                       : 0
Format                                   : PCM
Format settings, Endianness              : Little
Codec ID                                 : 1
Duration                                 : 1min 15s
Bit rate mode                            : konstant
Bit rate                                 : 2 304 Kbps
Channel(s)                               : 2 Kanäle
Sampling rate                            : 48,0 KHz
Bit depth                                : 24 bits
Stream size                              : 20,6 MiB (100%)
```

The interesting aspects are:

- I don't have to analyse anything.
- It tells me everything I want to know.
- The description contains data I entered in a program that edits the wave info chunks I want to read out.
- The file also tells me the length, size, sampling rate, bit depth, file size, path and filename.

How should I start going about this?

I guess I need to build a GUI, have a database and use some libraries handling the file readout and playback?

I'm not quite sure where to start.

The only things I have written so far are small command line tools and calculators.

But I would like to do some GUI.

**Answer:** Yes, the chunk library looks perfect.

I would start by working through a quick Python tutorial, followed by one of the GUI packages tutorial.
Tkinter is the most global one, supported on all platforms, built into Python:

- [Learn Python the hard way](http://learnpythonthehardway.org)
- [Python GUI programming](http://www.tutorialspoint.com/python/python_gui_programming.htm)