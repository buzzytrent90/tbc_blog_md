---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.2
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.392472'
original_url: https://thebuildingcoder.typepad.com/blog/1142_hackergarten.html
post_number: '1142'
reading_time_minutes: 1
series: general
slug: hackergarten
source_file: 1142_hackergarten.htm
tags:
- revit-api
title: Hackergarten Meetup
word_count: 293
---

### Hackergarten Meetup

In order to become more immersed and fluent in web and mobile technologies, I participated in the
[Hackergarten](http://www.hackergarten.net)
[meetup](http://www.meetup.com/Hackergarten-Basel/events/175980272) in Basel last week.

We were a dozen people, mostly experienced in large-scale modern web and database technologies.

The suggested topics were legion, including WebRTC, Spike/Chatapp, Node Webkit, JS tools, CouchDB, Ionic Framework, High scalability, Guice, Gradle plugin, Android slide show, Polymar JS (similar to Angular).

I sat down with three other guys with the goal of exploring
[PouchDB](http://pouchdb.com).

It is an open-source JavaScript database inspired by Apache CouchDB that runs well within the browser and enables applications that work as well offline as they do online, storing data locally while offline, then synchronising it with CouchDB and compatible servers when the application is back online, keeping the user's data in sync no matter where they next login.

We installed the [bower](http://bower.io) package manager to simplify the development work and package management:

We installed the [grunt](http://gruntjs.com) JavaScript task runner in order to run a local web server for testing:

The web server is provided by the
[grunt-contrib-connect](https://github.com/gruntjs/grunt-contrib-connect) plugin.

Those packages were installed and running within minutes, we had a framework set up for developing, and a local web server running for testing.

We started exploring PouchDB step by step in the
[DjDoodle](https://github.com/jeremytammik/DjDoodle)
project that I set up on GitHub for testing purposes.

As a summary, I can say I saw a lot of powerful and impressive technology.

One little immediate consequence I took was to install and start using Chrome instead of Firefox :-)