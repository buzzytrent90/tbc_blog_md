---
post_number: "1212"
title: "Job Opportunities, Events, Plans and Planes"
slug: "job_event_plan_plane"
author: "Jeremy Tammik"
tags: ['revit-api', 'schedules', 'views', 'windows']
source_file: "1212_job_event_plan_plane.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1212_job_event_plan_plane.html"
---

### Job Opportunities, Events, Plans and Planes

Lots of stuff is going on and we have an exciting heavy duty weekend ahead of us:

- [ADN API evangelist job opportunities](#2)
- [Autodesk Exchange Apps hackathon this weekend](#3)
- [October events and travel in Europe](#4)
- [Comparing a plane and a face](#5)

#### ADN API Evangelist Job Opportunities

I recently mentioned an
[ADN API evangelist job opening](http://thebuildingcoder.typepad.com/blog/2014/05/new-york-travel-preparation-and-adn-job-opening.html#2) in
San Francisco.

We are actually looking for other talented new people to join the ADN DevTech team as well.

The main openings right now are for M&E in North America and for any discipline in China.

However, if you can convince us that you are a really good fit, you can be located virtually anywhere.

If interested, please get in touch, wherever you are!

For instance, with
[Stephen Preston](http://adndevblog.typepad.com/cloud_and_mobile/stephen-preston.html),
Global Manager of ADN Developer Technical Services (DevTech).

#### Autodesk Exchange Apps Hackathon This Weekend

Just as a reminder, this coming weekend, the entire ADN team will be busy assisting at the
[Autodesk Exchange Apps Hackathon](http://thebuildingcoder.typepad.com/blog/2014/08/autodesk-exchange-apps-hackathon.html).

#### Topics

Discussions and presentations during the Hackathon include – and are not limited to:

- How to architect your Autodesk App Exchange application for the following platforms:

- AutoCAD
- Revit
- Inventor
- 3ds Max
- Maya

- Handling IPN notifications- Implementing simple copy protection for your app- Architecting your app to sell on monthly subscription- Selling online web services on Exchange Apps- Making use of Autodesk ‘cloud' services such as our new, web-based, zero-client 3D model viewer- Sustainability apps

Head on over to
[exchangeapphack.com](http://exchangeapphack.com) for
details on the event, a FAQ, the schedule and speaker line up, Rules and Registration link.

#### Preparation

We are looking forward to an exciting event and to helping you get your apps submitted before the deadline on September 30th.

We hope you'll share our excitement and become part of the action in reaching our common customers and sharing and selling your apps.

We're ready to help you!

What you need to do to prepare:

1. Review store and apps information at
   [www.autodesk.com/developapps](http://www.autodesk.com/developapps).
2. Visit
   [exchangeapphack.com/agenda](http://exchangeapphack.com/agenda) for the agenda.
3. [Join the Exchange Apps Hackathon](https://meetings.webex.com/collabs/meetings/join?uuid=MAMJPSK9OBWH4GGID8IKHAVNB7-4P6) on Sept 20th and 21st.
4. If you are participating in cloud or sustainability app competitions, be ready to present your app on at the end of the hackathon on September 21st @ 1.00 PM Pacific time.
5. Visit
   [exchangeapphack.com/resources/videos-2](http://exchangeapphack.com/resources/videos-2) to
   watch exchange store related videos.
6. Hackathon will be conducted using WebEx
   <http://www.webex.com>.

#### System Requirements

Make sure your computer is compatible with WebEx. See the
[WebEx System Requirements](http://support.webex.com/support/system-requirements.html) for
more information about supported platforms.
If you have never used WebEx, you can test your computer's readiness by joining a
[test meeting](http://www.webex.com/lp/jointest) before
the Hackathon event.

Note: WebEx works with most Web browsers. However, certain settings and security restrictions can sometimes prevent WebEx from working properly. If you are having a problem launching WebEx, you may be able to quickly resolve it by using a different Web browser on your computer.

#### Hardware

Please use a computer with speakers and a microphone with mike. Using of headphone with mike will avoid unwanted echoes.

To join the Hackathon, click on the link provided in the email invitation to get to the meeting's information page. Enter your name and email address and click the Join button.

Setup the audio conference – use call using computer option.
Have your webcam switched on as much as possible, e.g., refer to this WebEx training one-minute training video on
[how to share your webcam](http://www.youtube.com/watch?v=ufNYyeTjbj4).

#### Interaction and Q&A

Please actively participate in the Hackathon event by talking or chatting.
Select the Chat tab on your screen to write a message to the presenter, to another participant, or to the full group ('everyone').
Use the chat window to ask questions or unmute and ask the questions through voice.
When you are not talking, please mute your microphone.

And, of course, please feel free to contact us at
[exchangeapphack@autodesk.com](mailto:exchangeapphack@autodesk.com).

#### October Events and Travel in Europe

So what comes after this weekend?

Assuming we survive both the event and the post-event celebrations...

I recently mentioned the dates for the
[DevDays 2014](http://thebuildingcoder.typepad.com/blog/2014/08/upcoming-event-calendar.html#2) toward
the end of the year, and the
[European events](http://thebuildingcoder.typepad.com/blog/2014/08/upcoming-event-calendar.html#4) coming
up in October in Zurich, Brussels, Darmstadt and Berlin.

As it turns out, I am currently planning on attending all four of these, so that will be a busy and exciting travelling month for me.

#### Comparing a Plane and a Face

Finally, let's discuss a more technical Revit API related question as well...

I recently looked at
[planes, projections and picking points](http://thebuildingcoder.typepad.com/blog/2014/09/planes-projections-and-picking-points.html).

Here is a simple geometrical question related to that:

**Question:** In the Revit API, if I have a Plane and a Face, how can I check if they are in the same plane?

Would comparing Z values and their normals suffice?

**Answer:** If your face is a planar face and the planar face is horizontal, the answer is yes.

In general, a plane in 3D space is uniquely defined by exactly four real numbers:

- Three for the normal vector
- One for the signed distance from the origin

If and only if those four are equal, you have the same plane.

For further details, please refer to the recent discussion of the
[mathematical and Revit plane definitions](http://thebuildingcoder.typepad.com/blog/2014/09/planes-projections-and-picking-points.html#10).