---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: news
optimization_date: '2025-12-11T11:44:17.034015'
original_url: https://thebuildingcoder.typepad.com/blog/1915_roadmap_survey.html
post_number: '1915'
reading_time_minutes: 3
series: general
slug: roadmap_survey
source_file: 1915_roadmap_survey.md
tags:
- references
- revit-api
- sheets
- views
title: Roadmap Survey
word_count: 574
---

### Revit Roadmap, API and DA4R Survey
A quick post to highlight the opportunities to provide feedback on the Revit product, the Revit API and the Forge Design Automation API for Revit:
- [Revit public roadmap update](#2)
- [Revit and DA4R API survey 2021](#3)
- [MacOS Big Sur upgrade](#4)
- [The Economist on ransomware and cybersecurity](#5)
#### Revit Public Roadmap Update
The [Revit Public Roadmap Update Summer 2021](https://blogs.autodesk.com/revit/2021/07/06/revit-public-roadmap-update-summer-2021)
was published last month with an invitation to provide feedback via likes:
> Autodesk Revit 2022 shipped in April and we have added new features to the Revit Public Roadmap.
For a live look, check out
the [Kanban board on Trello and like your favourites...](https://trello.com/b/ldRXK9Gw/revit-public-roadmap)
![Revit public roadmap](img/2021-07-06_revit_public_roadmap.jpg "Revit public roadmap")
#### Revit and DA4R API Survey 2021
Specifically targeted at us Revit add-in and Forge app programmers,
the [Revit and Design Automation for Revit API Survey 2021](https://forge.autodesk.com/blog/revit-and-design-automation-revit-api-survey-2021) is
now also ready and eagerly awaiting your input.
> The Revit product team is conducting a survey to improve the functionality of Revit services:
> [Go to the Revit API survey](https://autodeskfeedback.az1.qualtrics.com/jfe/form/SV_ex5UwT1A2lj0s6y)
> Please take this brief 5-minute survey to help the Revit team prioritize new features and upcoming enhancements to the future releases of the Autodesk Revit and the Forge Design Automation for Revit APIs.
> The survey remains open till September 24th.
So, you have exactly one month's time from now.
Thank you in advance for your time and interest!
#### MacOS Big Sur Upgrade
Not related to Revit or its API, company policy forced me to upgrade to Big Sur MacOS 11.5.2.
The upgrade went smoothly.
However, a few things caused problems for me personally after the successful upgrade.
Luckily, other people have faced and solved the same issues in the months since the initial Big Sur release –
so, it was a good thing I was able to wait a while before diving into it:
- [Unable to mount my `/a`, `/j`, `/m` and `/p` folders in the root dir](https://www.quora.com/Can-you-mount-the-root-system-file-system-as-writable-in-Big-Sur-MacOS-Big-Sur-Apple)
- [Komodo editor stopped running](https://community.komodoide.com/t/komodo-and-big-sur-do-not-upgrade/5191/15)
- [The VeraCrypt encryption tool failed](https://techstuffer.com/veracrypt-macos-bigsur-compatibility/)
Mainly for my personal future reference, here are the steps I ended up taking to mount my shortcut folders in the root directory:

```
mount --> /dev/disk1s5s1 on / (apfs, sealed, local, read-only, journaled)
mkdir -p -m777 ~/mount
sudo mount -o nobrowse -t apfs /dev/disk1s5 ~/mount
cd ~/mount/
sudo ln -s /Users/jta/a
sudo ln -s /Users/jta/j
sudo ln -s /Users/jta/music m
sudo ln -s /Users/jta/Pictures p
sudo ln -s /Volumes v
sudo bless --folder ~/mount/System/Library/CoreServices --bootefi --create-snapshot
```

#### The Economist on Ransomware and Cybersecurity
Still in the realm of computers and technology, but not Revit API related either,
\*The Economist\* published a nice overview
explaining how [ransomware highlights the challenges and subtleties of cybersecurity](https://www.economist.com/briefing/2021/06/19/ransomware-highlights-the-challenges-and-subtleties-of-cybersecurity) and
how governments want to defend themselves – and attack others.