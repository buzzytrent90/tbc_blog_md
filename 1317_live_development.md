---
post_number: "1317"
title: "Live Development and a Share Bar"
slug: "live_development"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'python', 'revit-api', 'vbnet']
source_file: "1317_live_development.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1317_live_development.html"
---

### Live Development and a Share Bar

Let's reiterate a question that comes up quite regularly, on [live development](#3), i.e. debugging and editing your code without having to restart Revit and reload a model every time you make a change.

First, however, let me say a few words about the new [share bar](#2).

#### The Building Coder Share Bar

I spent a couple of hours last night adding the share bar that you now see on the right hand side.

Kean Walmsley
[added a ShareThis button to Through the Interface](http://through-the-interface.typepad.com/through_the_interface/2009/07/sharethis-button-added-to-through-the-interface.html) way
back in 2009, and I only just noticed.

I went to the [ShareThis web site](http://www.sharethis.com) and
attempted to use the default TypePad code to add it automatically.

Alas, that does not work for The Building Coder.
It reports that 'This blog uses a design which doesn’t support widgets at this time', and I was left to figure the rest out on my own.

I solved this by requesting code for any web page.
That returns two snippets of code, a header and a footer part, looking like this:

Header:

```
<script type="text/javascript">var switchTo5x=true;</script>
<script type="text/javascript" src="http://w.sharethis.com/button/buttons.js"></script>
<script type="text/javascript" src="http://s.sharethis.com/loader.js"></script>
```

Footer:

```
<script type="text/javascript">
stLight.options({
  publisher: "20197c6e-9ed3-4217-bc73-7690e00d2456",
  doNotHash: false, doNotCopy: false, hashAddressBar: false});
</script>
<script>
var options={
  "publisher": "20197c6e-9ed3-4217-bc73-7690e00d2456",
  "position": "right",
  "ad": { "visible": false, "openDelay": 5, "closeDelay": 0},
  "chicklets": { "items": ["twitter", "facebook", "googleplus",
    "linkedin", "pinterest", "email", "sharethis"]}};
var st_hover_widget = new sharethis.widgets.hoverbuttons(options);
</script>
```

Analysing Kean's blog, I see that his ShareThis code snippet is placed in the right-hand sidebar and thus appears automatically by inclusion in all relevant pages.

Unfortunately, that did not work in my case, and I had to add the two snippets separately in all the relevant template pages.

My entire list of blog page templates looks like this:

![Typepad archive templates](img/typepad_archive_templates.png)

Out of these, I added the header and footer snippets to the following:

- Archive Index Template
- Main Index Template
- Master Post Index
- Reverse Post Index
- Category Archives
- DateBased Archives
- Individual Archives
- Pages

Unfortunately, the buttons for sharing via email and ShareThis did not work for me.

They just greyed out the screen and nothing further happened.

So I took the easy way out and removed those two.

I also noticed that the Twitter button by default automatically adds a suffix saying '... via @sharethis'.

I modified that to refer to me instead by adding the `chicklets_params` entry as described in these instructions on how to
[remove @sharethis twitter tweet](https://wordpress.org/support/topic/remove-sharethis-twitter-tweet).

My final footer code snippet now looks like this:

```
<script type="text/javascript">
stLight.options({
  publisher: "20197c6e-9ed3-4217-bc73-7690e00d2456",
  doNotHash: true, doNotCopy: true, hashAddressBar: true});
</script>
<script>
var options={
  "publisher": "20197c6e-9ed3-4217-bc73-7690e00d2456",
  "position": "right",
  "ad": { "visible": false, "openDelay": 5, "closeDelay": 0},
  "chicklets": { "items": ["twitter", "facebook", "googleplus",
    "linkedin", "pinterest", "email", "sharethis"]},
  chicklets_params: { twitter:{ "st_via":"jeremytammik" } } };
var st_hover_widget = new sharethis.widgets.hoverbuttons(options);
</script>
```

I hope you like the new sharing options and make good use of them   :-)

Back to the important topic of Revit API live development and debugging:

#### Live Development

This topic was raised once again by the following query:

**Question:**
I have a general question (and hopefully not too vague!)

I've become very accustomed to the development approach offered by the Legacy COM API, namely that of "live" interaction with AutoCAD. I could run AutoCAD from an Automation client (in this case from a Common Lisp Listener) and send instructions to AutoCAD to draw lines, change Zoom extents, so on and so forth. The advent of .NET appears to have shut the door on this sort of development approach, particularly with respect to Revit.

So far as I understand there are only two general approaches available, in order to interact "live" with Revit while developing a .NET program.

First, is to run a Debug session from Visual Studio, attached to Revit, and step through lines of C# or VB.NET, for example.

Second, is to embed a scripting language (an example being the Python Shell) and interact with the application from a custom IDE.

Is there anything else that people are doing? Consider for example the Lisp dialect ClojureCLR (the .NET port of Clojure). I'd love to be able to work in a Clojure IDE and somehow interface with Revit (or AutoCAD) in a way reminiscent of the Legacy COM approach. Is there something people are doing with the DLR, for instance, to achieve that sort of "live" coding, or is this all a .NET 'pipe dream'?   :^)

Thanks.

**Answer:**
Thank you for your very pertinent query.

This has been a topic of lively discussion and experimentation for years, with numerous solutions from various sides.

Some people have succeeded in making use of the Visual Studio 'live editing and debugging' functionality for the Revit API.

The most recent reports I saw on this were based on Visual Studio 2013, which I have not installed.

Other attempts made use of the Add-In Manager or reloading the add-in DLL from memory via reflection instead of a disk file.

I pointed out some of these various solutions on The Building Coder blog now and then.

It does not work out of the box, however.

Here are several examples and related discussions that might help:

- [Reload Add-in for Debug Without Restart](http://thebuildingcoder.typepad.com/blog/2012/12/reload-add-in-for-debug-without-restart.html)
- [Load Your Own External Command on the Fly](http://thebuildingcoder.typepad.com/blog/2013/05/load-your-own-external-command-on-the-fly.html)
- [Reloading and Debugging External Commands on the Fly](http://thebuildingcoder.typepad.com/blog/2013/05/reloading-and-debugging-external-commands-on-the-fly.html)
- [RevitRubyShell for Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/05/revitrubyshell-for-revit-2014.html)
- [Revit Add-in Unit Testing](http://thebuildingcoder.typepad.com/blog/2013/07/revit-add-in-unit-testing.html)
- [The Dynamo Revit Unit Test Framework](http://thebuildingcoder.typepad.com/blog/2013/10/the-dynamo-revit-unit-test-framework.html)
- [Intimate Revit Database Exploration with the Python Shell](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html)
- [Debugging Revit 2014 API with Visual Studio 2013](http://thebuildingcoder.typepad.com/blog/2013/11/debugging-revit-2014-api-with-visual-studio-2013.html)
- [Visual Studio 2013 May Partially Support Edit and Continue](http://thebuildingcoder.typepad.com/blog/2013/12/visual-studio-2013-may-partially-support-edit-and-continue.html)

If you can get the Visual Studio edit and continue functionality to work for you, that is perfect.

I can very highly recommend making use of the Revit Python Shell, because I have used it myself and it is super easy to set up.

The Revit Ruby Shell is probably equally good and based on a more modern language.

Not having any Ruby experience, though, I have stuck with the Python shell so far.

The most reliable and only officially supported method that I am aware of is to develop and debug your code as a macro in the SharpDevelop IDE and then convert it to an add-in DLL once you are done.

I hope this helps.

I really look forward to hearing what solutions for this people are currently working with!