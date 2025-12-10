---
post_number: "1062"
title: "RoomEditorApp Idling and Benchmarking Timer"
slug: "roomeditorapp"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'revit-api', 'rooms', 'schedules', 'views']
source_file: "1062_roomeditorapp.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1062_roomeditorapp.html"
---

### RoomEditorApp Idling and Benchmarking Timer

I solved my two RoomEditorApp
[Idling issues](http://thebuildingcoder.typepad.com/blog/2013/11/roomeditorapp-architecture-and-external-application.html#4)!

They were especially worrying due to the fact that I am planning to present this application in my Autodesk university class DV1736 on *Cloud-Based, Real-Time, Round-Trip, 2D Revit Model Editing on Any Mobile Device*.

By the way, you should now enrol in the AU classes you are interested in – unless you already did :-)

Go to the
[AU class catalogue](https://events.au.autodesk.com/connect/publicDashboard.ww),
sign in, click 'View All Classes', use the filters or keyword search to locate an event you want to schedule and add it to your personal schedule using the scheduling options and blue plus sign.

Here are my four classes, updated from the AU
[class catalogue preview](http://thebuildingcoder.typepad.com/blog/2013/08/open-mep-connector-warning.html#2) version:

- **[DV1736](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=1736)** –
  Cloud-Based, Real-Time, Round-Trip, 2D Revit Model Editing on Any Mobile Device –
  This presentation demonstrates real-time, round-trip editing of a simplified 2D rendering of an Autodesk Revit intelligent model on any mobile device with no need to install any additional software whatsoever beyond a web browser. How can this be achieved? A Revit software add-in exports polygon renderings of room boundaries and other elements such as furniture and equipment to a cloud-based repository that is implemented using an Apache CouchDB NoSQL database. On the mobile device, the repository is queried and the data rendered in a standard browser using server-side generated JavaScript and SVG. The rendering supports graphical editing, specifically translation and rotation of the furniture and equipment. Modified transformations are saved back to the cloud database. The Revit add-in picks up these changes and updates the Revit intelligent model in real-time. All of the components used are completely open source, except for Revit itself. This is an advanced class for experienced programmers
  ([handout](http://aucache.autodesk.com/au2013/sessionsFiles/1736/99/handout_1736_dv1736_2d_revit_model_editor_handout.pdf) and
  [slides](http://aucache.autodesk.com/au2013/sessionsFiles/1736/104/presentation_1736_dv1736_2d_revit_model_editor_slides.pdf)).
- **[DV1914](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=1914)** –
  Revit API Expert Roundtable: Open House on the Factory Floor –
  Interact with a panel of Autodesk Revit API experts from Autodesk to get answers to your questions and discuss all relevant topics of your choice. If you are writing add-ins for Revit software, then this is the perfect forum to get to know better the people who shape the APIs you work with and to explain your views, ideas, and problems directly face-to-face. Note that prior .NET programming and Revit programming experience is required and that this class is not suitable for beginners
  ([handout](http://aucache.autodesk.com/au2013/sessionsFiles/1914/85/handout_1914_dv1914_revit_api_expert_roundtable_handout.pdf) and
  [slides](http://aucache.autodesk.com/au2013/sessionsFiles/1914/83/presentation_1914_dv1914_revit_api_expert_roundtable_slides.pdf)).
- **[DV2010](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=2010)** –
  Advanced Revit 2014 API Features and Samples –
  This class focuses on some of the major new Autodesk Revit 2014 API features. We look at API access to the project browser, dockable panels, copy and paste, command launching, the graphics pipeline, schedule formatting, and additions to the view API including demonstration and discussion of sample code. We also provide an overview of all the new Revit 2014 SDK samples. Note that prior .NET and Revit programming experience is required and that this class is not suitable for beginners
  ([handout](http://aucache.autodesk.com/au2013/sessionsFiles/2010/101/handout_2010_dv2010_advanced_revit_2014_api_handout.pdf) and
  [slides](http://aucache.autodesk.com/au2013/sessionsFiles/2010/106/presentation_2010_dv2010_advanced_revit_2014_api_slides.pdf)).
- **[DV3464-R](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=3464)** –
  Making Revit Add-ins That Cooperate with Worksharing: A Roundtable Session –
  Are you an Autodesk Revit-based software developer who has run into issues supporting your add-ins in a workshared environment? Have you hit situations where elements can’t be edited or updated due to conflicts? As a followup discussion to “DV1888: Facing the Elephant in the Room: Making Revit Add-ins That Cooperate with Worksharing”, join this roundtable session to further discuss the techniques that are available to developers to operate on workshared models. Bring questions from your own development experience or more detailed questions inspired by the lecture and discuss the possibilities with a group of experienced Revit API developers. Knowledge of C# and Revit API is required, and prior experience with Revit worksharing will be helpful
  ([slides](http://aucache.autodesk.com/au2013/sessionsFiles/3464/374/presentation_3464_Worksharing%20API%20Roundtable.pdf)).

Back to my RoomEditorApp, though, and the nitty-gritty details of resolving my Idling issues.

#### RoomEditorApp Idling Issues

I recently wrote about my updated Revit 2014 version of the RoomEditorApp, the Revit add-in part of my cloud-based real-time round-trip 2D Revit model editing application, the
[RoomEditorApp GitHub repository](http://thebuildingcoder.typepad.com/blog/2013/10/roomeditorapp-for-revit-2014-on-github.html) and its
[architecture and external application](http://thebuildingcoder.typepad.com/blog/2013/11/roomeditorapp-architecture-and-external-application.html) implementation.

In closing, I mentioned two worrying
[Idling related problems](http://thebuildingcoder.typepad.com/blog/2013/11/roomeditorapp-architecture-and-external-application.html#4):

- Unsubscribing from the Idling event had no effect.
- Decreased responsiveness of the Idling event.

Happily, and somewhat surprisingly, I yesterday resolved them both.

The issue with the unsubscription was easy, as I recently discovered and discussed the cause.

I was still worried that I might have to switch back to Revit 2013 for the demo due to the unresponsiveness, however.

Luckily, I thought of adding some benchmarking instrumentation code to find out where the issue was, and that helped find and fix the root cause.

It was my own fault :-)

Here are the steps:

- [Fixing the RoomEditorApp Idling unsubscription](#3)
- [Adding a benchmarking timer](#4)
- [Benchmarking instrumentation](#5)
- [Initial benchmarking results and conclusion](#6)
- [Improving the CouchDB sequence number query](#7)
- [Twiddling with Idling responsiveness](#8)
- [Download](#9)

#### Fixing the RoomEditorApp Idling Unsubscription

My attempts to unsubscribe from the Idling event were failing due to the fact that each new external command invocation generates a new different class instance.
The unsubscribing instance was therefore providing a different member method to unsubscribe than the original one used to subscribe, so the original just kept going, as explained in the discussion on
[singleton application versus multiple command instances](http://thebuildingcoder.typepad.com/blog/2013/11/singleton-application-versus-multiple-command-instances.html).

That was really easy to fix, since I only had to modify the external application to keep track of the handler used to subscribe, and reuse that specific stored method delegate when unsubscribing again.

I published the fix in
[release 2014.0.0.16](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.0.16).

#### Adding a Benchmarking Timer

While I was at it, I decided to spend at least a little time trying to understand the cause for this unresponsiveness.

But where to start?

Well, first of all, I needed to understand where the time was actually being consumed.

To do so, I looked at my
[timer code for benchmarking](http://thebuildingcoder.typepad.com/blog/2012/01/timer-code-for-benchmarking.html) and
the JtTimer class that I used to create some
filtered element collector benchmarks.

Here is the spruced up version that I ended up using for the following tests:

```csharp
/// <summary>
/// Performance timer for profiling purposes.
/// For a full description, please refer to
/// http://thebuildingcoder.typepad.com/blog/2010/03/performance-profiling.html
/// </summary>
public class JtTimer : IDisposable
{
  #region Internal TimeRegistry class
  class TimeRegistry
  {
    #region Internal data and helper methods
    class Entry
    {
      public double Time { get; set; }
      public int Calls { get; set; }
    }

    static Dictionary<string, Entry> \_collection
      = new Dictionary<string, Entry>();

    /// <summary>
    /// Return the percentage based on total time.
    /// </summary>
    /// <param name="value">value</param>
    /// <param name="totalTime">total time</param>
    /// <returns></returns>
    static double GetPercent(
      double value,
      double totalTime )
    {
      return 0 == totalTime
        ? 0
        : Math.Round( value \* 100 / totalTime, 2 );
    }
    #endregion // Internal data and helper methods

    /// <summary>
    /// Add new duration for specified key.
    /// </summary>
    public static void AddTime(
      string key,
      double duration )
    {
      Entry e;
      if( \_collection.ContainsKey( key ) )
      {
        e = \_collection[key];
      }
      else
      {
        e = new Entry();
        \_collection.Add( key, e );
      }
      e.Time += duration;
      ++e.Calls;
    }

    /// <summary>
    /// Write the report of the results to a text file.
    /// </summary>
    public static void WriteResults(
      string description,
      double totalTime )
    {
      // Set up text file path:

      string strReportPath = Path.Combine(
        Path.GetTempPath(), "PerformanceReport.txt" );

      FileStream fs = new FileStream( strReportPath,
        FileMode.OpenOrCreate, FileAccess.Write );

      StreamWriter streamWriter = new StreamWriter( fs );
      streamWriter.BaseStream.Seek( 0, SeekOrigin.End );

      // Sort output by percentage of total time used:

      List<string> lines = new List<string>(
        \_collection.Count );

      foreach( KeyValuePair<string, Entry> pair
        in \_collection )
      {
        Entry e = pair.Value;

        lines.Add( string.Format(
          "{0,10:0.00}%{1,10:0.00}{2,8}   {3}",
          GetPercent( e.Time, totalTime ),
          Math.Round( e.Time, 2 ),
          e.Calls,
          pair.Key ) );
      }
      lines.Sort();

      string header
        = " Percentage   Seconds   Calls   Process";

      int n = Math.Max( header.Length,
        lines.Max<string>( x => x.Length ) );

      if( null != description
        && 0 < description.Length )
      {
        n = Math.Max( n, description.Length );
        header = description + "\r\n" + header;
      }
      string separator = "-";
      while( 0 < n-- )
      {
        separator += "-";
      }
      streamWriter.WriteLine( separator );
      streamWriter.WriteLine( header );
      streamWriter.WriteLine( separator );

      foreach( string line in lines )
      {
        streamWriter.WriteLine( line );
      }
      streamWriter.WriteLine( separator + "\r\n" );
      streamWriter.Close();
      fs.Close();
      Process.Start( strReportPath );
      \_collection.Clear();
    }
  }
  #endregion // Internal TimeRegistry class

  string \_key;
  Stopwatch \_timer;
  double \_duration = 0;

  /// <summary>
  /// Performance timer constructor.
  /// </summary>
  /// <param name="what\_are\_we\_testing\_here">
  /// Key describing code to be timed</param>
  public JtTimer( string what\_are\_we\_testing\_here )
  {
    Restart( what\_are\_we\_testing\_here );
  }

  /// <summary>
  /// Automatic disposal when the the using statement
  /// block finishes: the timer is stopped and the
  /// time is registered.
  /// </summary>
  void IDisposable.Dispose()
  {
    Stop();
  }

  /// <summary>
  /// Write and display a report of the timing
  /// results in a text file.
  /// </summary>
  public void Report( string description )
  {
    TimeRegistry.WriteResults(
      description, \_duration );
  }

  /// <summary>
  /// Restart the measurement from scratch.
  /// </summary>
  public void Restart( string what\_are\_we\_testing\_here )
  {
    \_key = what\_are\_we\_testing\_here;
    \_timer = Stopwatch.StartNew();
  }

  /// <summary>
  /// Stop the timer.
  /// </summary>
  public void Stop()
  {
    \_timer.Stop();
    \_duration = \_timer.Elapsed.TotalSeconds;
    TimeRegistry.AddTime( \_key, \_duration );
  }
}
```

#### Benchmarking Instrumentation

To use the timer, I equipped various methods of interest with a using call to its constructor, e.g. like this:

```csharp
  /// <summary>
  /// Return the last sequence number.
  /// </summary>
  public int LastSequenceNumber
  {
    get
    {
      using( JtTimer pt = new JtTimer(
        "LastSequenceNumber" ) )
      {
        ChangeOptions opt = new ChangeOptions();

        CouchChanges<DbFurniture> changes
          = \_db.GetChanges<DbFurniture>( opt );

        CouchChangeResult<DbFurniture> r
          = changes.Results.Last<
            CouchChangeResult<DbFurniture>>();

        return r.Sequence;
      }
    }
  }
```

As you can see, I just add the using statement and a call to the JtTimer constructor to each method I would like to track performance for, nothing else.

All the rest is handled automatically by the JtTimer destructor and other internal methods.

I also added a call to instantiate a top-level timer keeping track of the total time and reporting results each time subscription is started and stopped, respectively, like this:

```csharp
class App : IExternalApplication
{
  /// <summary>
  /// Subscription debugging timer.
  /// </summary>
  static JtTimer \_timer = null;

  /// <summary>
  /// Toggle on and off subscription to
  /// automatic cloud updates.
  /// </summary>
  public static void ToggleSubscription(
    EventHandler<IdlingEventArgs> handler )
  {
    if( Subscribed )
    {
      Debug.Print( "Unsubscribing..." );
      \_uiapp.Idling -= \_handler;
      \_handler = null;
      \_buttons[3].ItemText = \_subscribe;
      \_timer.Stop();
      \_timer.Report( "Subscription timing" );
      \_timer = null;
      Debug.Print( "Unsubscribed." );
    }
    else
    {
      Debug.Print( "Subscribing..." );
      \_uiapp.Idling += handler;
      \_handler = handler;
      \_buttons[3].ItemText = \_unsubscribe;
      \_timer = new JtTimer( "Subscription" );
      Debug.Print( "Subscribed." );
    }
  }
```

Here are all the occurrences of the string "JtTimer" in the entire project (copy and paste somewhere or view source to see truncated lines in full):

```
Find all "JtTimer", Find Results 1, "Entire Solution", "*.cs"
  App.cs(34):           static JtTimer _timer = null;
  App.cs(234):          _timer = new JtTimer( "Subscription" );
  CmdSubscribe.cs(43):  using( JtTimer pt = new JtTimer( "OnIdling" ) )
  DbUpdater.cs(59):     using( JtTimer pt = new JtTimer( "DbUpdater ctor" ) )
  DbUpdater.cs(166):    using( JtTimer pt = new JtTimer( "UpdateBim" ) )
  JtTimer.cs(3):        // JtTimer.cs - performance profiling timer
  JtTimer.cs(25):       public class JtTimer : IDisposable
  JtTimer.cs(157):      public JtTimer( string what_are_we_testing_here )
  Properties\AssemblyInfo.cs(61):  // 2013-11-18 - 2014.0.0.17 - added JtTimer
  RoomEditorDb.cs(24):  using( JtTimer pt = new JtTimer( "RoomEditorDb ctor" ) )
  RoomEditorDb.cs(52):  using( JtTimer pt = new JtTimer( "LastSequenceNumber" ) )
  RoomEditorDb.cs(81):  using( JtTimer pt = new JtTimer( "LastSequenceNumberChanged" ) )
  Matching lines: 12
  Matching files: 6
  Total files searched: 21
```

#### Initial Benchmarking Results and Conclusion

In the beginning, I was not stopping the top-level timer properly, so I had no grand total, and therefore no percentages.

The very first result looked like this, with just two timers active, showing that I was headed in the right direction:

```
------------------------------------------
Subscription timing
 Percentage   Seconds   Calls   Process
------------------------------------------
      0.00%      0.04       1   UpdateBim
      0.00%     41.73     420   OnIdling
------------------------------------------
```

This already tells me that the Idling event handler is consuming a lot of time, whereas the actual BIM update is quick.

Adding a few more timers here and there starts producing useful results:

```
---------------------------------------------------
Subscription timing
 Percentage   Seconds   Calls   Process
---------------------------------------------------
      0.00%      0.00       1   DbUpdater ctor
      0.00%      0.05       1   UpdateBim
      0.00%      3.27      51   RoomEditorDb ctor
      0.00%     76.40     480   OnIdling
      0.00%     80.35      49   LastSequenceNumber
---------------------------------------------------
```

The LastSequenceNumber property is eating up absolutely all the time!

It is called once before we start subscribing to the Idling event, and then from inside the event handler.

All of the Idling event handler time is consumed by calls to this property.

I already showed the method implementation as an example of the [benchmarking instrumentation](#5).

Looking more closely at this method, I noted that it retrieves ***all*** the furniture documents from the database in order to determine the most recent sequence number.
The sequence number is used to afterwards retrieve all database changes that occurred after a certain point.

This call will obviously take longer and longer time the more entries we add to the database.

#### Improving the CouchDB Sequence Number Query

In the calls made to the LastSequenceNumber property from the Idling event handler, it is used only to check whether any new changes occurred.

At that point, we already have an initial sequence number to start from, and can ask the database to return only changes that occurred after that point in time, using a 'since' argument, e.g. like this:

```csharp
  /// <summary>
  /// Determine whether the given sequence number
  /// matches the most up-to-date status.
  /// </summary>
  public bool LastSequenceNumberChanged( int since )
  {
    using( JtTimer pt = new JtTimer(
      "LastSequenceNumberChanged" ) )
    {
      ChangeOptions opt = new ChangeOptions();

      opt.Since = since;
      opt.IncludeDocs = false;

      CouchChanges<DbFurniture> changes
        = \_db.GetChanges<DbFurniture>( opt );

      CouchChangeResult<DbFurniture> r
        = changes.Results.LastOrDefault<
          CouchChangeResult<DbFurniture>>();

      Debug.Assert( null == r || since < r.Sequence,
        "expected monotone growing sequence number" );

      return null != r && since < r.Sequence;
    }
  }
```

Replacing the call to the LastSequenceNumber property by a call to the new LastSequenceNumberChanged method in the Idling event handler shows a huge improvement:

```
----------------------------------------------------------
Subscription timing
 Percentage   Seconds   Calls   Process
----------------------------------------------------------
      0.00%      0.00       1   DbUpdater ctor
      0.00%      0.11       1   UpdateBim
      0.00%      3.25    1741   RoomEditorDb ctor
      0.00%      5.35       1   LastSequenceNumber
      0.00%     19.79    1738   LastSequenceNumberChanged
      0.00%     23.55    1951   OnIdling
----------------------------------------------------------
```

Now a much larger number of calls to the RoomEditorDb constructor and the LastSequenceNumberChanged method can be completed in much less time.

The one and only initial call to LastSequenceNumber is still awfully expensive and could probably also be improved, but the LastSequenceNumberChanged method is quick, and the whole Idling handling is immediately much more responsive.

#### Twiddling with Idling Responsiveness

I added the Stop method to the timer class and call when unsubscribing from Idling, as shown above in the [benchmarking instrumentation](#5) code.

That switches on the top-level timer providing the 100% measurement and enables the percentage calculations, like this:

```
----------------------------------------------------------
Subscription timing
 Percentage   Seconds   Calls   Process
----------------------------------------------------------
      0.00%      0.00       1   DbUpdater ctor
      0.25%      0.05       1   UpdateBim
      8.85%      1.67     486   LastSequenceNumberChanged
     16.06%      3.03     489   RoomEditorDb ctor
     17.05%      3.22   48667   OnIdling
     27.89%      5.27       1   LastSequenceNumber
    100.00%     18.89       1   Subscription
----------------------------------------------------------
```

With a more responsive behaviour, I am able to experiment a bit.
For instance, I removed the stopwatch in the Idling event handler again, call SetRaiseWithoutDelay immediately, and ignore 99 out of hundred calls to the Idling event handler.
As you can see, the Idling handler is called a hundred times for each check of the LastSequenceNumberChanged property:

```
----------------------------------------------------------
Subscription timing
 Percentage   Seconds   Calls   Process
----------------------------------------------------------
      0.00%      0.00       2   DbUpdater ctor
      0.11%      0.08       2   UpdateBim
      4.46%      3.23    1693   RoomEditorDb ctor
      8.35%      6.05       1   LastSequenceNumber
     21.72%     15.73    1689   LastSequenceNumberChanged
     27.39%     19.84  168900   OnIdling
    100.00%     72.44       1   Subscription
----------------------------------------------------------
```

The one single call to the LastSequenceNumber property costs more time than almost 500 calls to LastSequenceNumberChanged and almost 50000 calls to the Idling event handler.

Whew.

#### Download

This application lives in the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp) and
the versions discussed above are:

- [2014.0.0.16](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.0.16) with
  the unsubscription fixed as described in the discussion of
  [singleton application versus multiple command instances](http://thebuildingcoder.typepad.com/blog/2013/11/singleton-application-versus-multiple-command-instances.html).
- [2014.0.0.17](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.0.17) with
  added benchmarking timer and code, new JtTimer class, RoomEditorDb.LastSequenceNumber identified as the culprit, LastSequenceNumberChanged implemented, which only retrieves new changes and saves a huge amount of time, so the Idling event handling is now responsive and snappy.

I hope that this is of interest to you for numerous reasons.

The most important lesson should be: please benchmark your code!