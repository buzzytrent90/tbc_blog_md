---
post_number: "1201"
title: "Revit Server Thumbnail Requires Redistributable"
slug: "rvt_server_thumbnail"
author: "Jeremy Tammik"
tags: ['revit-api']
source_file: "1201_rvt_server_thumbnail.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1201_rvt_server_thumbnail.html"
---

### Revit Server Thumbnail Requires Redistributable

Here is a quick answer to a simple Revit Server question to close for this week:

**Question:** My application retrieves thumbnails of models stored on Revit Server 2014 via REST calls using the following code:

```
  WebRequest request = WebRequest.Create(
    "http://<ServerName>"
    + "/RevitServerAdminRESTService2014"
    + "/AdminRESTService.svc/<folder>"
    + "|demorevitserver_ab.rvt/thumbnail"
    + "?width=200&height=200" );

  request.Method = "GET";

  request.Headers.Add( "User-Name",
    "d5783faa-3548-4e6e-be9a-aeca05c59352" );

  request.Headers.Add( "User-Machine-Name",
    "MyComputerXyz" );

  request.Headers.Add( "Operation-GUID",
    Guid.NewGuid().ToString() );

  // Next line fails
  // The remote server returned an error:
  // (500) Internal Server Error.

  WebResponse response = request.GetResponse();
```

I checked the log file AdminRESTService.log on the server and do not see what the problem could be there.

How can I solve this, please?

**Answer:** In this case, the log file does not reveal the exact cause of the failure.

However, we analysed a similar case in the past.
The problem there turned out to be that the 'Microsoft Visual C++ 2010 Redistributable (x64)' was not installed on the server.

Please check whether this is also the cause in your case.

The installer has meanwhile been updated to include the required component.

**Response:** I finally found a solution that works for me: installing both the 'Microsoft Visual C++ 2010 Redistributable x64’ and the 'Microsoft Visual C++ 2012 Redistributable x64’ solves the problem.

So not only the 2010 version as you suggested, but both the 2010 and 2012 versions.

#### Stop Suffering

Let's end the week with another quick answer to an apparently not-so-simple question, some sound non-technical advice.

If you are suffering from anything at all that is not of a physical nature,
[Bob Newhart](http://en.wikipedia.org/wiki/Bob_Newhart) has
the following simple and effective remedial advice to offer –
[Stop It](https://www.youtube.com/watch?v=Ow0lr63y4Mw):

Some professional friends of mine tell me that they do indeed keep this advice in mind when offering support for their clients.