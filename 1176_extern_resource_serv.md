---
post_number: "1176"
title: "Referenced Files as a Service"
slug: "extern_resource_serv"
author: "Jeremy Tammik"
tags: ['doors', 'parameters', 'references', 'revit-api', 'rooms', 'views', 'walls', 'windows']
source_file: "1176_extern_resource_serv.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1176_extern_resource_serv.html"
---

### Referenced Files as a Service

Many presentations at the Autodesk Technical Summit were confidential, but not all.

One important class on published functionality of general interest that we can share here is *1-3 Referenced Files as a Service – a new way to provide data to Revit* by Diane Christoforo, Revit Senior Software Engineer.

#### Abstract

How did we transform Revit to reference data from services instead of files? When one Revit project can consist of many different files – CAD links, other linked Revit models, the keynote table, assembly codes, rendering decals – how do you manage all that data in a cloud environment?

In this talk, we'd like to introduce the Referenced Files as a Service project. With this new service, anyone can create a Revit add-on to allow Revit to reference data from any source. It supports arbitrary data sources such as a website, database or Vault. (Normal files still work too!)

As an added bonus, Revit can think of these resources as "resources" and not specifically as "files". In the case of keynotes and assembly codes, no file ever has to exist on disk at any point in the process.

In this talk, we'll cover two interesting examples. First, we'll show you how to get Revit links straight off of the Autodesk wiki. Second, we'll show how to build a keynote "file" from a database. We'll talk about how Revit's architecture changed to support these examples, and how transforming the product to a service architecture simplifies the process of working with new data.

We'll also talk about the challenges we encountered working on the project. We had to do some creative testing to work with functionality that wasn't compatible with our normal test framework. This was also the first time most of the team was working on the referenced files code, and we learned some useful lessons about getting team-members up to speed.

#### Structure

1. Overview – why do we want to get away from files
2. Revit projects before Referenced Files as a Service

1. Many external files
2. Several different implementations
3. How Revit links work
4. How keynotes work

3. The External Resources architecture

1. Built off of the existing External Services framework
2. DB layer

1. Servers handle Revit links, assembly codes, keynotes, or any combination
2. ExternalResourceReference replaces file paths
3. Servers define their resource format
4. How servers tell Revit what resources they provide

3. UI layer

1. Servers can provide their own error handling
2. Revit does not have to know about all possible problems

4. Development challenges

1. Other considered proposals
2. Working on code you don't know
3. Creative testing – what to do when your feature doesn't work with the test framework

5. Demo / in-depth examples

1. Revit links from a wiki

1. Shared coordinates and "open (and unload)"

2. Keynotes from a database

1. No actual files
2. Can build one keynote table from multiple sources

Here is an effective summary of the ideas underlying this framework.

Old concept:

![Old concept](img/extern_resource_serv_1.png)

New concept:

![New concept](img/extern_resource_serv_2.png)

Click here to browse the full
[slide deck](zip/1-3_Diane_Christoforo_Referenced_Files_Public.pdf) presented
at the Autodesk Technical Summit in Toronto.

For more immediate access, I extracted and reformatted the pure text content below.

For optimal understanding, you might want to go through the slide deck and text below in parallel:

#### The Whole Talk, In Three Sentences

- Revit models contain data from many external sources.
- We built a framework that allows third parties to register plugins to provide this data.
- Now we don't have to be responsible for handling every format someone will ever want.

#### Outline

