---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.482136'
original_url: https://thebuildingcoder.typepad.com/blog/1190_rvt_server_central.html
post_number: '1190'
reading_time_minutes: 4
series: general
slug: rvt_server_central
source_file: 1190_rvt_server_central.htm
tags:
- revit-api
- views
title: Accessing a Revit Server Central Model Path
word_count: 728
---

### Accessing a Revit Server Central Model Path

I am still mainly enjoying my break, and back to work for just a couple of days in between.

For today, I present the following summary of a recent discussion on accessing a Revit server central model path including some useful new hints from the development team.

**Question:** Given a path to a central model on a Revit server like `rsn://xxxx`, is it possible to know if the given path points to an existing project or not?

**Answer:** Here our previous overview of the
[Revit Server REST API](http://thebuildingcoder.typepad.com/blog/2013/08/the-revit-server-rest-api.html).

Given the path to the central model on the Revit server, I would assume that you can simply use the Revit Server REST API to query the project model folder contents and file details.

**Question:** I am trying to determine whether the project being saved exists or not during the DocumentSavingAs event.

The first problem is that in this event, the event argument Pathname property does not contain a full server path `rsn://xxx...` when the project is being saved in a Revit server.
It only contains the path relative to the server.

Trying to determine whether that file exists as a real file or in the Revit server I run into a second problem: although I can get the Revit server address from Application.GetRevitServerNetworkHosts, I also need to know the Revit server version, because the REST API calls depend on it.

How can I retrieve get the Revit server version?

Can I safely assume that the Revit version and the Revit server version will always match?

**Answer:** I discussed issue with the development team and received the following answers to your questions:

**[Q]** Given a path to a central model on a Revit server, is it possible to know if the given path points to an existing project or not?

**[A]** No. Given a ModelPath object constructed from `rsn://xxx` using ModelPathUtils, it cannot determine whether or not the project referred in the path already exists or not.

**[Q]** Although I can get the Revit server address from Application.GetRevitServerNetworkHosts, I also need to know the Revit server version.

**[A]** According to the Revit Server installation document, the user should manually configure the file `RSN.ini`:

- %programdata%\Autodesk\Revit Server {version}\Config\RSN.ini

Insert the appropriate Revit Server name/IP of the corresponding version.
When calling the Application.GetRevitServerNetworkHosts method, it will find the corresponding RSN.ini file and output the list of server with the corresponding version, provided that this file is configured correctly.

**[Q]** How can we get the Revit server version?

**[A]** There is currently no feature available to determine the version number of an existing Revit Server.

**[Q]** Can we safely assume that the Revit version and the Revit server version will always match?

**[A]** Yes. Actually, there does exist a version number when the Revit client tries to communicate with the specified Revit Server. If the Revit Server finds itself not matching the version requested by the client, it will refuse the connection.

**Question:** In the DocumentSavingAs event argument Pathname property, I still do not see the full server path when the project is being saved in a Revit server.

Revit seems to be aware of whether the file it is working on is saved locally or to a Revit Server (cf. the ModelPath Class), and as far as I know Revit must talk to the Revit Server in order to handle the worksharing stuff.

How can I determine the absolute path with the server address during this event?

**Answer:** You can get the target Revit Server address as well as the target path in the DocumentSavingAs event handler. Please refer to the property DocumentSavingAsEventArgs.PathName that specifies the target path to which the document is to be saved. It is a string. When SavingAs to a Revit Server model, it contains something like `RSN://hostname/path/to/model.rvt`. You can parse the string yourself to retrieve the Revit Server name. Alternatively, you can use the ModelPathUtils.ConvertUserVisiblePathToModelPath method that converts this string into a ModelPath object for you. The ModelPath.CentralServerPath property of that object will also return the Revit Server name.

We tested this with Revit 2015 and Revit Server 2015 and can confirm that it works well.