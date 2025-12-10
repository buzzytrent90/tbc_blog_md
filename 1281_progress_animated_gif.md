---
post_number: "1281"
title: "Displaying Progress Bar and Generating Animated GIF"
slug: "progress_animated_gif"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'parameters', 'revit-api', 'vbnet', 'windows']
source_file: "1281_progress_animated_gif.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1281_progress_animated_gif.html"
---

### Displaying Progress Bar and Generating Animated GIF

A frequent request for Revit add-ins performing time-consuming tasks is to display a progress bar.

My own progress bar implementation is provided by the ADN MEP sample application
[AdnRme](https://github.com/jeremytammik/AdnRme),
originally dating way back to
[2009](http://thebuildingcoder.typepad.com/blog/2009/08/mep-sample-ribbon-panel.html) and
earlier still.

Now Limin He of [FM:Systems](http://www.fmsystems.com) encountered
and resolved an issue with his progress bar in VB.NET, raising the following
[comment](http://thebuildingcoder.typepad.com/blog/2014/05/multithreading-throws-exceptions-in-revit-2015.html#comment-6a00e553e16897883301bb07e7bfe3970d) on
the topic of the new
[exceptions raised by multithreading in Revit 2015](http://thebuildingcoder.typepad.com/blog/2014/05/multithreading-throws-exceptions-in-revit-2015.html):

**Question:**
We came across the same issue described above and submitted a ticket in ADN.

I tried the workaround you suggested using Application.DoEvents but the problem is the form has to be modeless which is not what we want.

Updating a Revit parameter in the background while running a Windows form with a progress bar did not crash Revit before 2014.

Any solid workaround for providing status update to the user while calling a time-consuming API at the same time?

**Answer:**
You absolutely cannot safely perform any modification on the Revit database or use the Revit API in any way whatsoever, even for read-only purposes, except in a valid Revit API context;
here is the [final word on that](http://thebuildingcoder.typepad.com/blog/2014/11/the-revit-api-is-never-ever-thread-safe.html).

A valid Revit API context is provided by Revit by a callback function, e.g. execution of an external command or in an event handler.

The workaround is clear and simple: implement an external event.

**Response:**
Thanks for the explanation. But the workaround uses a modeless form, that means user still can make changes through the Revit UI while the .NET API is called – isn't that less protected?

My project is in fact not multi-threaded, it just uses one single thread with Revit running in the background while a modal Windows form updates the status in the front.
I'm surprised that it causes Revit 2015 to crash and works fine in previous versions.

In the end, I figured out a workaround to solve the problem.

Here is the [source code](https://github.com/lotusinriver/MonitorProgressRevit) that I uploaded to
[update a progress bar from a modal dialogue while calling the Revit API](https://github.com/lotusinriver/MonitorProgressRevit).

I hope it helps others using a similar approach to implement this feature as well.

**Answer:**
Your solution caused me some confusion...

For instance, you do not provide an add-in manifest, so I had to write it myself.

The solution name is MonitorProgressRevit, the project name is MonitorProgress.vbproj, and the resulting .NET assembly DLL is MonitorProgess.dll, with one 'r' missing.

It took a few attempts and some time experimenting to figure all this out...

Still working on it, though...

In the end, I got it compiled and tested successfully on Revit 2015.

I created a pull request to update your repository with my changes.

Can you pull it, please?

**Response:**
I merged [your pull request](https://github.com/lotusinriver/MonitorProgressRevit/pull/1) and
made you a project collaborator.

Regarding the description of this repository, I think your blog provides more detailed information about this issue.
The repository readme includes the link of your blog, so the user should be able to get it.

The purpose of my project is to provide a modal dialog with status update when running the Revit API (e.g., to update an element instance parameter), and also the user can cancel the action at any time during the process.

**Answer:**
Here is a 59-second video demonstrating Limin's add-in modifying a Revit element parameter value and displaying the on-going progress in a live progress bar:

![MonitorProgress 3](img/monitor_progress_442_3.gif)

(Actually, this is not the first 77 MB animated GIF that I produced, but the final optimised 3 MB version discussed below).

#### Command Line Creation of an Animated GIF File

As you can see above, I created an animated GIF file to display Limin's active progress bar up and running in all its glory.

In the past, I used YouTube to display similar animations.
The disadvantage of that approach is that it forces you, the user, to click on a button to start the animation running.

The only method to display an animation immediately on a web page with no need for any user interaction whatsoever is the trusty old
[animated GIF](https://en.wikipedia.org/wiki/GIF#Animated_GIF) format,
dating way back to the late 1980's, initially patented, and now freely useable again, since the patents meanwhile expired.

I use Camtasia on Mac for recording. On Windows, Camtasia can generate animated GIF files, but alas, the Mac version lacks this feature.

So I went ahead and set up a Unix based command line tool or two to create one myself.

As far as I can tell, the simplest way to achieve this is to use ffmpeg and gifsicle as described by Sharad Chhetri in
[converting a video file into GIF on the Linux command line](http://sharadchhetri.com/2013/07/10/convert-video-file-into-gif-file-through-command-line-in-linux).

I performed the following steps:

**1.**
I compiled and installed gifsicle from the tar.gz package provided by the
[Gifsicle page on lcdf.org](http://www.lcdf.org/gifsicle).

**2.**
I used the build and installation instructions from the
[Gifsicle GitHub repository](https://github.com/kohler/gifsicle) to
create my binary Mac OS executable.

**3.**
I cloned the [MonitorProgress GitHub project](https://github.com/lotusinriver/MonitorProgressRevit),
compiled and installed it, started Revit and set up a very small Revit window width of only 884 pixels.

**4.** I recorded an MP4 file using Camtasia for Mac, generating the file monitor\_progress.mp4, 13831227 bytes in size.

**5.** I converted the MP4 file to 596 individual GIF images using ffmpeg:

```
$ ffmpeg -i monitor_progress.mp4 -r 10 output/out%04d.gif

$ ls -o output/
total 153416
  161986  out0001.gif
  162589  out0002.gif
  162555  out0003.gif
. . .
  128736  out0594.gif
  128649  out0595.gif
  129043  out0596.gif
```

The contents of the output folder are ca. 77 MB in size.

**6.** I used gifsicle to create a looping animated gif:

```
$ gifsicle -d 10 -l  output/*.gif > monitor_progress.gif
```

The result is one single GIF file, monitor\_progress.gif, still 76869920 bytes in size.

**7.** You can display it in the web browser using the following minimal HTML wrapper monitor\_progress.html:

```
  <html>
    <body>
      <img src="monitor_progress.gif" width="442"/>
    </body>
  </html>
```

This displays the same result in the browser as the animated image above.

#### Optimise the Animated GIF File Generation

After working for a while with this 77 MB GIF and noticing that it was creating some sluggishness in the browser interaction, I had another look and found Chris Messina's suggestions on how to
[convert a MOV to a GIF like a boss](http://chrismessina.me/b/13913393/mov-to-gif),
so I repeated the conversion process a second time using the following command line:

```
$ gifsicle -d 10 -l --optimize=3 --resize-width 442
  output/*.gif > monitor_progress_2.gif
```

That produces a warning message saying "gifsicle: warning: huge GIF, conserving memory (processing may take a while)".

The new animated GIF is 2152053 bytes (ca. 2 MB) in size instead of 77 MB.

It looks pretty similar in the browser, except that the grey-scale and light blue gradient colours are somewhat mixed up:

![MonitorProgress 2](img/monitor_progress_2.gif)

#### Reduce Size in Camtasia Not Gifsicle

To avoid the visible quality deterioration in the GIF resized by gifsicle, I decided to make one more attempt reducing size during the initial rendering to MP4 in Camtasia, producing a 442 x 542 pixel video monitor\_progress\_442.mp4 only 5736643 bytes big, then converting it straight to individual GIF images with ffmpeg and combining them into an animated GIF using gifsicle like this:

```
$ ffmpeg -i monitor_progress_442.mp4 -r 10 output/out%04d.gif
$ gifsicle -d 10 -l output/*.gif > monitor_progress_442.gif
```

Each individual GIF image is about 45641 bytes this time, and the animated GIF monitor\_progress\_442.gif 26715360 bytes.
No colour deterioration, but rather large.

Next, I tested the gifsicle optimisation levels without resizing:

```
$ gifsicle -d 10 -l --optimize=1 output/*.gif > monitor_progress_442_1.gif
$ gifsicle -d 10 -l --optimize=2 output/*.gif > monitor_progress_442_2.gif
$ gifsicle -d 10 -l --optimize=3 output/*.gif > monitor_progress_442_3.gif
```

The resulting file sizes make a surprising jump:

```
  26715360  monitor_progress_442.gif
  21314901  monitor_progress_442_1.gif
   3240386  monitor_progress_442_2.gif
   3136043  monitor_progress_442_3.gif
```

Their quality all seems fine; here is the last and smallest of them:

![MonitorProgress 3](img/monitor_progress_442_3.gif)