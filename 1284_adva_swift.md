---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.677924'
original_url: https://thebuildingcoder.typepad.com/blog/1284_adva_swift.html
post_number: '1284'
reading_time_minutes: 7
series: general
slug: adva_swift
source_file: 1284_adva_swift.htm
tags:
- python
- revit-api
- transactions
- views
- windows
title: View and Data API Sample in Swift and Mac OS Upgrade
word_count: 1428
---

### View and Data API Sample in Swift and Mac OS Upgrade

As you know, we have a pretty impressive number of
[samples](https://github.com/Developer-Autodesk) demonstrating
the use of the
[View and Data API](https://developer-autodesk.github.io) web
services.

We are currently reviewing them all to ensure their documentation is reliable and consistent.

I picked Adam Nagy's
[workflow sample written in Swift](https://github.com/Developer-Autodesk/workflow-macos-swift-view.and.data.api) to review.

That required a look at the
[Swift programming language](https://en.wikipedia.org/wiki/Swift_(programming_language)) itself
and a system upgrade before I could start with the review itself.

Before getting to that, let me also mention the upcoming 3D Web Festival, and then wrap up with yet another update of The Building Coder samples:

- [3D Web Festival](#2)
- [Swift](#3)
- [Upgrading OS X, Xcode and Parallels](#4)
- [Swift Command Line](#5)
- [Swift Shell Scripting](#6)
- [View and Data API workflow in Swift](#7)
- [Started eliminating automatic transaction mode](#8)

#### 3D Web Festival

Are you a creative human being interested in 3D on the web?

Then the [3D Web Festival](http://www.3dwebfest.com) may be just your thing.

It will showcase 3D web sites presenting mixtures of music, art and technology, bringing together the best of the 3D Web – presented as Live Performance Art – amazing, delightful, surprising and at times disturbing, including creations by artists and developers of all kinds live with musical accompaniment.

Time and place is Wednesday May 13th from 7 to 10 pm at the Folsom Street Foundry in San Francisco, with ticket proceeds going to the
[Roxie](http://www.roxie.com), a community-based, non-profit theatre.

#### Swift

The [Swift programming language](https://en.wikipedia.org/wiki/Swift_(programming_language)) was
created by [Apple](https://developer.apple.com/swift) for iOS and OS X development.

It uses the same underlying machinery as Objective C with a much simpler syntax.

Python is one of the languages that helped inspire Swift, and Swift also provides a
[REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) or
read-eval-print loop, even though it is a compiled language.

Working with Swift code requires the development environment Xcode 6.1, which in turn requires OS X 10.9.4 or higher, nicknamed Mavericks.

Up until this morning, I was still on 10.8.5, aka Mountain Lion, so it was clear that some serious computer system updating would be required for me to be able t make use of it.

#### Upgrading OS X, Xcode and Parallels

Well, I took the plunge and upgraded straight from 10.8.5 to 10.10.2 Yosemite, since that was the suggestion made by the Apple AppStore.

The next step was upgrading Xcode from my previous version of 5.5.1 to 6.1.1.

That was harder, and I initially failed, until I read the recommendation to simply
[uninstall the previous version](http://apple.stackexchange.com/questions/71662/how-to-can-i-upgrade-xcode-4-1-to-4-5-2-properly) and
then reinstall from scratch.
Even then it was not as simple as it sounded, though.
It took me several attempts before I really had eliminated the previous installation and the reinstallation truly installed the new version instead of just automatically updating the old one.

As far as I can tell so far, all my other applications work fine after the update, except Parallels Desktop, which I use to run Windows application like Revit and Visual Studio on a virtual machine within the Mac OS.

The version of Parallels 8 that I was using before the upgrade did not work at all.
Upgrading that to the newest version of 8 enabled me to start up and use the virtual Windows machines again.

I was unable to access my virtual Windows drives from the Mac environment, though.

The Windows drives can be mounted in the Mac file system, enabling access to Windows files from the Mac command line and applications.

For instance, I have all my Revit add-ins on the C:\ drive and use git on the Mac terminal command line to clone them down from GitHub and push them and their changes back up again.
It would be really painful have to have to move them on a Mac drive, because some loading and debugging does not work unless they are on a local Windows drive.

Happily, simply upgrading to Parallels 10 finally resolved that issue as well, and the Windows drive `C:\` now appears in Mac OS as `/Volumes/C`, just like it always did:

```
  /Volumes/ $ ls -o
  drwxrwxrwx  1 jta   16384 Feb 18 21:04 C
  lrwxr-xr-x  1 root      1 Feb 19 07:34 jtharddisk -> /
```

I seem to be back to normal again, all systems go, and I can run Swift.

#### Swift Command Line

Once Xcode 6.1.1 is installed, the Swift command line is accessible:

```
  $ xcrun swift
  Welcome to Swift!  Type :help for assistance.
    1> :help

    . . .

    1> var x = 1
  x: Int = 1
    2> x+x
  $R0: Int = 2
    3>
```

As it so kindly tells you, you can type `:help` for help, and `:quit` or Ctrl-D to get out of the Swift read-eval-print loop again.

On OS X Yosemite and Xcode 6.1 or later, as in my case, you can run Swift by just typing `swift` in the command line.

#### Swift Shell Scripting

You can create a Swift shell script and use the shebang to run it as a program directly like this:

```
  $ cat hello.sh
  #!/usr/bin/env xcrun swift
  println("hello world")

  $ chmod +x hello.sh
  $ ./hello.sh
  hello world
```

Alternatively, you can pass a Swift script file as an argument to the swift command line like this for the same effect:

```
  $ cat hello.sw
  println("hello world")

  $ swift hello.sw
  hello world
```

Nice tutorials to get started are provided on Santosh Rajan's
[Swift programming](https://medium.com/swift-programming) site, e.g.,
[1. Learn Swift by running Scripts](https://medium.com/swift-programming/1-learn-swift-by-running-scripts-73fdf8507f4b).

A couple of other cool examples are provided by
[Blake Merryman's Swift scripts](https://github.com/blakemerryman/Swift-Scripts).

#### View and Data API Workflow in Swift

Adam's View and Data API workflow sample is by no means a simple Swift script, though, but a full-blown Xcode IDE application:

![Xcode IDE with MacViewStarter app](img/MacViewStarter_xcode.png)

Running the app in the Xcode debugger displays the main screen:

![MacViewStarter app](img/MacViewStarter_app.png)

All you have to do is enter your Autodesk View and Data API credentials, obtained from the
[Autodesk Web Services API](https://developer.autodesk.com) web
page, request an access token, enter a valid bucket name and select a file to upload.

That adds the resulting model identifier – aka
[URN](https://en.wikipedia.org/wiki/Uniform_resource_name) or
uniform resource name – to the URN drop-down list:

![MacViewStarter generates a URN](img/MacViewStarter_urn.png)

The image box at the bottom displays the model thumbnail preview once translation is complete, which may take a while for a large model:

![MacViewStarter displays a thumbnail once translation completes](img/MacViewStarter_thumbnail.png)

To make further use of the translated model, please refer to the numerous other
[View and Data API samples on GitHub](https://github.com/Developer-Autodesk).

Look at the [View and Data API sample overview](https://developer-autodesk.github.io) to
decide where to go next from here.

#### Started Eliminating Automatic Transaction Mode

Let's wrap up with yet another update to
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).

I now started removing the use of the old automatic transaction mode.

It is probably best never to use it at all nowadays, and go for either Manual or ReadOnly mode, depending on whether you need to update the model or not.

The first occurence that I happened upon is in the age-old
[CmdInstallLocation](http://thebuildingcoder.typepad.com/blog/2009/09/revit-install-location.html) command,
so I updated that from Revit 2010 to 2015 and replaced TransactionMode.Automatic by ReadOnly in it.

Please note that that command is probably not needed at all nowadays, since other methods are available to determine the Revit installatin location, e.g., using the
[perpetual GUID algorithm](http://thebuildingcoder.typepad.com/blog/2013/04/perpetual-guid-algorithm-and-revit-2014-product-guids.html).

Anyway, the updated command is included in
[release 2015.0.117.4](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.117.4) with
more to follow.