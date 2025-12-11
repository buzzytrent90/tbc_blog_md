---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: code_example
optimization_date: '2025-12-11T11:44:15.438600'
original_url: https://thebuildingcoder.typepad.com/blog/1167_roomeditorapp_plural.html
post_number: '1167'
reading_time_minutes: 5
series: general
slug: roomeditorapp_plural
source_file: 1167_roomeditorapp_plural.htm
tags:
- csharp
- elements
- family
- python
- revit-api
- rooms
- views
title: RoomEditorApp PluralString and Free Educational Software
word_count: 955
---

### RoomEditorApp PluralString and Free Educational Software

Yesterday, I mentioned some last-minute clean-up of the JavaScript part of my room editor,
[adding the handlebars module and a call to refresh](http://thebuildingcoder.typepad.com/blog/2014/05/room-editor-with-handlebars-and-refresh.html) on
save.

Today, for the sake of completeness, I'll add a last-minute pass over the Revit add-in part as well, including an even more trivial change.

I'll also mention that I had some time out in nature again, before jetting to Toronto for two days next week, celebrating a deep and wonderful
[sweat lodge](http://en.wikipedia.org/wiki/Sweat_lodge) or
[back to life ceremony](http://www.barefootsworld.net/sweatlodge.html) on
a beautiful plot of land with some friends, my son, overflowing nature and a vast view.

Good preparation, earthing.

#### PluralString Helper Method

During the work on the JavaScript part, I noticed that I could make the message generation and logging code slightly more succinct and readable by implementing a little helper method to encapsulate my previous PluralSuffix calls.

In C#, the result is the new Util.PluralString method:

```csharp
  /// <summary>
  /// Return an English pluralised string for the
  /// given thing or things. If the thing ends with
  /// 'y', the plural is assumes to end with 'ies',
  /// e.g.
  /// (2, 'chair') -- '2 chairs'
  /// (2, 'property') -- '2 properties'
  /// (2, 'furniture item') -- '2 furniture items'
  /// If in doubt, appending 'item' or 'entry' to
  /// the thing description is normally a pretty
  /// safe bet. Replaces calls to PluralSuffix
  /// and PluralSuffixY.
  /// </summary>
  public static string PluralString(
    int n,
    string thing )
  {
    if( 1 == n )
    {
      return "1 " + thing;
    }

    int i = thing.Length - 1;
    char cy = thing[i];

    return n.ToString() + " " + ( ( 'y' == cy )
      ? thing.Substring( 0, i ) + "ies"
      : thing + "s" );
  }
```

I can use the new method to simplify some of my message generation and logging code, e.g. like this:

```python
  Debug.Print( "Selected {0} from {1} displaying "
    + "{2}, {3} with HasMaterialQuantities=true",
    Util.PluralString( Count, "category" ),
    Util.PluralString( \_nViews, "view" ),
    Util.PluralString( \_nElements, "element" ),
    \_nElementsWithCategorMaterialQuantities );
```

That previously looked like this:

```python
  Debug.Print( "Selected {0} categor{1} from "
    + "{2} view{3} displaying {4} element{5}, "
    + "{6} with HasMaterialQuantities=true",
    Count, Util.PluralSuffixY( Count ),
    \_nViews, Util.PluralSuffix( \_nViews ),
    \_nElements, Util.PluralSuffix( \_nElements ),
    \_nElementsWithCategorMaterialQuantities );
```

An example of the resulting message is identical:

```
  Selected 5 categories from 5 views displaying
  1901 elements, 877 with HasMaterialQuantities=true
```

All is well in the best of all worlds if we can worry our pretty little heads about this kind of trivia, isn't it?

#### Download

The RoomEditorApp source code, Visual Studio solution and add-in manifest live in the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp).

The version discussed above is
[release 2015.0.2.18](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2015.0.2.18).

For something slightly less trivial and much more useful, how about this important news item reproduced from Kean's note on
[free Autodesk software for students and educators](http://through-the-interface.typepad.com/through_the_interface/2014/05/free-autodesk-software-for-students-and-educators.html)?

#### Free Autodesk Software for Students and Educators

Students world-wide can download and use pretty much the full range of Autodesk software for free\* from the
[Education Community site](http://students.autodesk.com).

The
[public announcement](http://news.autodesk.com/press-release/autodesk-consumer-group-and-education/autodesk-transforms-education-business-model-hel) was
made a few weeks ago.

This is great news for students around the world!
Although the press release specifically mentions North America, this is apparently a worldwide initiative.

By the way, once you are nearing the end of your studies, you might also want to check the
[Autodesk careers site](http://autodesk.com/careers) as
a next step.

![Free educational Autodesk software](img/adsk_free_edu.png)

*\* Free Autodesk software and/or cloud-based services are subject to acceptance of and compliance with the terms and conditions of the software license agreement or terms of service that accompany such software or cloud-based services. Software and cloud-based services provided without charge to Education Community members may be used solely for purposes directly related to learning, teaching, training, research or development and shall not be used for commercial, professional or any other for-profit purposes.*

#### Multi Culti

I may have mentioned now and then that I am interested in music of all kinds, and my son Christopher deals with sound recording professionally.

Here is his newest music recommendation for me:

[Multi Culti](https://soundcloud.com/multiculti) – "Music to trip to. Music to meditate on. Music to heal the world. Music to upgrade your DNA. Music to get your whole family dancing. Music to teach you. Music to live by. Music to die to. Music to multiply your mind."

I hope you enjoy it too :-)

#### Four Tips on Creativity

While I'm at it anyway, let's go on...

Another recommendation by Christopher,
[The Music of Sound](http://www.musicofsound.co.nz), led me to these
[four tips on creativity](http://designtaxi.com/news/365478/Four-Tips-On-Creativity-From-Bill-Watterson-Creator-Of-Calvin-Hobbes/?t=1400535942577) from
Bill Watterson, creator of the Calvin and Hobbes comics that I really enjoy:

1. Lose yourself in your work – By blocking out the world, you're better able to produce work to the best of your ability.
2. Create for yourself – Forget that there's an audience and focus on making something for yourself.
3. Make it beautiful – People invest time and effort into viewing your work, so make sure it's worth their while.
4. Every medium has power – Though comics might seem like an inconsequential platform, they possess the power to forge a deep connection with readers.

I definitely endorse these.

Have you noticed?   :-)