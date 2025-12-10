---
post_number: "1231"
title: "Berlin Hackathon Results, 3D Viewer and Web News"
slug: "berlin_hack_va3c_adva"
author: "Jeremy Tammik"
tags: ['csharp', 'geometry', 'parameters', 'python', 'references', 'revit-api', 'rooms', 'schedules', 'views']
source_file: "1231_berlin_hack_va3c_adva.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1231_berlin_hack_va3c_adva.html"
---

### Berlin Hackathon Results, 3D Viewer and Web News

We completed the Berlin hackathon last weekend, and many other noteworthy and exciting things are going on, concerning the Revit API, the 3D web viewers, web services and more:

- [Berlin hackathon projects and winners](#2)
- [Avoid unnecessary unit conversion](#3) – embrace the natives
- [3D viewer news](#4)
- [AutoCAD as a web service](#5)
- [Collada STL ADVA export settings](#6)
- [RvtVa3c update](#7)

#### Berlin Hackathon Projects and Winners

The
[Berlin hackathon](https://twitter.com/hashtag/tmuhack) completed and the winners were announced last weekend.

I took notes of four teams' project presentations:

- US4, us four –
  social data visualisation in 3D;
  create JSON data to display 3D graph to visualise friends, followers, business contacts, etc. in the browser;
  technology: looked at the Autodesk viewer, thinking of feeding it via OBJ files;
  generating these files is harder than using straight three.js, though;
  extract data from github, twitter, facebook, etc.
- [MovieMemory](https://m3my.github.io) –
  We built a game using [Firebase](https://www.firebase.com), recently acquired by Google and rather pricey.
  It also uses nodejs, angular, npm, bower and grunt.
  We also looked at Meteor as a smaller, simpler, more self-contained db alternative.
  Some see it [Meteor as an alternative to angular](http://differential.io/blog/meteor-vs-angular);
  Meteor can also be used [in conjunction with Angular](http://angularjs.meteor.com/tutorial).
  We grabbed movie details from both [imdb](http://www.imdb.com) and [wikipedia](https://www.wikipedia.org)
  and sent them through the [neophonie](http://www.neofonie.de) web service to automatically extract keywords for each film.
  The film poster is grabbed from imdb as well.
  Did you know about [sparql](http://dbpedia.org/sparql), enabling you to query wikipedia programmatically as a database?
  Two cards are generated for each film: one with the keywords, the other with the banner.
  The aim is to reveal cards, remember their locations, and match as many pairs as possible.
- Shopoolit by Pawan –
  four sections: popular stores, favourites, inspire;
  add to favourites, add to schedule;
  see details: who is going, with peer to peer rating;
  activity list linked to calendar, saved in list, scroll, share these plans, see number of requests;
  see pending and accepted requests;
  basic profile, photo and basic bio or facebook.
- Tourist attraction web scraper by Jake, like 360 cities, and reminiscent of my recent
  [PoiPointer](http://poipointer.github.io) project:
  scrape images of tourist attractions for any city, display images, work in app on phone;
  mobile city trip planner;
  when out walking, you see something and don't know what it is;
  take a picture, take image, compare with google images, identify object, return description from wikipedia;
  what technologies? node and amber in front and back, python script to scrape sites.

The first place was awarded the US4 team.
Here they are after Peter Schlipf presented them the main prize, four smart watches sponsored by Autodesk:

![US4, TMUHack winners](img/tmuhack_us4_2.jpg)

The second place went to the [MovieMemory](http://m3my.github.io) team:

![MovieMemory, TMUHack runners-up](img/tmuhack_m3my_2.jpg)

They were awarded the [neophonie](http://www.neofonie.de) prize, a drone helicopter with built-in camcorder.

Here they are during the live presentation of their newly created online [movie memory game](http://moviememory.de):

![MovieMemory, TMUHack runners-up](img/tmuhack_m3my.jpg)

You can try out MovieMemory live yourself right now.
You need a partner to get going.
Simply go to [moviememory.de](http://moviememory.de), copy the game link created and pass it on to your playing partner to start playing right away:

![MovieMemory start page](img/moviememory.png)

#### Avoid Unnecessary Unit Conversion – Embrace the Natives

Here is some valuable and sound advice on working in native units and avoiding unnecessary conversions provided by Scott Wilson in his answers to the Revit API discussion thread on
[converting survey points](http://forums.Autodesk.com/t5/revit-api/convert-to-survey-points/m-p/5361921):

**Question:** I am extracting four corners of a rectangular column.
My code works perfectly in one project but not in another.
The only difference between the two that I noticed is that the project and survey points are the same in the first, whereas they differ in the second.

**Answer:** From a quick look through the code, it looks like you might be mixing your dimensional units.
Are the points out by a multiple of 304.8 by any chance?

Halfway through you are converting from feet to millimetres (parameter values h and b); later, you apply a transform (with its translation values in feet) to points created using these metric values.
I suspect that the error is masked in your first project due to the translation from project to survey points being zero.

I'd suggest leaving all dimensions in feet until you need to display or export them.

**Response:** Yes I am multiplying by 304.8 as Revit returns the values in feet but the project is in millimetres.

What should I do to resolve the issue?

**Answer:** Why do you need to convert into metric during the calculation?

I can't see anywhere in the code snippet where you are displaying or exporting the interim values, so just leave them all in feet and let Revit work in its native units. If you need the output values in metric, convert them as the last step.

I also just noticed that you are converting the location point rotation into degrees and then round that to the nearest degree.
You then convert this rounded value back into radians to perform some trig calcs.
This is going to be quite inaccurate and inconsistent, just leave angles in raw radians without rounding.

Don't convert / mix units or perform rounding mid-way through a calculation, you are just asking for trouble.
If you are uncomfortable working in feet and radians, I'd suggest making the effort to embrace them.

Many thanks to Scott for this recommendation, and for his numerous other great replies in the Revit API discussion forum!

Please everybody take heed of this very sound advice!

#### 3D Viewer News

Theo Armour, initiator of the
[vA3C open source three.js AEC 3D model viewer](https://va3c.github.io) posted
an update on the current vA3C status and the following call to action:

> The [latest release of the vA3C Viewer](http://va3c.github.io/viewer/va3c-viewer-html5/latest) is looking good.
>
> It would be great to hear from you that something in this effort is worthwhile or useful to you.
>
> Virtually everything we set out to do back in May is being done – sometimes elegantly, some times not so, but never mind.
>
> Over the summer, I thought the work on the viewer would finish and I could get back to my 3D mapping work and etc.
>
> In the last few weeks, those thoughts have kind of been exploded. I am coming to the realization that we asked for just the tip of the iceberg.
>
> Metaphorically, the viewer is can be seen as theatrical stage set. The next step is to use this stage to produce theatre.
>
> For example, you all know Oculus Rift, Google Cardboard, Magic Leap and the whole virtual reality thing is exploding.
>
> A few weeks ago I built a [Google cardboard three.js viewer template](http://jaanga.github.io/cookbook/cardboard/readme-reader.html).
>
> The intention is to build this capability into the vA3C Viewer.
>
> But guess who used my templates and got there first? The Autodesk 360 Viewer!
>
> My new friend (and Jeremy's colleague) Kean Walmsley was pointed in our direction by Jim Quanci and Cyrille Fauvel and started [gearing up for the VR Hackathon](http://through-the-interface.typepad.com/through_the_interface/2014/10/autocad-io-api-a-new-batch-processing-web-service.html).
>
> Not only that, but he has already built demos on top of my [voice recognition template](http://jaanga.github.io/cookbook/voice-commands/readme-reader.html) and produced the series of further exciting results listed below.
>
> And there's much, much more beyond this.
>
> From my point of view, this all looks like the start of a classic Internet disruption by a small start-up.
>
> The technology is there; the industry that needs disrupting (AEC) is obediently stagnating.
>
> All that's needed is the founders...

Here is an overview if Kean's recent posts:

- [Gearing up for the VR Hackathon](http://through-the-interface.typepad.com/through_the_interface/2014/10/gearing-up-for-the-vr-hackathon.html)
- Creating a stereoscopic viewer for Google Cardboard using the Autodesk 360 viewer:

- [Part 1](http://through-the-interface.typepad.com/through_the_interface/2014/10/creating-a-stereoscopic-viewer-for-google-cardboard-using-the-Autodesk-360-viewer-part-1.html)
- [Part 2](http://through-the-interface.typepad.com/through_the_interface/2014/10/creating-a-stereoscopic-viewer-for-google-cardboard-using-the-Autodesk-360-viewer-part-2.html)
- [Part 3](http://through-the-interface.typepad.com/through_the_interface/2014/10/creating-a-stereoscopic-viewer-for-google-cardboard-using-the-Autodesk-360-viewer-part-3.html)

- [VR Hackathon 2014 in SF](http://through-the-interface.typepad.com/through_the_interface/2014/10/vr-hackathon-2014-in-sf.html)

Too exciting to miss!

#### AutoCAD as a Web Service

After all the exciting 3D stuff listed above, Kean went on to write about
[making programmatic use of the AutoCAD core services via a web service](http://through-the-interface.typepad.com/through_the_interface/2014/10/autocad-io-api-a-new-batch-processing-web-service.html).

This is another topic that every application developer should be thinking about and aware of.

Just as Theo points out, things are really taking off!

#### Collada STL ADVA Export Settings

Talking about the 3D viewer, I built my daughter Marie a new writing desk, and a friend of hers created this model of Maria's room to discuss our plan:

He had to explore a bit to get the Collada STL export setting right for the model to display correctly in the Autodesk View and Data API viewer; here they are:

![Collada STL export settings for the Autodesk View and Data API](img/collada_stl_export_settings.png)

#### Custom User Settings Storage and RvtVa3c Update

Back to the Revit API and the vA3C viewer again, and based on the discussion with David and Theo on the
[failure to load a big model](https://github.com/va3c/viewer/issues/6),
I tried reducing the file size of the JSON geometry output produced by the RvtVa3c three.js and vA3C editor model exporter and added support for runtime reading of user settings and switching between indented and non-indented JSON.

The necessary switch is provided by the JsonConvert class defined by the Newtonsoft.Json library that I am using to serialise the classes collected from the custom exporter to JSON, and set like this in its call to serialise them:

```csharp
  Formatting formatting
    = UserSettings.JsonIndented
      ? Formatting.Indented
      : Formatting.None;

  File.WriteAllText( \_filename,
    JsonConvert.SerializeObject(
      \_container, formatting, settings ) );
```

I implemented a new class UserSettings to provide a simple way for the end user to control this property.

I obviously considered using the built-in .NET user settings classes, but they interact rather heavily with the operating system and can be difficult to use in the context of a Revit add-in.

I also considered using a custom implementation, e.g. this
[custom settings class for WinForms](http://www.blackbeltcoder.com/Articles/winforms/a-custom-settings-class-for-winforms) by
Jonathan Wood, the Black Belt Coder.

In the end, however, I preferred to avoid all hassles with other people's code and paradigms and simply write my own.

My UserSettings class checks whether a file with the same name and location as the Revit add-in exists, except the filename extension DLL is replaced by TXT.
If so, it reads the user preferences from there.
If not, it creates a new file using the default value that can easily be modified by the user.

To support its parsing of a Boolean value, I implemented this new utility method to read a true or false value from a string:

```python
  /// <summary>
  /// Extract a true or false value from the given
  /// string, accepting yes/no, Y/N, true/false, T/F
  /// and 1/0. We are extremely tolerant, i.e., any
  /// value starting with one of the characters y, n,
  /// t or f is also accepted. Return false if no
  /// valid Boolean value can be extracted.
  /// </summary>
  public static bool GetTrueOrFalse(
    string s,
    out bool val )
  {
    val = false;

    if( s.Equals( Boolean.TrueString,
      StringComparison.OrdinalIgnoreCase ) )
    {
      val = true;
      return true;
    }
    if( s.Equals( Boolean.FalseString,
      StringComparison.OrdinalIgnoreCase ) )
    {
      return true;
    }
    if( s.Equals( "1" ) )
    {
      val = true;
      return true;
    }
    if( s.Equals( "0" ) )
    {
      return true;
    }
    s = s.ToLower();

    if( 't' == s[0] || 'y' == s[0] )
    {
      val = true;
      return true;
    }
    if( 'f' == s[0] || 'n' == s[0] )
    {
      return true;
    }
    return false;
  }
```

That is used to determine the value of the UserSettings class JsonIndented property like this:

```python
class UserSettings
{
  const string \_JsonIndent = "JsonIndent";

  const string \_error\_msg\_format
    = "Invalid settings in '{0}':\r\n\r\n{1}"
    + "\r\n\r\nPlease add {2} = {3} or {4}.";

  static bool SyntaxError( string path, string s )
  {
    Util.ErrorMsg( string.Format(
      \_error\_msg\_format, path, s, \_JsonIndent,
      Boolean.TrueString, Boolean.FalseString ) );

    return false;
  }

  public static bool JsonIndented
  {
    get
    {
      string path = Assembly.GetExecutingAssembly()
        .Location;

      path = Path.ChangeExtension( path, "txt" );

      if( !File.Exists( path ) )
      {
        File.WriteAllText( path,
          \_JsonIndent + "=" + Boolean.TrueString );

        Util.ErrorMsg( string.Format(
          "Created a new user settings file at '{0}'.",
          path ) );
      }

      string s1 = File.ReadAllText( path );

      int i = s1.IndexOf( \_JsonIndent );

      if( 0 > i )
      {
        return SyntaxError( path, s1 );
      }

      string s = s1.Substring( i
        + \_JsonIndent.Length );

      i = s.IndexOf( '=' );

      if( 0 > i )
      {
        return SyntaxError( path, s1 );
      }

      s = s.Substring( i + 1 ).Trim();

      bool rc;

      if( !Util.GetTrueOrFalse( s, out rc ) )
      {
        return SyntaxError( path, s1 );
      }

      return rc;
    }
  }
}
```

Editing the automatically generated file and specifying an invalid Boolean value such as the string "a non-Boolean value" produces the following error message:

![RvtVa3c invalid Boolean value error message](img/rvtva3c_JSON_indent_invalid_boolean.png)

Please excuse the overkill :-)

The entire updated RvtVa3c custom exporter add-in implementation is provided in the
[RvtVa3c GitHub repository](https://github.com/va3c/RvtVa3c),
and the version discussed above is
[release 2015.0.0.26](https://github.com/va3c/RvtVa3c/releases/tag/2015.0.0.26).

Enjoy!