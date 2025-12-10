---
post_number: "1318"
title: "CopyElements, Revit 2016 Scalability, Python and Ruby"
slug: "2016_scalability"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'python', 'revit-api', 'rooms', 'sheets', 'transactions', 'views', 'windows']
source_file: "1318_2016_scalability.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1318_2016_scalability.html"
---

### CopyElements, Revit 2016 Scalability, Python and Ruby

Here are a bunch of infos on and updates for Revit 2016, as well as some important hints on the use of the ElementTransformUtils.CopyElements method:

- [CopyElements view argument is for 2D only](#2)
- [Revit 2016 scalability enhancements](#3)
- [Revit 2016 Python shell](#4)
- [Revit 2016 Ruby shell](#5)
- [RevitLookup for Revit 2016](#6)

#### CopyElements View Argument is for 2D only

You should only use the ElementTransformUtils.CopyElements views argument when working with view dependent elements, i.e., 2D elements.

This was pointed out by Arnošt Löbel in the Revit API discussion forum thread raised by HD12310 on using
[ElementTransformUtils.CopyElements from a linked document](http://forums.autodesk.com/t5/revit-api/elementtransformutils-copyelements-from-linked-document/m-p/5574351),
who also points out some other important usage considerations:

**Question:**
I am trying to copy paste linked elements into the active document.

This is something that Revit can do manually: select a linked element, then ctrl + c and ctrl + v.

But I guess there is an API reason why this won't work:

```csharp
  ElementTransformUtils.CopyElements( view3DInLink,
    ids, view3DInHost, null, new CopyPasteOptions() );
```

It errors out on "The specified view cannot be used as a source or destination for copying elements between two views." (Which I presume is the linked view.)

It does work if you just want to copy the families from the link (without placing them) like this:

```csharp
  ElementTransformUtils.CopyElements( linkedDoc, ids,
    thisDoc, null, new CopyPasteOptions() );
```

Is there any other method that will copy and paste linked elements, or should this work?

If not, then Create.NewFamilyInstance is probably the only option here   :-)

**Answer:**
Afaik it should work.

Maybe it will help if you do it in several separate steps, e.g. first some supporting elements, like styles, etc., then the instances?

It is probably not entirely obvious from the documentation, but the Copy method that takes views as arguments only work for copying of view-specific elements. And, indirectly, since 3D views – which is what I assume you are copying between – cannot even have view-specific elements, they can never be used in that particular Copy method. That is why you see the error you got even before Revit went on testing whether the elements themselves were view-specific or not.

The work-around is simple: Use the Copy method that takes two documents instead. That should work.

**Response:**
Thanks for the replies.

I finally found my mistake... finally, at 00:25 :-)

I read somewhere that you needed to supply the FamilySymbolIds.

Which is what I did, so I got the family types, not the instances.

(Cool enough, since Transfer Project Standards doesn't transfer families.)

But then I realized you probably get what you ask for here.

So I supplied the element ids of the linked family instances and voila, they pasted right in :-)

It even works on workplane based families; haven't tried face based yet.

For anyone interested, here's the code:

```csharp
using System;
using System.Collections.Generic;
using System.Collections;
using System.Collections.ObjectModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
namespace TransferFamilys
{
  [TransactionAttribute(TransactionMode.Manual)]
  public class CopyPasteFamilys : IExternalCommand
  {
    public class CopyUseDestination
      : IDuplicateTypeNamesHandler
    {
      public DuplicateTypeAction
        OnDuplicateTypeNamesFound(
          DuplicateTypeNamesHandlerArgs args )
      {
        return DuplicateTypeAction.UseDestinationTypes;
      }
    }

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      Document hostDoc = commandData.Application
        .ActiveUIDocument.Document;

      // Get the link

      FilteredElementCollector links
        = new FilteredElementCollector( hostDoc )
          .OfClass( typeof( RevitLinkInstance ) );

      Document linkedDoc = links.Cast<RevitLinkInstance>()
        .FirstOrDefault().GetLinkDocument();

      // Get familys in link

      FilteredElementCollector linkedFamCollector
        = new FilteredElementCollector( linkedDoc );

      ICollection<ElementId> ids = linkedFamCollector
        .OfClass( typeof( FamilyInstance ) )
        .OfCategory( BuiltInCategory.OST\_GenericModel )
        .ToElementIds();

      if( ids.Count == 0 )
      {
        TaskDialog.Show( "Copy Paste",
          "The link does not contain the specified elements." );
      }
      else
      {
        using( Transaction targetTrans
          = new Transaction( hostDoc ) )
        {
          CopyPasteOptions copyOptions
            = new CopyPasteOptions();

          copyOptions.SetDuplicateTypeNamesHandler(
            new CopyUseDestination() );

          targetTrans.Start(
            "Copy and paste linked families" );

          ElementTransformUtils.CopyElements(
            linkedDoc, ids, hostDoc, null,
            copyOptions );

          //hostDoc.Regenerate();
          targetTrans.Commit();
        }
      }
      return Result.Succeeded;
    }
  }
}
```

Many thanks to HD12310 for raising this and sharing the solution!

#### Revit 2016 Scalability Enhancements

This topic was raised by kondaments'
[query](http://www.youtube.com/watch?v=fo55QTwBdBM&google_comment_id=z13riljqnrmycdanv23wwzeh3vmrvjrit04) on the
[Revit 2016 API News video](http://www.youtube.com/watch?v=fo55QTwBdBM) provided with the
[DevDays Online Recording](http://thebuildingcoder.typepad.com/blog/2015/04/revit-2016-api-news-and-devdays-online-recording.html):

**Question:**
At 6:23 and later, what are the specs of the machine(s) used? Graphics card, processor etc.?

**Answer:**
Thank you for your query.

This actually provides a great chance to talk about how the Revit 2016 scalability enhancements are machine independent and actually make the specs of the specific machines involved less important. Unlike some of our competitors we are being smarter about how we process data, rather than trying to process more data faster.

The person creating the demo likely used their own machine to create the video – actually, different demos may have been created by different people.
I think most machines are HP Z600's or the newer Dell Precision 7810 (or higher).

The significant performance improvements in Revit 2016 should be seen with a wide array of hardware and are not limited to any particular technology.

#### Revit 2016 Python Shell

Daren Thomas updated
[RevitPythonShell](https://github.com/architecture-building-systems/revitpythonshell) for
Revit 2016, and it now lives on GitHub:

**Daren:**
RPS is now on GitHub at [github.com/architecture-building-systems/revitpythonshell](https://github.com/architecture-building-systems/revitpythonshell).

I just created an installer for Revit 2016 for it.

A couple of users have been pushing the boundaries.
Maybe you have heard of the non-modal shell, that I think could be quite interesting for interactive development.

I would like to include your RevitLookup tool in RPS, with a function like snoop(Element) – according to the license, that should be ok.
What do you think?

**Jeremy:** Actually, that raises several topics I would like to address:

#### Revit Python Shell

Congratulations on moving to GitHub.
I think that is a great choice.

Your home page there looks great.

Congratulations on migrating to Revit 2016.

Congratulations on the modeless shell. I was not aware of that.

#### Revit Ruby Shell

Have you looked at the [Revit Ruby Shell](https://github.com/hakonhc/RevitRubyShell)?

It has a modeless shell too, afaik.
I hope you are already in touch and sharing with that project.
If not, you absolutely should be!
These two projects have so much in common!
And are both so great!

#### RevitLookup

Yes, of course you can incorporate RevitLookup into the Revit Python Shell.

It is on GitHub too, at [github.com/jeremytammik/RevitLookup](https://github.com/jeremytammik/RevitLookup).

Please do not just copy the code and paste it into the Python shell!

Instead, let us define an API for it and use that to incorporate it more professionally as a separate component.

I will happily work together with you on that.

That would be fun!

#### GeoSnoop

Another thing that I was thinking of adding to RevitLookup is a functionality to snoop geometry, viewing it as geometry, not numerical data, i.e. curves and stuff.

Maybe two viewers, a 2D one for planar things and a 3D one for solids.

The 2D one is named GeoSnoop and has been implemented for ages but not incorporated into RevitLookup yet, e.g., in these discussions:

- [Size and location of viewports on a sheet](http://thebuildingcoder.typepad.com/blog/2014/04/determining-the-size-and-location-of-viewports-on-a-sheet.html)
- [Room and furniture loops using symbols](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html#7)

#### Revit 2016 Ruby Shell

[Håkon Clausen](https://github.com/hakonhc) just updated the
[Revit Ruby Shell](https://github.com/hakonhc/RevitRubyShell) for Revit 2016 as well.

#### RevitLookup for Revit 2016

Last but not least, I published a minor update to [RevitLookup](https://github.com/jeremytammik/RevitLookup) today:

The previous version was referring to the Revit Copernicus pre-release assembly DLL locations in `C:\Program Files\Autodesk\Revit Copernicus`.

I now installed the final official version of Revit 2016 and changed the location to `C:\Program Files\Autodesk\Revit 2016`.

Enjoy!