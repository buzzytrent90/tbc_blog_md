---
post_number: "0869"
title: "The BIM 360 Glue Viewer and REST API"
slug: "bim360_glue_api"
author: "Jeremy Tammik"
tags: ['levels', 'parameters', 'references', 'revit-api', 'views', 'windows']
source_file: "0869_bim360_glue_api.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0869_bim360_glue_api.html"
---

### The BIM 360 Glue Viewer and REST API

As you certainly know by now, Autodesk provides a range of powerful cloud-based solutions, specifically
[Autodesk 360](https://360.autodesk.com),
[PLM 360](http://www.autodeskplm360.com) and
[BIM 360 Glue](https://bim360.autodesk.com)
([reference](http://wikihelp.autodesk.com/BIM_360_Glue/enu/Help/Help)).

Of special interest to AEC developers, the BIM 360 Glue platform includes support for programming interfaces and a software development kit, the
[BIM 360 Glue SDK](http://bim360.autodesk.com/api/doc/index.shtml),
providing a set of tools for developers to interface with it.

The programming support consists of two distinct components:

1. [Display component](#1): embeddable 3D viewer to provide visual access to models within custom third party web applications.
2. [Web services API](#2): REST based service data access to users, projects, models, etc.

#### Display Component

The Glue Viewer is an embeddable component used to show 3D models from the Glue platform

Viewer parameters allow control of the GUI, i.e. application controls such as navigation, display lists, etc., so the viewer experience can be customised by developers.
![Glue viewer](img/glue_viewer.png)

The viewer currently supports Windows 32 and 64 bit platforms, just like the Glue web application.

#### Web Services API

The Glue web services API provides a RESTful data access interface to query and modify data and return JSON or XML.

It enables developers to integrate external applications, e.g., project management systems, accounting systems and custom developed solutions with the BIM 360 Glue platform.

Glue stores information about user interactions within the system.
These operations are referred to as 'Actions'.
Typical Actions can be:

- Uploading a model- Creating a view- Adding mark-ups- Creating a clash report

![Glue actions](img/glue_actions.png)

The Glue Web Service API returns many of these actions in API responses.

Actions can be loaded to the viewer to show the user the exact view and state of a model when the action was performed.

#### Structure of a Client Request

For a typical REST API request, the client sends a HTTP/HTTPS request, POST or GET, depending on the particular service call.
Required parameters are sent to the server for authentication.
An example URL for a login request could be something like:

```
https://bim360.autodesk.com/api/security/v1/login.json
```

The server responds with an HTTP Status of 200 for success and appropriate other codes for failures.

The Glue web service API provides the following service groups:

- Security Service: Responsible for security management of Glue user accounts.- Project Service: Manage projects within the Glue platform.- Model Service: Manage 3D models within the Glue platform.- User Service: Manage user information.- Action Service: Access and manage action records.- System Service: Retrieve information and check availability.

Here is an example of a web services API call to the Model Service to retrieve information of a given model specified by the model\_id:

```
https://bim360.autodesk.com/api/model/v1/info.json?
  &company_id=autodesk_jt
  &api_key=c7e595c69a35422c8414e51736f67761
  &auth_token=f79462fc5a5346ba9dbd4f395781f9b4
  ×tamp=1349801232
  &sig=de4c1771541d4789a1e63762c60b34f1
  &model_id=53c41451-49e7-4b0a-919c-1499604d10c6
  &sterm=
  &pretty=1
```

#### So what can we do with this?

With the SDK, partners and customers can build a variety of fully interactive web based custom applications and integrations taking advantage of the Glue platform.

A simple call flow for a custom application might be:

- Perform user authentication.- Query and list available models.- Use the Glue viewer to display the model within the custom application.

This enables tasks like system integration, project management, accounting, etc.
The web services API provides full data access to system objects:

- Users- Projects- Models- Actions

An application can automatically update projects, models, etc. with information from external systems.

Basically, we are looking at the following architectural hierarchy, from bottom to top:

- BIM 360 Glue Platform @ Amazon- Internet Connectivity- BIM 360 Glue API Server- BIM 360 Glue API- System Integrations Enabled via Glue API

#### Where do I start?

Download the
[BIM 360 Glue SDK](http://bim360.autodesk.com/api/doc/index.shtml),

You will need an API Key and an API Secret.
These are unique identifiers assigned for each user, company, or partner.

Furthermore, a Glue developer account is required, a user name and password.
A normal user account can be converted to a developer one on request.There are even further levels: a default developer account works within the privileges of normal glue user, whereas a privileged developer account can perform requests on behalf of other users as well.

The authentication to the BIM 360 Glue Platform starts with an API key and secret assigned by Autodesk.
Once you have these two pieces of information, API calls can be authenticated to the BIM 360 Glue Platform in one of two ways:

- User name and password; using the API key and secret, a signed request including the user name and password is sent to the BIM 360 Glue Service, which returns an authorisation token.- The authorisation token is used for all subsequent calls.

Here is a quick reference to further sources of information and two examples:

- [SDK overview](http://bim360.autodesk.com/api/doc/index.shtml)
- Detailed [web service API documentation](https://bim360.autodesk.com/api/doc/doc_api.shtml)
- [Viewer support – file formats](https://bim360.autodesk.com/file_compatibility.html)
- [Integration with project management system – CMIC](http://youtu.be/LSeJd6ioHC0)
- [Glue embedded within a web application – Maximo](http://www.youtube.com/watch?v=EZaCSKdIHto)

I am looking forward to hearing what you come up with using this, and to exploring it in more depth myself anon.
Have fun!