1. [The Problem](#4.1): It's hard to parse data from many sources
2. [The Idea](#4.2): We don't care where the data comes from
3. [Framework Design & Implementation](#4.3)

#### 1. The Problem

#### Anatomy of a Revit project

Here's a very misleading picture of a Revit model showing one single Revit RVT project file:

- Very large hospital.rvt

A real customer project uses data from many sources – other linked Revit models, DWGs, DWF markups, keynotes, rendering decals, shared parameters, etc.

- Very large hospital.rvt

- East Wing.rvt
- Furniture Layout.rvt
- Core and Shell.rvt
- West Wing.rvt
- Site.dwg
- Site2.dwg
- 2014 Imperial keynotes.txt
- 2013 assembly codes.txt
- Second floor hallway art.jpg
- Original.ifc

#### Typical Issues in Q&A Format

Here are some questions from users and corresponding hypothetical answers from previous versions of Revit:

**[Q]** I'm going to download this model from Autodesk 360. You can automatically get all the links and set them up, right?

**[A]** The developers haven't taught me how to use A360.

**[Q]** We want to put our files on this website. You can get them from there, right?

**[A]** What's a website?

**[Q]** We want Revit to build one keynote table from two files so we don't have to put it together by hand. You can do that, right?

**[A]** The developers haven't taught me how to do that.

Revit only knows how to handle files on disk or models on Revit server.

Every time we want to get data from someplace new, we have to:

- Add new UI so the user can browse to the right place
- Add new code to get and parse the data
- Understand what errors can occur and handle them
- Add new UI to explain any errors

#### Where we're going, we don't need files

The cloud doesn't have 'files', so even if we did implement all the individual features people request, we still have a problem.

#### 2. The Idea – Move to a Service Model

1. Think of external data as a 'resource', not a 'file'.
2. Revit no longer handles all data itself.
3. Developers register a plugin to provide resources.
4. Revit doesn't care where the data started out.

Now people can provide the data from whatever source they like!

#### 3. Design and Implementation

Design Goals

1. Impose as few format restrictions on the user as possible.
2. Local files should also use the new framework – 'eat your own dog food'.

Converting our own code to use the new framework was both a good proof-of-concept and a good bug-finder.

#### ExternalResourceService

The new external resource service framework is built on top of Revit's existing ExternalServices framework.

We added two interfaces:

- IExternalResourceServer

- Tells Revit what resources it can provide
- Loads resources when requested

- IExternalResourceUIServer

- Provides feedback to the user after resources have been loaded

All of the plugins are Revit API add-ons. We added a new ExternalService called the ExternalResourceService. Plugin developers can write their own instance of IExternalResourceServer (and optionally IExternalResourceUIServer, if they want to provide any feedback to the user.)

#### Usage in Revit 2015

You can use the External Resources framework to build a plugin supplying:

- Revit links
- Keynotes
- Assembly codes

What are these things?

Well:

- Revit links – you can link one Revit model into another in a read-only fashion.
- Keynotes – keynotes are a special kind of tag that indicates information about the purpose of an object, e.g., 1001010 'Fire Safety'.
- Assembly codes – assembly codes are a classification system for families. They're based on Uniformat.

Revit uses each of these as follows:

- Revit links – links can be used to break up a large model for performance reasons; it is also used to divide a project for collaboration – the architect will link in the structural engineer's model, for example.
- Keynotes – non-keynote tags display an existing property of a door, wall, window, etc. When you place a keynote tag, you choose new information to associate with that object.

Keynotes and assembly codes have virtually the same implementation, so the following examples will focus on Revit links and keynotes.

#### Demo Video

This two-minute [demo video](http://youtu.be/pK0OdAe4fWM) shows loading the keynote table from a wiki page on the Autodesk wiki, modifying the wiki content, reloading the data into the model, and modifying a linked file through the external resource service:

This demo was also shown in the
[Revit 2015 News and DevDays presentations](http://thebuildingcoder.typepad.com/blog/2014/04/revit-2015-api-news-devdays-online-recording.html).

#### Workflow sequence:

1. Revit asks the plugin for the list of resources it provides.
2. User selects a resource
3. Revit asks the plugin to load the resource
4. The plugin gives Revit the resource
5. Revit asks the plugin to handle any errors
6. The plugin handles errors or displays UI

#### Describing a Resource – ExternalResourceReference

An ExternalResourceReference object describes a resource:

Instead of storing file paths, external resources now store a new structure called an ExternalResourceReference encapsulating 'filename', 'path' and 'path on disk or Revit Server'.

Local resources have these as well.

Inside the ExternalResourceReference:

- (Guid) serverId – which plugin provides the resource.
- (String) version – the version of the resource.
- (String) inSessionPath – the 'display name' of the resource – what the user will see.
- (IDictionary<String, String>) referenceInformation – info to identify the resource. The plugin defines the format.

Resource maps can be as simple as:

```
  'resource' = '4'
```

They can also be more complicated:

```
  'name' = 'hospital third floor restroom'
  'user' = 'dchris'
  'language' = 'en-us'
  'format' = 'txt'
  'last-accessed' = '3-26-14'
```

With regards to the reference map, our local resources store the path on disk, and whether the path is relative or absolute.

The plugin that works with the wiki stores the id of the wiki page in its ExternalResourceReference. This means that, even if the title of the page changes, Revit still gets any updates made to that page.

#### Loading a Resource – IExternalResourceServer.LoadResource

```
  void IExternalResourceServer.LoadResource(
    Guid loadRequestId,
    ExternalResourceType resourceType,
    ExternalResourceReference desiredResource,
    ExternalResourceLoadContext loadContext,
    ExternalResourceLoadContent loadContent)
```

- loadRequestId – Revit sends a Guid with every load request so they can be uniquely identified.
- resourceType – useful for plugins that handle multiple types of resource.
- desiredResource – the resource which Revit would like the plugin to load.
- loadContext – information about the state of Revit at the time of the request.
- loadResults – the results of loading the resource. Each resource type has its own subclass.

ExternalResourceLoadContext provides information such as: what was the previous resource (so the plugin can tell if it's loading the same resource), whether the load was an automatic or explicit action (The plugin may want to do different actions on file open versus when the user presses the 'reload' button), and the name of the Revit model requesting the resource.

#### ExternalResourceLoadContent

Each resource type has its own subclass.

Keynotes and assembly codes can be built directly.

The plugin builds a table of key/value pairs and returns it to Revit.

No files needed anywhere!

Revit links have to return a locally cached path.

Revit's still the only thing that can build a Revit model.

The user doesn't need to know or care that there's a local copy of the model.

The plugin also sets a success/failure value, so Revit can quickly see whether the operation succeeded or not. The default value is 'uninitialized', so we also have some idea of whether something went terribly wrong on the server's end.

Other IExternalResourceServer functions:

- IsResourceWellFormed – we don't know what format the server is using, so it has to answer this question for us.
- AreSameResources – again, the server needs to tell us if two resources are the same.
- GetResourceVersionStatus – Revit skips loading if we already have the most recent version of a resource.
- SetUpBrowserData – this is how the server tells Revit what resources it can provide. Revit then lists the resources in the UI.
- SupportsExternalResourceType – servers can provide any or all of keynotes, assembly codes, and Revit links.

For AreSameResources – originally we just compared the referenceInformation, but we realized that two resources might have different strings but still be considered 'the same' by the server. As an example, internally you can use relative or absolute paths. But the relative and absolute versions of the same file are still ... the same file, even though their reference information would be different.

#### Reponses and Error handling

We don't know what will go wrong.

How can Revit respond to errors like these?

- Revit model 'MyServer://myhouse.rvt' is missing from the server.
- The network connection is very slow, so the operation was cancelled. Try again?
- You aren't logged into the website. Log in.
- Your subscription to 'Keynotes for Everybody' has expired. Would you like to renew?
- We couldn't reverse the polarity of the neutron flow.
- Your computer is full of bees. Please reboot.

Revit can't anticipate everything that can go wrong on the server end. We let the plugin do error handling because the plugin knows how to fix its own problems.

These situations are managed using the IExternalResourceUIServer.HandleLoadResourceResults method:

```
  void IExternalResourceUIServer.HandleLoadResourceResults(
    Document document,
    IList<ExternalResourceLoadData> loadData)
```

- document – The Revit model which the resource is loaded into.
- loadData – A list of load results. The plugin can see any Revit-internal errors that occurred, and decide what UI, if any, to display.

These are the details of the function the plugin uses to provide results back to Revit.

There are two notable things here:

1. The plugin has full access to the Revit model at this point, so it can do its own link reloading if, for example, the user just needed to log in to a service.
2. We give the plugin all resources it loaded. This enables the add-in to avoid displaying the same 'Your resource is missing' dialogue five times over if it tried to load five links and they all failed.

#### Where to Learn More

You can download the Revit 2015 SDK including the ExternalResourceServer add-in SDK sample add-in from the
[Revit Developer Centre](http://www.autodesk.com/developrevit).

#### Add-in Demonstrating Real-world Use

The Kiwi Solutions
[Easy Keynoter](http://www.kcsl.biz/#!easy-keynoter/com5) add-in
shows one impressive practical use of the Revit 2015 'referenced files as a service' feature and solves several long standing issues with regards to keynotes in Revit.