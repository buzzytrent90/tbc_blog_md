---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 12.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.317430'
original_url: https://thebuildingcoder.typepad.com/blog/1107_api_features.html
post_number: '1107'
reading_time_minutes: 16
series: general
slug: api_features
source_file: 1107_api_features.htm
tags:
- csharp
- elements
- geometry
- levels
- parameters
- revit-api
- vbnet
- views
- windows
title: A Couple of Revit API Aspects and Features
word_count: 3159
---

### A Couple of Revit API Aspects and Features

The two most common recommendations I have for newbie Revit API developers are:

- Understand the Revit end user product paradigm, workflow and best practices
- Forget everything (well, some things) you know about any other CAD system programming

I just gave a Revit API "beginners" workshop to a bunch of experiences programmers.

They had already enjoyed an end user product training to cover the first item above.

They had also had a month to digest the Revit API
[getting started](http://thebuildingcoder.typepad.com/blog/about-the-author.html#2) material.

On that basis, we were able to cover absolutely all aspects of the Revit API in full depth in just two days, including creating new little samples to explore programmatic section view creation and positioning, Idling and external event handling, and the dynamic model updater framework DMU.
I hope to be able to clean up and discuss these nice new minimal samples soon.

Anyway, that removed all doubt in my mind – as if I ever had had any – that the Revit API is basically small and simple.

Still, views on that may differ widely and are sometimes vehemently expressed, as you can see below.

A whole bunch of interesting issues related to the design of the Revit API itself came up recently in several contexts, so let's collect some poignant statements and illuminating revelations addressing them right here and now:

A whole bunch of interesting issues related to the design of the Revit API itself came up in several different contexts recently, so let's collect some poignant statements and illuminating revelations addressing them right here and now:

- [The Revit API is an abomination](#2)
- [Revit API compatibility across versions](#3)
- [Update release may solve some issues](#4)
- [Migration](#5) – upgrading code from Revit 2013 to 2014
- [Catching and modifying the Revit save as event](#6)

#### The Revit API is an Abomination

This rather poignant verdict was submitted last week by Adrian Hodos in his
[comment](http://thebuildingcoder.typepad.com/blog/2014/01/no-inheritance-and-no-strong-naming.html?cid=6a00e553e16897883301a73d77a325970d#comment-6a00e553e16897883301a73d77a325970d) on
the lack of
[inheritance and strongly named .NET assemblies](http://thebuildingcoder.typepad.com/blog/2014/01/no-inheritance-and-no-strong-naming.html):
> To put it frankly, the Revit API is an abomination.
> It should be taught as a model of how not to program at CS schools everywhere.
> I could spend hours talking about the coding horrors one has to make just to do something trivial (in other CAD programs) like applying a transform to a solid, or infer a relation between a bunch of objects.
> And let's not forget about the "wise" decision of only allowing access from managed languages.
> This makes wonders for the performance of the code :)
> And don't even mention the word multi-threaded, it is an unknown concept in the Revit world.

All the aspects mentioned by Adrian are true, so please think twice before deciding to embark on a Revit API project.

The only hope we have is to try to gain some understanding and accept these facts.

Thankfully, this is vastly easier after reading and digesting the polite and insightful answers to Adrian's critique provided by Arnošt Löbel, Sr. Principal Engineer of the Revit API development team:

I hear you, Adrian, I really do. However, I would like you to (try) understand that there are always important reasons for the decisions the Revit R&D makes and serious considerations are always part of the decision process. For one, we believe that having a .NET API shell over Revit native internal provides most users with more stable platform, and that applies to Revit API developers as well as the end users that enjoy their add-ins. In my personal opinion, the .NET environment makes the API also more approachable to users that aren't professional developers. At the same time, we do not restrict what advanced developers can do in their add-ins. If they wish to delegate some heavy calculations to other modules written in unmanaged languages, they quite easily may do that. In fact, Revit supplies some of its own add-ons that are written as multi-language, the cloud renderer being one example.

As for multithreading, I actually do not believe our users think we ignore their wishes and requests out of spite. I know they understand that putting together a multithreaded environment is not trivial. It is not easy to write such applications from scratch. It is far more difficult to rewrite an old application to be completely multithreaded. And even with that difficulty being always part of the equation, some applications are just mode difficult to multithread than others. I am not saying impossible, but sometimes it is just too darn hard. Revit actually has many internal components that are threaded; however, trying to thread access to a database as complex and tiered as Revit's data model is another task all together.

I'd like to add one comment about the complexity (and, well, ugliness) of the Revit API. I think the same could be said about Revit itself when comparing it with other applications. Revit is indeed different. I do believe there are trivial tasks that would take minutes to do in AutoCAD while they can take an hour easily in Revit. On the other hand, however, I would not have to be pressed hard to come up with tasks that take minutes in Revit, but days in AutoCAD. I believe the same sentiment can be extended to other platforms and applications. There are users who swear to be much faster on their Macs then others can be on their Windows, and vice versa. And of course, the Unix hard-core crowd swears to be much more efficient with their command lines than all the GUI lovers combined. :-) But at the end of the day it is all about the right tool for the right tasks, isn't it.

The Revit API may not be perfect.
There are and always will be plenty of things we would like to be different, and would be, if we had our way and time to work on them.

Many thanks both to Adrian for putting these facts and his feelings so clearly and bluntly, and to Arnošt for picking them up in the same vein.

I hope and believe that this will prove really useful to anybody faced with these issues, whatever their reactions and thoughts may be.

#### Revit API Compatibility Across Versions

Another aspect of the Revit API that is disconcerting to some developers is its lack of backwards compatibility.

Unlike AutoCAD, which provides backwards API compatibility across years and even decades, the Revit API sometimes deprecates certain sub-optimal properties, methods and classes one year and removes them the next, triggering questions such as this one on
[Revit API compatibility across versions](http://forums.autodesk.com/t5/Revit-API/Revit-API-Compatibility-Across-Versions/m-p/4818483) raised
in the Revit API discussion forum by Terry Dotson of
[DotSoft](http://www.dotsoft.com):

**Question:** We have been developing add-ons for AutoCAD for 20+ years and are thinking about testing the water in Revit, mainly the Architectural version.

If a .NET based plugin is produced for a given version, will it typically work in future releases?

...

**Answer:** I am sorry to say that the answer to your first and main question is simply no.

Generally, a Revit add-in requires recompilation to run on a new major release of Revit.

Say you have implemented and installed your add-in for and on Revit A, and the user updates to a new major release Revit B.

Due to the fact that the .NET framework isolates the add-in from the underlying API to a certain extent, all the functionality in the Revit B API that remains unchanged from Revit A will continue working.

However, unlike AutoCAD, Revit does not maintain backwards API compatibility.

For instance, access to the endpoints of a curve element used to be via the indexed EndPoint property, taking an argument 0 for the start and 1 for the end endpoint.

In C#, this property was automatically translated to a method get\_EndPoint, whereas in Visual Basic, it remained unchanged, an indexed property EndPoint.

In Revit 2014, this property was declared deprecated and replaced by a new method GetEndPoint that is and remains a method under all circumstances and in all languages.

In the next major release of Revit, the deprecated method will be declared obsolete and removed.

From that moment onward, a Revit 2012 add-in trying to access the EndPoint property (or get\_EndPoint method in C#) will cease working.

So Revit provides much less backwards API compatibility than AutoCAD.

Users are encouraged to update in a more timely fashion.

One important tip for AutoCAD developers is to try to
[let go of the expectations](http://www.marcandangel.com/2013/09/02/5-things-you-should-know-about-letting-go) that working with AutoCAD APIs has fostered ...
[Revit is a different animal](http://thebuildingcoder.typepad.com/blog/2012/02/bim-versus-free-geometry-and-product-training.html) and
the experience and expectations coming from any other CAD system can be just as much a hindrance as an asset, especially when thinking about
[porting an AutoCAD application](http://thebuildingcoder.typepad.com/blog/2012/10/porting-an-autocad-application.html).
Basically, the workflows are so different that a straight port will almost never make any sense whatsoever.

On the other hand, the Revit API is much smaller and simpler than AutoCAD's.
An experienced programmer should be able to get going within a few hours, and understand everything there is to know within a few days, which is definitely not possible in AutoCAD.

One last thing: in Revit add-in development, it is more important than in AutoCAD to have a good product and user workflow understanding before beginning to think about any application development, both because the workflows in real BIM projects are more complex, and because the API more closely mimics and supports the user interface functionality. As always, it is important to go with the flow, and not fight against it.

Oh yes, yet another last thing: an example of setting up a Visual Studio project to
[target multiple versions](http://thebuildingcoder.typepad.com/blog/2013/11/multi-version-visual-studio-revit-add-in-wizard.html) of
Revit simultaneously.
Actually, it provides even more, namely a flavour of
[The Building Coder Visual Studio add-in wizard](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.20) to
automatically do it for you.

#### Revit Update Release May Solve Some Issues

Sometimes the mid-cycle Revit update releases include API enhancements that solve certain issues.

For example, a developer recently asked the following question, then provided the answer himself:

**Question:** I am using a dockable panel and I need to change the ActiveView.
I subscribe to the external events framework to help me to access the active view and get into a valid Revit API context.
This works fine in first change, but after that the ActiveView property returns null and I get an ArgumentException saying "Value cannot be null. Parameter name: Element)".

I used the exact same code in a modeless form, and it works fine there.
What might be the problem, please?

**Answer:** I am glad to report that I updated to the
[Revit 2014 update release 2](http://thebuildingcoder.typepad.com/blog/2013/11/revit-2014-update-release-2.html),
UR2, and it works fine now.

#### Migration – Upgrading Code from Revit 2013 to 2014

The issues that arise when migrating code from one version of Revit to the next are typically pretty trivial.

In general, most of them can easily be avoided completely by ensuring that your code compiles on the current version without generating any warnings before starting to migrate it to the next version, as demonstrated by
[eliminating compiler warnings and deprecated calls](http://thebuildingcoder.typepad.com/blog/2013/02/eliminating-compiler-warnings-and-deprecated-calls.html) for
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples),
once
[at the beginning of the Revit 2014 release cycle](http://thebuildingcoder.typepad.com/blog/2012/01/eliminating-compiler-warnings-and-deprecated-calls.html) and
again
[later on](http://thebuildingcoder.typepad.com/blog/2013/02/eliminating-compiler-warnings-and-deprecated-calls.html),
to clean up.

Here are some more examples of typical simple (not to say trivial) migration issues:

- [Access to the material cut pattern name](#5.1)
- [Replace Transform Rotation by CreateRotationAtPoint](#5.2)
- [Replace Level property by LevelId](#5.3)

##### Access to the Material Cut Pattern Name

**Question:** CutPattern and SurfacePattern.
Previously, this code worked fine:

```csharp
Dim Mat As Material = TryCast(e, Material)
Mat.CutPattern.Name
```

It fails in Revit 2014.
How would you access the Name property now?

**Answer:** You can obtain the cut pattern element id from the Material.CutPatternId property, call the Document.GetElement method on that to obtain the cut pattern element itself, and query name of the cut pattern from its Name property.

Actually, the warning message generated by the compiler and the Revit 2014 API help file RevitAPI.chm documentation all remind you very obviously of the equivalent methods and properties to use.

For example, the explanation of the CutPattern of Material class states:

```csharp
[ObsoleteAttribute("This property is obsolete in
Autodesk Revit 2014. Please use the CutPatternId
property instead.")]
public FillPatternElement CutPattern { get; }
```

##### Replace Transform Rotation by CreateRotationAtPoint

**Question:** I have the following Revit 2013 code:

```csharp
Dim dXpos As ProjectPosition
= lDoc.ActiveProjectLocation.ProjectPosition(
XYZ.Zero)
Dim LtLtransR As Transform = Transform.Rotation(
XYZ.Zero, XYZ.BasisZ, dXpos.Angle)
```

It causes an error when I change it to the following:

```csharp
Dim LtLtransR As Transform
= Transform.CreateRotationAtPoint(
XYZ.Zero, XYZ.BasisZ, dXpos.Angle)
```

The error shows up for XYZ.BasisZ, saying "Value of type 'Autodesk.Revit.DB.XYZ' cannot be converted to 'Double'."

It also complains about dXpos.Angle: "Value of type 'Double' cannot be converted to 'Autodesk.Revit.DB.XYZ'."

How can this be solved?

**Answer:** The second argument to the CreateRotationAtPoint method is an angle, which is passed in as a `double` real number.
You passed in a point XYZ instead.
Please replace it with a double value.

By the way, the first argument to CreateRotationAtPoint is the rotation axis, so it cannot be a zero point, and you need to fix that as well for it to work.

##### Replace Level Property by LevelId

**Question:** My code is causing a warning saying "Public Overridable ReadOnly Property Level As Autodesk.Revit.DB.Level' is obsolete: 'This property is obsolete in Revit 2014. Call Element.LevelId instead.'"

How would I go about to get Level.Name using LevelId?

**Answer:** Like your first question, retrieve the level object by the LevelId, then get the level element.
Here is the short code to do so:

```csharp
ElementId levelId = elem.LevelId;
Level level = Doc.GetElement(levelId);
string levelName = level.Name;
```

#### Catching and Modifying the Revit Save As Event

As said, the Revit API is small and simple.

Therefore, there are obviously huge areas that it does not cover at all.

Sometimes, we can take recourse to the .NET framework libraries or Windows API to address requirements beyond the scope of the Revit API.

**Question:** I'm trying to catch the save as dialog in Revit.
I want to override it with my own dialogue and, if the user presses 'cancel', still show the standard Revit save as dialogue.

I added the event DialogBoxShowing but it's not catching the save as dialog event.
I googled the question and discover that others have encountered the same issue, like in this discussion on
[TaskDialog vs DialogBox, Id vs HelpId, and DialogBoxShowing Event](http://spiderinnet.typepad.com/blog/2011/07/taskdialog-vs-dialogbox-id-vs-helpid-and-dialogboxshowing-event.html),
which states that only certain Revit dialogues trigger events that can be subscribed to.

Is there any other way I can override the save as dialogue in Revit?

**Answer:** Thank you for your query and the pointer to the nice Spiderinnet article.

I do not believe that the Revit save as dialog is handled like a normal task dialogue or covered by the DialogBoxShowing event.

You therefore probably have to resort to the native Windows API or the .NET framework libraries to access and modify it and its behaviour.

I described the three possible levels of catching and handling dialogues in Revit several times, e.g. in the post on
[dismissing a dialogue using the Windows API](http://thebuildingcoder.typepad.com/blog/2009/10/dismiss-dialogue-using-windows-api.html).

The third and most hard-core method uses the Windows API and a Windows hook, enabling you to catch absolutely any dialogue whatsoever on your system.

However, there are also simpler and more targeted standard Windows methods to modify only the behaviour of the existing save as dialogue instead of trapping absolutely all dialogues.

Here are a bunch of pointers to other
[posts related to the DialogBoxShowing event](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32) with
examples and other ways to handle dialogues programmatically, but they will not provide much in the way of customisation of the save as dialogue.

By the way, depending on what your goal is, you might also be interested in simply trapping and handling the one of the various simple Revit API events triggered by the document saving itself, e.g., DocumentClosed, DocumentClosing, DocumentSavedAs, DocumentSaved, DocumentSavingAs, DocumentSaving, FileExported, FileExporting, etc.

Back to the save as dialogue and the Windows API: the SaveFileDialog is part of the operating system Common Dialog Box Library and will actually look different on different versions of the OS. There are lots of ways to
[customise common dialog boxes](http://msdn.microsoft.com/en-us/library/ms646960(VS.85).aspx) using
the native Win32 API.
Actually, maybe all you need is a Windows hook and catching the CDN\_FILEOK message.

By the way, Windows Vista introduced a new
[Common Item Dialog](http://msdn.microsoft.com/en-us/library/bb776913%28VS.85%29.aspx) providing
a COM based API and making it easier to use in a Windows Forms application.

Here are some rather dated in-depth explanations and explorations of all aspects of this kind of customisation:

- [Implementing a Read-Only 'File Open' or 'File Save' Common Dialog](http://www.codeproject.com/Articles/5782/Implementing-a-Read-Only-File-Open-or-File-Save-Co) by
  dkotchan, 26 Jan 2004
- [Vista Goodies in C++: Using the New Vista File Dialogs](http://www.codeproject.com/Articles/16678/Vista-Goodies-in-C-Using-the-New-Vista-File-Dialog) by
  Michael Dunn, 29 Dec 2006

I hope this helps, that the discussions above help you understand some new aspects of the Revit API and its philosophy, and that in turns helps you get started or continue using it happily and productively.