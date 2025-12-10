---
post_number: "0856"
title: "Moving to the Mac"
slug: "moving_to_mac"
author: "Jeremy Tammik"
tags: ['family', 'python', 'revit-api', 'windows']
source_file: "0856_moving_to_mac.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0856_moving_to_mac.html"
---

### Moving to the Mac

I have been quiet for a whole week now, busy setting up my Mac, learning new functionality, techniques, keyboard shortcuts, editors, you name it.

Under Windows, I was already using quite a number of batch files and command line Unix-style tools, so I am havery happy to have a real Unix shell to play with now.

On the other hand, each batch file needs some fixes applied to convert it to a shell script, and every keystroke needs some additional neutrons firing to switch from the Windows habits to new Mac conventions.

Several of my colleagues are also using Macs nowadays, although they seem to be mostly remaining inside a Windows environment and using Parallels to run it on the Mac.
I am [going native](http://en.wiktionary.org/wiki/go_native) :-)

I installed the Xcode envornment to have access to standard development tools like make and gcc, the
[Komodo](http://komodo-edit.en.softonic.com) text editor,
[TrueCrypt](http://www.truecrypt.org) to manage all my secret stuff,
Firefox, Skype, Office, Parallels, and much more.

Some of my own command line tools are in C, others in Python.
Everything works fine, so far, many things required some tweaks or cleanup, and much remains to be done.

#### AU Material Submitted

I managed to keep the deadline yesterday for submission of the Autodesk University material.

For your delectation and the pleasure of online search engines, here is a reproduction of it in its current state.

I may and probably will update it a bit more before the final event, but this should give you a rough idea of everything:

- [CP4109](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=4109) Revit API Round Table: Meet the Champions
  - Tuesday, Nov 27, 5:30 PM - 7:00 PM- Handout- Presentation- [CP4108](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=4108) Revit MEP Programming: All Systems Go
    - Wednesday, Nov 28, 1:00 PM - 2:30 PM- Handout- Presentation- Material- [CP4107](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=4107) Let's face it: New Autodesk Revit 2013 UI API Functionality
      - Thursday, Nov 29, 1:00 PM - 2:30 PM- Handout- Presentation- Material

All done on the Mac :-)

Meanwhile, Joe and Saikat brought a case to my attention and asked me to publish a qiuick note on it, since the answer is apparently not completely trivial to find:

#### Determine Whether Project Document is Workshared or Not

**Question:** I'm creating a utility to copy central (workshared) RVT files to a specific folders on the user laptop to make a local copy, then open it in Revit.

How can I determine whether the RVT file is a central file (workshared) or stand-alone before copying it?

If stand-alone, it would not require copying; it could just be opened in Revit.

It could be opened in memory first, but that would increase the time to eventually open and activate it for work.

I looked into the TransmissionData, but I can find no solution.
Is there one?

**Answer from Joe:** I think BasicFileInfo.IsWorkshared property is what you are looking for.

The BasicFileInfo class can retrieve some metadata file information without fully opening the RVT file in Revit.

This was briefly mentioned between the lines when recently discussing how to
[detach and discard worksets](http://thebuildingcoder.typepad.com/blog/2012/10/detach-workset-and-taskdialog-command-link-order.html).
I hope it helps to highlight this fact for itself as well.

#### Create Radial Dimension

Saikat published another example on the AEC DevBlog, showing how to access the sketch of a circular extrusion in a family document and make a call to
[NewRadialDimension to create a radial dimension](http://adndevblog.typepad.com/aec/2012/11/adding-radial-dimension-to-a-circular-column-in-edit-extrusion-mode-using-revit-api.html).

There, my first blog post completed and published on a Mac.

Now to get back to real work again, gradually, at least for a short period before sinking into the gory mess of conference time.