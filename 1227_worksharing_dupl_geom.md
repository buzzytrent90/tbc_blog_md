---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.555055'
original_url: https://thebuildingcoder.typepad.com/blog/1227_worksharing_dupl_geom.html
post_number: '1227'
reading_time_minutes: 11
series: general
slug: worksharing_dupl_geom
source_file: 1227_worksharing_dupl_geom.htm
tags:
- doors
- elements
- geometry
- parameters
- revit-api
- rooms
- sheets
- views
- walls
- windows
title: Worksharing and Duplicating Element Geometry
word_count: 2284
---

### Worksharing and Duplicating Element Geometry

I had two more interesting email conversations on Revit API questions, on
[add-ins in a worksharing environment](#2) and
[duplicating element geometry for detailing and fabrication](#3).

Actually, both of these topics have been discussed in the past, so a lot of the work I did was to dig out the existing material again to make these developer partners happy.

Today I therefore have an opportunity to talk about the Revit API and nothing but the Revit API again for a change, after all the web related stuff in the past few posts.

We will probably return to more web related stuff in the next few ones as well, since I am in Darmstadt right now, getting ready for my presentation on the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46),
*Interaktives 3D Modell in beliebige Webseiten einbetten*,
at
[Autodesk University Germany](https://cluster.ems-secure.de/registrations/autodesk/au.2014/ems.registration.php) here tomorrow.

Following that, next weekend, I will be attending the third and final hackathon this month in
[Berlin](http://www.meetup.com/TechMeetups-Berlin/events/161213342),
after spending the past two weekends at the hackathons in
[Zurich](http://thebuildingcoder.typepad.com/blog/2014/10/hackzrh-fluelisee-memento-jobs-and-books.html#2) and
[Brussels](http://thebuildingcoder.typepad.com/blog/2014/10/brussels-hackathon-and-determining-pipe-wall-thickness.html#2).

Finally, to wrap up for today, my son Christopher, sound recording professional, also pointed out a funny video demonstrating a truly
[inimitable presentation style](#4) that
I definitely find worth taking a look at.

#### Add-ins in a Worksharing Environment

Scott Conover presented an absolutely invaluable class at Autodesk University 2013,
**[DV1888](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=1888)**,
on *Making Revit Add-ins That Cooperate with Worksharing*,
followed by a lively and highly competent roundtable discussion
**[DV3464-R](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=3464)**:

> Are you having issues supporting your add-ins in a workshared environment? Have you hit situations where elements can’t be edited or updated due to conflicts? As a follow-up discussion to DV1888, join this roundtable session to further discuss the techniques that are available to developers to operate on workshared models.

I mentioned these sessions
[before](http://thebuildingcoder.typepad.com/blog/2013/11/roomeditorapp-idling-and-benchmarking-timer.html) and
[after](http://thebuildingcoder.typepad.com/blog/2013/12/au-day-2-worksharing-and-revit-2014-api-roundtables.html#5) the
event and promised to publish the full material and roundtable notes sometime.

Apparently, that time has now come, and the long-awaited
[roundtable session notes](2.1) are
included below.

This information is too important to ignore, and too complex to understand without some expert guidance.

I was prompted to return to this material by the following questions:

**Question:** I have a question about worksharing.

I need to store global document settings for several different users.

How can I achieve this effectively in a worksharing environment?

I first tried to use the
[ProjectInfo approach](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html).

That is
[problematic in a workshared environment](http://thebuildingcoder.typepad.com/blog/2013/12/devlab-munich.html),
since this element is
[shared between all users](=).

Using a DataStorage element is not an immediate solution, since it can cause into similar issues, as discussed in the thread on
[tips for extensible storage in a worksharing environment](http://forums.autodesk.com/t5/revit-api/extensible-storage-in-a-worksharing-environment-any-tips/m-p/5112392).

I am a bit unclear about the whole topic, to tell the truth.

- When am I an *owner*, and when a *borrower* of an element?
- Who has write permission?
- How do I request access to an element?
- In another scenario, I can be the owner of an element, but it is out of date.
- All of this also applies to data storage elements.

This all seems rather complex.

Now I would like to explore the API possibilities, e.g. those implemented in WorksharingUtils, etc.

Oops... Unfortunately, the Revit API does not even appear to provide any such class...

The end user tutorial offers enough information on this to get started, so I have lots to read about to understand the product workflow better...

**Answer:** You should take a look at Scott Conover's Autodesk University 2013 class and roundtable on this topic presented in
[AU day two with worksharing and API roundtables](http://thebuildingcoder.typepad.com/blog/2013/12/au-day-2-worksharing-and-revit-2014-api-roundtables.html).

**Response:** "Sorry, this session is no longer available"...

I was only able to download the audio recording and am listening to that now.

**Answer:** For your convenience, I uploaded my local copy of Scott's material:

- [Handout](file:////a/doc/au/2013/doc2/dv1888_worksharing_api_handout.pdf)
- [Slide deck](file:////a/doc/au/2013/doc2/dv1888_worksharing_api_presentation.pdf)
- [Sample code](file:////a/doc/au/2013/doc2/dv1888_worksharingapiapplication.zip)

Scott later pointed out that it is in fact also still available online from the AU web site via:

- [Autodesk University](http://au.autodesk.com)
- [Classes on demand](http://au.autodesk.com/au-online/overview)
- Search for 'dv1888'
- [DV1888](http://au.autodesk.com/au-online/classes-on-demand/class-catalog/2013/revit-for-architects/dv1888):
  *Facing the Elephant in the Room: Making Revit Add-ins That Cooperate with Worksharing*

The roundtable recording
[DV3464-R](http://au.autodesk.com/au-online/classes-on-demand/class-catalog/2013/revit-for-architects/dv3464-r):
*Making Revit Add-ins That Cooperate with Worksharing: A Roundtable Session* is also still posted.

The AU site highlights the upcoming conference and new classes, and yet all the old ones are still available by searching the online course catalog for the presenter name or class number.

**Response:** That is exactly what I need!

Very detailed and down to earth.

**Answer:** As a follow-up, here are the long-awaited notes from the roundtable session:

#### Add-ins in a Worksharing Environment – Roundtable Session Notes

Notes from the Roundtable Session **DV3464-R** on *Making Revit Add-ins That Cooperate with Worksharing* on
December 4, 2013, with Sasha Crotty, Scott Conover, Jeremy Tammik and a dozen expert Revit API and product user participants.

Worksets are used differently in every company, too many variables.

Example usage: Extract sheet parameters for drawing lists, 5-7 buildings in a campus, ca. 60 models, need consolidated drawing list. Open models view journal script, open without worksets. Batch process PDF, open with all worksets. Close Revit using UI automation code and handle dialogues. Open detached is slower than closing in the journal.

Caching element information use unique id. Store document identifier, e.g. path or link instance id. Tracking elements for lighting. If too many elements change, the results are invalidated.

Autoinsertion with families? Autocreate workset for a given object? The API can currently not create new worksets. Some people moved away from this approach. I have seven worksets and want no more.

Have to check in all changes at once, not just selected elements.

Restricting User Access: For us the best answer is not to have worksets. Most uses of multiple worksets are misuses, e.g. for visibility. We set up visibility right in the template and need no worksets.

Elements are still loaded (at least partially) with worksets, just not displayed.

Discipline specific models? We put different disciplines into a single workset and in a single model.

We used to have 30 worksets. We ran into problems. Mix of dumb and smart users. We use worksets to restrict access, e.g. prohibit plumber from moving a duct. We may have 7-8 worksets per trade. We have model managers.

All of our views have view templates assigned that drive the display, separate for display and design.

Two completely different approaches.

Add-in Editing Conflicts and Data Storage: Error messages a re self-explanatory to our users, so we see no need to hide those. Might be more elegant moving to separate DataStorage elements. The DataStorage element ends up in a special workset, named Data Storage Elements. It does matter what workset it is in, because it will not be accessible otherwise depending on what worksets are open. Closed versus locked workset? Use the standard workset or a hidden one. I cannot create new worksets programmatically. It would be nice to specify the workset. I wish that default behaviour for DataStorage elements would be to go someplace that is independent of this whole thing, and as a developer you have an option to specify something different. Issue resolved.

Add-in Misbehaviour Handling and Loading: As an in-house developer, I can just display exceptions to the user and they will bring them to us; it cuts debugging time way down.

Deployment is a challenge, even for AppStore add-ins.

When Revit shuts down, my add-in opens and external application, waits for Revit to shut down, and then copies the add-ins to a local file.

You can load add-ins dynamically, but not unload. You can load an add-in from a byte stream to avoid file issues.

Updater: There are different flavours of things being updated, and via updater is different from other updates. Idling event is still user changes, so only updaters is different. An updater is like a regeneration, in that it triggers regeneration of other elements. System change, e.g. regeneration and updater, versus user change.

File Operations: We have no files at all that are owned by single users, except families, but they are handled differently anyway. Maintaining and controlling standards, ensuring everything is in the right workset, worksets are locked out, eliminate boring stuff and focus on design.

Working with Revit Server? Yes, since 2012. We had disasters and abandoned it. We love it. Server 2012 did not play nicely for us. We are the most up-to-date and running on virtual machines. Our save times went down to almost nothing. Network speed? Ca. 100 Mbit. We have not had any crashes in 2.5 years. Riverbed. 2008 technology to 2013. Yes, expensive. We did run without, and it was a bit slower. We tweaked our settings and worked with Michael Brandy, and it flies. We even put a Revit Server in our single user offices. We would love to learn all the details of your solution.
Automate saving a new central to server?
How about an updater that can run on the central model?

#### Exporting Individual Element Geometry

I put together a new topic collection for The Building Coder on
[exporting individual element geometry](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.47),
prompted by the following conversation:

**Question:** We need to duplicate the exact geometry of a Revit Architecture element, e.g. a wall, including its openings, to a new frozen element.
The purpose is to have the exact geometry of the wall without the windows or doors.

We tried to use the Get\_geometry method but found no way to re-create a new element from it.

We are a fan of your blog, and any help is appreciated.

**Answer:** You cannot recreate all of the exact geometry in a new element.

Revit geometry is basically read-only.

This is obvious and unavoidable.

Revit is parametric.

The geometry is not.

Still, the new DirectShape provides and easy way to duplicate at least some of the geometry, as long as it is not curved or complicated.

On the other hand, I do not think you should be doing this at all.

You will mess up your BIM.

Remember, the BIM is an ***exact*** representation of reality.

In reality, you cannot simply duplicate a wall.

If you want another wall, you have to build a new one.

**Response:** Thank you for the quick answer. I agree with you on the fact that it’ll mess up the BIM, but the use case is to extract an independent sub-set of the geometry and send it to a drafting supplier without the context.

So, the purpose is to extract the exact Wall geometry with opening and save it in a dead geometry as RVT, IFC, DWG, or something.

**Answer:** That is a totally different way of putting things.

I provided several different pretty full-fledged solutions for similar tasks on The Building Coder, and now defined this as a new topic group for you, on
[exporting individual element geometry](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.47):

- Exporting walls and floors to SAT:

- [Export Walls and Floors to SAT](http://thebuildingcoder.typepad.com/blog/2012/01/export-walls-and-floors-to-sat.html)

- Exporting wall parts to DXF for CNC fabrication:

- [Export Wall Parts Individually to DXF](http://thebuildingcoder.typepad.com/blog/2013/03/export-wall-parts-individually-to-dxf.html)
- [ExportCncFab on GitHub and RevitLookup Update](http://thebuildingcoder.typepad.com/blog/2013/10/exportcncfab-on-github-and-revitlookup-update.html)
- [AU 2013 Class FB2938 – Design to Fabrication](http://thebuildingcoder.typepad.com/blog/2013/12/au-day-2-worksharing-and-revit-2014-api-roundtables.html#3)
- [Driving CNC Fabrication and Shared Parameters](http://thebuildingcoder.typepad.com/blog/2013/12/driving-cnc-fabrication-and-shared-parameters.html)

- Saving a solid to a SAT file:

- [How to Save a Solid to a File](http://thebuildingcoder.typepad.com/blog/2013/09/how-to-save-a-solid-to-a-file.html)
- [Saving a Solid to a SAT File Implementation](http://thebuildingcoder.typepad.com/blog/2013/09/saving-a-solid-to-a-sat-file-implementation.html)

**Response:** Thanks a lot Jeremy for the answer. This is exactly what we are looking for.