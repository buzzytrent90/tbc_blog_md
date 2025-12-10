---
post_number: "1100"
title: "Web Workshop, Tech Summit Plans and Security"
slug: "web_workshop"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'parameters', 'python', 'revit-api', 'rooms', 'schedules', 'views', 'walls', 'windows']
source_file: "1100_web_workshop.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1100_web_workshop.html"
---

### Web Workshop, Tech Summit Plans and Security

Last week, I mentioned my short visit to Gothenburg, Sweden, for a mini web workshop hosted by
[CAD-Q](http://www.cad-q.com) and
promised you more information about it anon.

Well, the time has come to summarise the results from that venue, and also start planning my internal Autodesk Tech Summit proposal, due today, February 7, at the latest.

I was a bit surprised by the contents of the web workshop.
I had expected to contribute my part, for an hour or two, and then hear something from the others as well.
It ended up running from Thursday at two in the afternoon until after six, and continuing on Friday.
Unfortunately, I had to return to Switzerland already Friday morning, and, unexpectedly, the one and only scheduled presenter for Thursday afternoon was me :-)

![Web workshop agenda](img/web_workshop_agenda_jt.png)

I ended up presenting on the following topics:

- [Autodesk 360 Web Services](#2)
- [REST API Programming](#3)
- [Revit and the Cloud](#4)
- [RoomEditor – my Cloud-based 2D BIM Model Editor](#5)
- [Future Plans and Ideas](#6)
- [WhiteHat Security Presentation](#7)

All except the last two have already been covered extensively on the blog, so I had no problem filling the four hours ad hoc.

#### Autodesk 360 Web Services

In preparation for this workshop, I put together an
[overview of the A360 web services](http://thebuildingcoder.typepad.com/blog/2014/01/lots-of-clouds.html#5) and
their API coverage last week, mainly gleaned from the
[cloud & mobile platform web service API overview](http://thebuildingcoder.typepad.com/blog/2013/12/devdayau-chronicle-estorage-view-depth-sound-of-noise.html#4) presented
at the DevDay@AU in December 2013.

As said, if you are interested in making use of any of the available APIs, please apply to join one of the
[pilot programs](http://adndevblog.typepad.com/aec/2013/10/bim-360-glue-api-pilot-and-updated-samples.html).

#### REST API Programming

Here are some of the explorations I made so far in various areas of REST API programming:

- [Revit Server REST API](http://thebuildingcoder.typepad.com/blog/2011/11/revit-server-rest-api.html)
- [Copy a model from a Revit Server](http://thebuildingcoder.typepad.com/blog/2011/12/copy-a-model-from-a-revit-server.html)
- [AU Class CP3093 – My first cloud/mobile app with Revit Server – by Adam Nagy](http://thebuildingcoder.typepad.com/blog/2012/11/au-classes-on-python-ui-server-and-framework-apis.html#3)
- [The BIM 360 Glue viewer and REST API](http://thebuildingcoder.typepad.com/blog/2012/12/the-bim-360-glue-viewer-and-rest-api.html)
- [BIM 360 Glue REST API authentication using Python](http://thebuildingcoder.typepad.com/blog/2012/12/bim-360-glue-rest-api-authentication-using-python.html)
- [The Green Building Studio GBS REST API](http://thebuildingcoder.typepad.com/blog/2013/01/url-and-other-buttons-xyz-points-and-vectors.html#6)
- [Cloud & mobile topics](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html)
- [Room editor project overview and CouchDB configuration](http://thebuildingcoder.typepad.com/blog/2013/04/room-editor-project-overview-and-couchdb-configuration.html)
- [CouchDB implementation](http://thebuildingcoder.typepad.com/blog/2013/05/couchdb-implementation-and-github-repository.html)
- [The Revit Server REST API](http://thebuildingcoder.typepad.com/blog/2013/08/the-revit-server-rest-api.html)
- [Revit Server API access and VBScript](http://thebuildingcoder.typepad.com/blog/2013/08/revit-server-api-access-and-vbscript.html)
- [BIM 360 Glue API pilot and updated samples](http://thebuildingcoder.typepad.com/blog/2013/10/text-file-driven-automatic-placement-of-family-instances.html#9)
- [Cloud & mobile platform web service APIs](http://thebuildingcoder.typepad.com/blog/2013/12/devdayau-chronicle-estorage-view-depth-sound-of-noise.html#4) presented at the DevDay@AU
- [ReCap web service & cloud-based round-trip model editing](http://thebuildingcoder.typepad.com/blog/2013/12/au-day-3-recap-cloud-based-round-trip-model-editing.html)
- [REST POST request to Revit Server 2014](http://thebuildingcoder.typepad.com/blog/2014/01/rest-post-request-to-revit-server-2014.html)

Wow, that is a bit more than I expected, actually.
Please note that most REST APIs are very simple to use.
Pick a service that is useful to you and start exploring!

#### Revit and the Cloud

Revit is evolving slowly but surely (well, actually not so slowly at all) towards a structure that can be split into separate components, and enables individual bits of functionality to be replaced by custom implementations, which may or may not be remote and cloud based, just as you prefer, e.g.:

- [User MEP calculations](http://thebuildingcoder.typepad.com/blog/2013/07/user-mep-calculation-sample.html)
  ([on GitHub](http://thebuildingcoder.typepad.com/blog/2013/11/user-mep-calculation-sample-on-github.html))
- [Building Performance Analysis](http://thebuildingcoder.typepad.com/blog/2013/12/starting-to-clean-up-for-the-break.html#3) web services
- [Daylighting analysis](http://thebuildingcoder.typepad.com/blog/2013/12/starting-to-clean-up-for-the-break.html#4)

I think we can expect the use of these kind of externalised services to continue growing rapidly, presenting lots of new customisation and maintenance enhancement opportunities.

#### RoomEditor – my Cloud-based 2D BIM Model Editor

Also in preparation for this workshop, I summarised the available source code and presentation recordings for my RoomEditor, a
[cloud-based real-time round-trip simplified 2D BIM model editor](http://thebuildingcoder.typepad.com/blog/2014/01/lots-of-clouds.html#4).

Most of the work I completed to implement it has been presented extensively by The Building Coder under the
[cloud](http://thebuildingcoder.typepad.com/blog/cloud),
[desktop](http://thebuildingcoder.typepad.com/blog/desktop) and
[mobile](http://thebuildingcoder.typepad.com/blog/mobile) categories.

#### Future Plans and Ideas

As said, I am still in the process of submitting a proposal for the internal Autodesk Tech Summit.

I have two ideas in mind:

- [Adding security related functionality to the existing RoomEditor](#6.1)
- [Implementing a new more generic cloud-based round-trip real-time simplified 2D Revit BIM editor](#6.2)

#### Adding Security to the Existing RoomEditor

I talked this over with my colleague Gopinath Taget.
Here is a quick summary of our discussion:

**Question:** What security options would you suggest adding to the room editor?

Currently, it consists of two components:

- Revit add-in, connecting to the cloud database via REST.
- Apache CouchDB cloud database with server-side scripting in HTML and JavaScript.

I have not set up any security at all yet.

When setting up a new CouchDB database, it is originally set up as a so-called
[admin party](http://guide.couchdb.org/draft/security.html),
a free-for-all setting with no access restrictions whatsoever.
Anybody can go to the cloud database and has complete admin rights.

The first step would obviously be to switch on users management and credential handling in the cloud database.

**Answer:** I think the basics would be:

- Separate login/password to the DB with enough rights to access, e.g. separate read and write permissions, distinct from the admin privileges.
- Using SSH to communicate with the REST API.

It is unclear how to secure the REST API itself though.
A web API can never be private, unlike a web page that can use login/password authentication.
At most, you can require public/private certificates to identify the user making the calls.

You could use certificates to identify the API user and certificates for SSH.

Of course, more complex the app, the more potential for security holes.

Two of the most common vulnerabilities for many web applications are SQL injection (if it uses stored procedures in databases) and cross-site scripting, i.e. someone intercepting the user communication and injecting malicious code into it.

I don't think there is any danger of this scenario in your architecture.

**Question:** No. 1 is definitely top of the list, as I already stated.

No. 2 also seems high priority and may be covered by CouchDB itself.

I have no SQL, since I am using a NoSQL database, so the injection issue is moot.

**Conclusion:** Not much to do, really, beyond setting up the access rights and credential handling on the database, which is awfully obvious and very standard.

So maybe I'll skip this idea and go for the second one instead:

#### Implementing a New More Generic BIM Editor

The current RoomEditor generates its own simplified 2D plan view by determining room boundaries and projecting 3D family instances onto the XY plane.

A more useful and generic approach might base a similar cloud-based real-time round-trip simplified 2D BIM model editor on selected plan views in Revit instead.
It might also add an option to include other additional non-graphical properties and parameters, besides the pure minimal graphical data required to interactively manipulate the furniture placement.

Here is a suggestion for such an approach by Samir Balicevac of
[CAD-Q](http://www.cad-q.com):

- In Revit, select the plan views and specific categories to export, e.g., walls, furniture, etc.
- Export the non-graphical data, e.g. properties and parameters, plus the graphical data to 2D SVG.
- Import the graphical and non-graphical data into a NoSQL cloud database using the Revit unique id as a common key.
- Display the 2D plans in a graphical viewer in a web browser.
- Implement picking an element in the viewer, selecting it and displaying a modal window with non-graphical data.
- The non-graphical data can be edited and the selected element location modified by dragging, updating the cloud database.
- Update the Revit BIM model from the cloud database, including both graphical and non-graphical data.

This has a strong similarity to the existing implementation, plus several important advantages, e.g.:

- There is a real use case and strong need for such functionality; almost any application developer could make use of such a component in one, several, or all typical workflows.
- Non-graphical data could be handled in a generic, customisable, flexible manner, in addition to the current graphical location and rotation functionality.

Well, I only have a few more hours until the tech summit proposal submission deadline, so I had better start deciding fast.

#### WhiteHat Security Presentation

Talking about web application security above, did you ever wonder how hackers find vulnerable websites?
Or how much your personal data is worth?

Here is a pretty short and very instructive presentation on these topics by Ashley Hamilton,
[WhiteHat Security](http://www.whitehatsec.com) Application Security Engineer, covering the whole range of security from exploit to marketplace and demonstrating:

- How hackers locate vulnerable websites
- The tools they use to find and search your database
- How hackers determine risk versus reward
- How they sell your data and how much you’re worth

Here are the direct links to the:

- [Webinar recording (38 minutes)](http://go.whitehatsec.com/675YBI6740001te004F9G00)
- [Presentation slide deck](http://go.whitehatsec.com/675YBI6740001tf004F9G00)