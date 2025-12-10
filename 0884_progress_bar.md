---
post_number: "0884"
title: "Implement Progress Bar and Abort Lengthy Process"
slug: "progress_bar"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'windows']
source_file: "0884_progress_bar.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0884_progress_bar.html"
---

### Implement Progress Bar and Abort Lengthy Process

The time has come to talk of many things: of shoes and ships, and sealing-wax, of cabbages and kings
[[^](http://www.jabberwocky.com/carroll/walrus.html)]...
or no, let's continue sticking to the Revit API for now, more or less;
today's topics are:

- A [walk in the snow](#2) and a crystal covered baby tree
- Implementation of a Revit add-in [progress bar](#3)
- [Aborting a lengthy process](#4) from the progress bar
- Harry's [image generation video competition](#5)

#### A Walk in the Snow

I went for a walk in snow and fog on a local hill and appreciated this wind-driven ice crystal covered baby tree that:

![Wind-driven ice crystal covered baby tree](file:///j/photo/jeremy/2013/2013-01-15_belchen/32_crystal_covered_twigs_cropped.png)

I thought I would like to share that with you.
Sorry that it is so dark, but that's the way it was.

#### Implementing a Revit Add-in Progress Bar

I chatted with Harry Mattison on various topics during the winter break.
One of them was the implementation of a Revit add-in progress bar:

**Question:** I am trying to make a simple progress dialog based on your post on
[modeless dialogues](http://thebuildingcoder.typepad.com/blog/2009/02/revit-window-handle-and-modeless-dialogues.html).
It seems to work well basically but has a few problems:

1. The first time I run the command, the progress dialog flashes briefly on the screen and then disappears.- The text label does not display properly.- Sometimes the entire progress dialogue content turns white and the title bar says 'Not Responding'.

Could you help me figure out if my approach is flawed or if it just needs some small adjustments?

**Answer:** Thank you, but no thank you... :-)

Instead, I propose that you look at my existing modeless progress bar implementation in the
[ADN Revit MEP sample add-in](http://thebuildingcoder.typepad.com/blog/2012/05/the-adn-mep-sample-adnrme-for-revit-mep-2013.html),
which has worked flawlessly for years now.

**Response:** Your progress bar is great!
I wish I had found it earlier.

#### Aborting a Lengthy Process

Here's hopefully one last question about the progress bar:

How can the user cancel the operation from the progress bar?

**Answer:** You could implement some standard .NET stuff on the progress bar; maybe simply add a cancel button to it, and an event handler for that.

**Response:** After spending some time thinking about events and the like, I realized the progress bar cancel solution is almost trivial.

An idle event wouldn't work in my case, because Revit was never going idle.
It was too busy doing hard work of exporting images for my [Image-O-Matic](http://boostyourbim.wordpress.com/products) application.
Instead, all I needed to do is add the Abort button to the progress dialog, and put this little bit of code in the progress bar class:
```csharp
  private void btnAbort\_Click(
    object sender,
    EventArgs e )
  {
    btnAbort.Text = "Aborting...";
    abortFlag = true;
  }

  public bool getAbortFlag()
  {
    return abortFlag;
  }
```

Then, inside my foreach loops, I just need to check the status of the flag:
```csharp
  foreach( string phaseName in phaseList )
  {
    if( progressForm.getAbortFlag() )
      break;

    // . . .
  }
```

Many thanks to Harry for the fruitful discussion and helpful hint!

#### Image Generation Video Competition

Yet another item from Harry, this time a rather shameless plug, but I guess he earned it :-)

Harry announced a
[competition](http://boostyourbim.wordpress.com/2013/01/16/win-a-free-custom-built-revit-api-app-for-using-image-o-matic) in
which you can win a free, custom-built Revit API add-in according to your own specifications for downloading his
[Image-O-Matic](http://apps.exchange.autodesk.com/RVT/Detail/Index?id=appstore.exchange.autodesk.com%3aimage-o-matic%3aen) application,
generating a video with it, and uploading that to YouTube.

Have fun, and good luck to you!