---
post_number: "0648"
title: "Modeless Forms in Revit"
slug: "modeless_form"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'revit-api', 'views']
source_file: "0648_modeless_form.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0648_modeless_form.html"
---

### Modeless Forms in Revit

Today the moon is full.
I celebrated it last night already, sitting with a friend and having a fire on a hill with a 270 degree view to the east, west and south.
Very beautiful!

This full moon is special, because it is the occasion of the Chinese
[Mid-Autumn Festival](http://en.wikipedia.org/wiki/Mid-Autumn_Festival).
It "parallels the autumnal equinox of the solar calendar, when the moon is at its fullest and roundest".

![Mid-autumn full moon festival 2011](img/mid-autumn_full_moon_festival_2011.jpg)

This Chinese festival will even be celebrated in Switzerland tonight, as the
[Mondefest Basel 2011](http://www.basel.ch/mondfest_plakat_web2.pdf),
to honour the city partnership of Basel and Shanghai.

#### Accessing Revit from Outside an API Call-back Context

Back to the Revit API, here is a question that keeps cropping up again and again, prompting a summary of some basic aspects of interacting with Revit from a modeless form or external application, with an overview and pointers back to some of the previous posts and samples concerning this.

**Question:** How can I interact with Revit from a modeless form or external application?
I need to be able to switch back and forth between Revit and my form without closing it.

**Answer:** The Revit API is entirely designed to work only within pre-defined call-backs issued by Revit.

Therefore, you can only keep your form open while continuing to work in Revit in either one of two ways:

- Display your form from a Revit external command, but make it modeless. Then the command can complete and return control to Revit, while the form remains visible. As soon as the command has returned, though, you can no longer make use of the Revit API.- Display your form from an external application, not from a Revit external command. In that case you obviously also have no access to the Revit API.

The reasons and more background information on the current situation are given in the discussions of
[asynchronous API calls and idling](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html),
[modeless door lister flaws](http://thebuildingcoder.typepad.com/blog/2011/01/modeless-door-lister-flaws.html), and
the further posts that they point to.

As explained there, you cannot make use of the Revit API from an external application or a modeless context.

However, the API also provides the possibility to implement a workaround for this limitation, the Idling event.
It enables you to drive Revit from outside indirectly by posting an event to your Idling event handler, which has full access to the API and complete read and write access to the entire application object and all its documents.

To use this, subscribe to the Revit Idling event and raise a signal from your external application or modeless dialogue. In the Idling event handler, check for the raised signal and execute whatever functionality you need. How this can be done is demonstrated and described in full detail in my sample to
[display a live webcam image on a building element face](http://thebuildingcoder.typepad.com/blog/2010/06/display-webcam-image-on-building-element-face.html).

Another example, where Revit is driven from a modeless dialogue box, e.g. from a context outside of any Revit API call-back, also using the Idling event to "get back in" to the Revit API context, is given by my
[modeless loose connector navigator](http://thebuildingcoder.typepad.com/blog/2010/07/modeless-loose-connectors.html) sample.

The original version was written for Revit 2011. I recently
[updated and improved it for Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/08/modeless-loose-connector-navigator-update.html) as
well.

Additionally, a generic pattern for this process is described in
[pattern for semi asynchronous idling API access](http://thebuildingcoder.typepad.com/blog/2010/11/pattern-for-semi-asynchronous-idling-api-access.html).

Unfortunately, like any other call-back, Idling costs time and should therefore obviously be used with great care.
Use it sparingly and cautiously.