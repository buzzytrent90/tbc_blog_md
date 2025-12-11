---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.2
content_type: tutorial
optimization_date: '2025-12-11T11:44:16.543222'
original_url: https://thebuildingcoder.typepad.com/blog/1696_bim360_issues_api.html
post_number: '1696'
reading_time_minutes: 4
series: general
slug: bim360_issues_api
source_file: 1696_bim360_issues_api.md
tags:
- csharp
- filtering
- levels
- revit-api
- sheets
- views
title: Bim360 Issues Api
word_count: 896
---

### BIM 360 Issues API
Today, I am presenting a class on Design Automation for Revit or DA4R and the BIM 360 Issues API at the German Autodesk University in Darmstadt.
It is in German language: \*BIM360 News und Online Revit-Programmierung von BIM via Forge Design Automation API für RVT\*.
The first and main part is similar to yesterday's presentation
on [Forge Design Automation for Revit](http://thebuildingcoder.typepad.com/blog/2018/10/forge-design-automation-for-revit.html) at
the Forge DevCon conference.
Therefore, I will only share the BIM 360 slides (just the text, with most images missing) here now:
- [Overview](#3)
- [Four types of issues](#4)
- [Permissions](#5)
- [Attachments](#6)
- [Assignee](#7)
- [Issues API](#8)
- [Get issues](#8.1)
- [Create issues](#8.2)
- [Issue type and root cause of field issue](#8.3)
- [Directly attach local files (photo) to issues](#8.4)
- [Pushpin Forge viewer extension](#8.5)
- [.NET core sample](#8.6)
- [Node.js sample](#8.7)
- [Node.js sample 2](#8.8)
- [API limitations](#8.9)
For your convenience, here is the slide deck including images in PDF format:
- [Slide deck](zip/bim360_issues_api.pdf)
All the following information comes from a few base sources, so please refer to those to get it straight from the horse's mouth:
- [BIM 360 API documentation](https://forge.autodesk.com/en/docs/bim360/v1/overview)
- [Introducing the BIM 360 Issues API blog post](https://forge.autodesk.com/blog/introducing-bim-360-issues-api)
including [.NET core sample code `bim360-csharp-issues`](https://github.com/Autodesk-Forge/bim360-csharp-issues)
and [live demo](https://bim360issues.herokuapp.com)
- [BIM 360 Issues API sample by Node.js](https://forge.autodesk.com/blog/bim-360-issues-api-sample-nodejs)
including [`bim360-api-demo-node.js` for Issue API sample code](https://github.com/xiaodongliang/BIM360-API-Demo-Node.js/tree/IssueAPI)
and [live demo](https://bim360-issue-csv.herokuapp.com)
#### Overview
- Four Types of Issues
- Permissions
- Attachment
- Issues API
- Current Scope and Limitations
#### Four Types of Issues
- Document Issue in Project Level
- Field Issue in Project Level
- Document Issue in Document Level
- Field Issue in Document Level
#### Permissions
- Permission for Document Issues
- By default, every member of the project can create a document issue in project level
- By default, if a member has permission to a folder, she can create a document issue on the specific document within this folder
- Permission for Field Issues
- Basic: can view issues the member has created or assigned to
- View All: can view all, but cannot create
- Create: can create new and view created and assigned to
- View and Create: can create new issues and view all issues
- No permission with a folder: cannot view the model the field issue was created from
#### Attachments
- Attachment for Document Issue
- Attachment for Field Issue
- Field on PC can attach any kind of files to issue, such as photo
- Field on mobile can only attach photo format to an issue
- Where are the photos?
- A hidden folder named 'Photos’
- Save level as Plans, Project Files, Shop Drawings
#### Assignee
- Assign To
- User: valid member in this project
- Role: Engineer, Designer, ...
- IT: Project Engineer, Project Manager, ...
#### Issues API
- [Get Issues](#8.1)
- [Create Issues](#8.2)
- [Issue Type And Root Cause of Field Issue](#8.3)
- [Directly Attach Local Files (photo) to Issues](#8.4)
- [Pushpin Forge Viewer Extension](#8.5)
- [.NET Core Sample](#8.6)
- [Node.js Sample](#8.7)
- [Node.js Sample 2](#8.8)
- [API Limitations](#8.9)
#### Get Issues

```
GET: {{base_domain}}/issues/v1/containers/{{issue_container_id}}/issues?filter[status]=closed
GET: {{base_domain}}/issues/v1/containers/{{issue_container_id}}/quality-issues
```

- Issue Container
- Document Issue
- Field Issue (in API: Quality Issue)
- API Help: Document Issue, Field Issue
#### Create Issues
- Currently only in project level

```
POST: {{base_domain}}/issues/v1/containers/{{issue_container_id}}/issues
POST: {{base_domain}}/issues/v1/containers/{{issue_container_id}}/quality-issues
```

- API Help: Document Issue, Field Issue
#### Issue Type and Root Cause of Field Issue
- Issue Type
- Quality
- Safety
- PushList
- Commitment
- In API: enum index
- GET supported field issue types to get the corresponding meaningful string
- Root Cause
- GET Root-Cause-Types
#### Directly Attach Local Files (photo) to Issues
![Attach local file](img/bim360_issues_attach_file.png)
#### Pushpin Forge Viewer Extension
- Extension in Forge Viewer
- Same experience as BIM 360 UI
- Can toggle visibility of Issues
- Other data source (custom issue) could also use the skeleton
#### .NET Core Sample
- Get all document issues of one document
- Show up the pushpins by Pushpin Extension
- Live Demo:

[https://bim360issues.herokuapp.com](https://bim360issues.herokuapp.com)
- Blog:

[https://forge.autodesk.com/blog/introducing-bim-360-issues-api](https://forge.autodesk.com/blog/introducing-bim-360-issues-api)
#### Node.js Sample
- Demos on how to manipulate issues
- Attach local photos to an issue
- Load specific issue and its model
- Export all issues to an CSV file (differs from BIM 360 UI function; the CSV contains comments list)
- Live demo:

[https://bim360-issue-csv.herokuapp.com](https://bim360-issue-csv.herokuapp.com)
- Blog:

[https://forge.autodesk.com/blog/bim-360-issues-api-sample-nodejs](https://forge.autodesk.com/blog/bim-360-issues-api-sample-nodejs)
#### Node.js Sample 2
- Dashboards on stats of the issues
![Node.js sample dashboard](img/bim360_issues_nodejs_sample_dashboard.png)
#### API Limitations
- Creating Issue in Document Level is not supported
- Getting project users (Assignee) has not been exposed
- No endpoints to manage permissions
- Attachment workflow is confusing, mixing 2-legged and 3-legged token