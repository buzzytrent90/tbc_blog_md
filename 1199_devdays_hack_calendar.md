---
post_number: "1199"
title: "Upcoming DevDay, Meetup and Hackathon Event Calendar"
slug: "devdays_hack_calendar"
author: "Jeremy Tammik"
tags: ['revit-api', 'rooms', 'views']
source_file: "1199_devdays_hack_calendar.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1199_devdays_hack_calendar.html"
---

### Upcoming DevDay, Meetup and Hackathon Event Calendar

Here is a list of upcoming events, most of which I will be involved in in one way or another:

- [DevDays 2014](#2)
- [DevDay & DevHack in Las Vegas](#3)
- [Meetups and Hackathons](#4)

#### DevDays 2014

Here is the list of cities and dates where our DevDays events 2014 will be held around the world later this year.

These events are accessible to ADN members.
If you are not an ADN member and still would be interested in learning more about application development related to Autodesk products and technology, please
[contact us](http://www.autodesk.com/joinadn) and
let's discuss.

The DevDays conferences are the best way to learn about Autodesk's newest technologies and future products.
To get a feel for what goes on at these events, ADN members can check out the presentations from last year on the
[ADN extranet](http://adn.autodesk.com/adn) at
[Events](http://adn.autodesk.com/adn/servlet/index?siteID=4814862&id=5105692).

A complete DevDays 2014 agenda will be posted soon, and online registration will go live in late September. Note that the registration for the ADN Conference at AU in Las Vegas will go live on August 27, 2014.
See [below](#3) for more information on that.

New for 2014: We will be hosting Meetup events in most locations.
These events are open to everyone, not just ADN partners.
So, invite your technology friends along for an evening of learning and sharing in a casual environment.

#### DevDays 2014 Locations and Dates

- North America

- 2014-12-01 – ADN Conference at AU, Las Vegas, USA
- 2014-12-02 – DevHack at AU, Las Vegas, USA

- North Asia

- 2014-11-06 – Beijing, China, with evening Meetup
- 2014-11-07 – Beijing, China, DevHack at AU China
- 2014-11-10 – Shanghai, China, with evening Meetup
- 2014-11-11 – Shanghai, China DevHack
- 2014-11-17 – Seoul, South Korea
- 2014-11-19 – Tokyo, Japan
- 2014-11-20 – Tokyo, Japan, DevHack
- 2014-11-28 – Osaka, Japan

- South Asia

- 2014-11-11 – Sydney, Australia, DevHack at AUx Sydney
- 2014-11-12 – Sydney, Australia
- 2014-11-14 – Singapore, Singapore
- 2014-11-21 – Bangalore, India, with evening Meetup

- Western Europe

- 2014-12-08 – Paris, France, with evening Meetup
- 2014-12-09 – London, United Kingdom, evening Meetup
- 2014-12-10 – Farnborough, United Kingdom
- 2014-12-11 – Gothenburg, Sweden, with evening Meetup
- 2014-12-15 – Munich, Germany, with evening Meetup
- 2014-12-16 – Milan, Italy, evening Meetup
- 2014-12-17 – Milan, Italy

- Eastern Europe

- January 2015 – Moscow, Russia (exact date to be confirmed)

- Latin America

- 2015-01-14 – São Paulo, Brazil

#### DevDay & DevHack in Las Vegas

The ADN Conference at Autodesk University in Las Vegas, Nevada, USA, is taking place on December 1, with two separate DevHack events on December 2, for Design & Engineering and Media & Entertainment, respectively.

Registration will open today, on August 27, 2014.
Visit the registration website at
<http://au.autodesk.com/plan/pre-conference/adn-conference-devdays>.

Note:

- Sign in with your Autodesk ID or create one when prompted (this is **not** your ADN number).
- Make sure you check the ADN box and enter your ADN number. If you don't, you will not see the registration option for the ADN Conference and DevHack.
- Select your ADN Conference classes from the class catalogue.

#### Meetups and Hackathons

Over the coming months we DevTech engineers and ADN evangelists will be attending a number of different meetup and hackathon events. Here's a list of some of them:

- August 22 – September 5, 2014

  China Roadshow + DevLab + Meetup + Classroom Training in Beijing, Wuhan, Chengdu, and Xian, China

  In attendance: Xianhua Tang, Xiaodong Liang, Daniel Du, Joe Ye
- September 6-7, 2014

  [TechCrunch](http://techcrunch.com/events/disrupt-sf-hackathon-2014/event-home) Hackathon – San Francisco, CA

  In attendance: Jim Quanci, Stephen Preston, Cyrille Fauvel, Philippe Leefsma
- September 8-10, 2014

  [I♥3 APIs](http://iloveapis2014.com) Conference, San Francisco, CA

  In attendance: Jim Quanci, Stephen Preston, Cyrille Fauvel, Philippe Leefsma
- September 9, 2014

  HTML5 Meetup Group, San Francisco, CA (Autodesk sponsoring the event)

  In attendance: Jim Quanci, Stephen Preston, Cyrille Fauvel, Philippe Leefsma
- September 10, 2014

  Tokyo 2nd Meet Up, Tokyo, Japan

  In attendance: Toshiaki Isezaki
- September 12-14, 2014

  [AEC Hackathon](http://www.aechackathon.com/seattle.html), Seattle, WA

  In attendance: Augusto Goncalves
- October 10-12, 2014

  [hackzurich](http://hackzurich.com) Hackathon, Zurich, Switzerland

  In attendance: Philippe Leefsma, Jeremy Tammik
- October 17-18, 2014

  [Open Data](http://www.transformabxl.be/agenda/event/hackathon-open-data-brussels) Hackathon, Brussels, Belgium

  In attendance: Cyrille Fauvel, Philippe Leefsma, Jeremy Tammik
- October 23-24, 2014

  [Autodesk University Germany](https://cluster.ems-secure.de/registrations/autodesk/au.2014/ems.registration.php), Darmstadt, Germany

  In attendance: Jeremy Tammik
- October 25-26, 2014

  [#TMUhack](http://www.meetup.com/TechMeetups-Berlin/events/161213342) TechMeetups Hackathon, Berlin, Germany

  In attendance: Adam Nagy, Cyrille Fauvel

#### Toggling Annotation Categories per View

One little pure Revit API issue before closing...

**Question:** Is it possible to programmatically control the toggle determining whether to 'Show annotation categories in this view' in the Visibility/Graphics Overrides dialogue, marked in red below?

![Visibility/Graphics Overrides dialogue](img/visibility_graphics_overrides_show_annotation_categories.png)

**Answer:** Please try the read-write View.AreAnnotationCategoriesHidden property.