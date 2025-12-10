---
post_number: "1009"
title: "Revit Server API Access and VBScript"
slug: "revit_server_soap_vbs"
author: "Jeremy Tammik"
tags: ['revit-api', 'views']
source_file: "1009_revit_server_soap_vbs.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1009_revit_server_soap_vbs.html"
---

### Revit Server API Access and VBScript

I recently summarised the material describing the
[Revit Server API](http://thebuildingcoder.typepad.com/blog/2013/08/the-revit-server-rest-api.html).

Apparently, that still does not cover everything.

You want more? Well, here is more!

It led to the following two new queries and further clarification of this topic:

- [SOAP API and no access to Admin service](#2)
- [Accessing Revit Server using VBScript](#3)

Here are the gory details in all their glory:

#### SOAP API and No Access to Admin Service

**Question:** Can you please let me know the cause for the 'Endpoint not found' error message?

I'm trying to browse the AdminRestService.Svc through IIS(7.0):

![AdminRestService](img/rs_AdminRestServices.jpg)

It always reports 'Endpoint not found':

![Endpoint Not Found](img/rs_ep_not_found.png)

Am I missing some configuration setting?

I tried both ways, using the Web request method in .NET and also through IIS.

I searched the blog and Internet but couldn't find any resolution or articles on any Revit 2014 AdminRestService 'Endpoint not found' issues.

Am I right in assuming that the Application.CopyModel method is one of the methods on the above AdminRestService.svc service?

By browsing to
<http://localhost/RevitServerAdmin2014/default.aspx>,
I am able to connect successfully through the user interface and can even see all the models available on the server:

![AdminRestService](img/rs_revit_server_admin.png)

As said, I'm just unable to use the
<http://Localhost/RevitServerAdminRESTService/AdminRESTService.svc> URL.
It always replies with 'Endpoint not found.'

Currently I'm trying to use the RevitserverTool.exe utility and its createLocalRvt command to copy the Revit model to a different location within the server.
I hope this works.

My goal is to programmatically copy the model, e.g. Project-X.Rvt in the screen shot above, to a different location, open the file in a Revit MEP client and start the
[extraction process using the add-in](http://thebuildingcoder.typepad.com/blog/2013/08/the-revit-server-rest-api.html) I
developed.

**Answer:** Did you connect successfully through the user interface?

Did you look at
[Connecting to the Revit Server Administrator](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00005-More_Inf0/0315-Administ315/0319-Revit_Se319/0325-Revit_Se325/0326-Connecti326)?

Have you implemented it along the lines of the
[revitserverviewer.zip sample](http://thebuildingcoder.typepad.com/blog/2011/11/revit-server-rest-api.html)?

Does that sample work correctly for you, adapted to your server context?

Good.
That is all you have to worry about.

The behaviour you are seeing where the admin service endpoint cannot be browsed is as expected.

However, other endpoints, such as those available for the model service, should be just fine.

Why did you try to access the admin service?

The easiest and most efficient path for you will probably be to base your use of this API on the existing sample applications.

If they have no need of the admin service, the you presumably will not either.

Are you seeing any functional failures that lead you to believe there is an issue with your server?

Or were you just surprised to note the behaviour described below?

If you are running into some kind of blocking problem, it is not related to the inability to browse the admin service.
This has never been possible.

The Revit Server admin UI uses AdminService and not AdminRESTService.
It is possible that the REST service is not working on your machine but AdminService is.

Here are some important points to clarify about Revit Server:

1. REST service:

- It is RevitServerAdminRESTService2014.
- As said, it is **not** used by the admin UI.
- It is normal that you see 'Endpoint Not Found' when you access its base address
  <http://localhost/RevitServerAdminRESTService2014/AdminRESTService.svc>,
  because no resource corresponds to this base address.
- You can refer to the REST API help documentation for the detailed use.

2. The RevitserverTool.exe tool also does **not** connect to the REST service; instead it communicates with the Model service, which is a SOAP service.
3. Similarly, the Revit API call Application.CopyModel (as part of Revit) talks with the model service.

That should take care of that issue.

Next:

#### VBScript Access to the Revit Server REST API

Here is an example by Jeff Campbell of
[Burns & McDonnell](http://www.burnsmcd.com) showing
how to access Revit Server and download project files using both VBScript and AutoIT:

**Question:** Is it possible to access the REST API with VBScript?
I tried running a small script using the Microsoft.XMLHTTP method but am unable to get a valid return.

**Answer:** The Revit Server API is simple, and so is REST.

Basically, everything there is to say about it is listed (or pointed to) in the previous discussion of the
[Revit Server REST API](http://thebuildingcoder.typepad.com/blog/2013/08/the-revit-server-rest-api.html).

Regarding the use of REST from VBScript, I would suggest a simple Google search, e.g. for
['rest VBScript'](http://lmgtfy.com/?q=rest+VBScript).

I just tried it out, and the results look promising, so I think you are in luck.

**Response:** Thank you!
I got it worked out last night.

I was having an issue passing the headers after my initial open statement but I now have it working.

I started with your blog post, and that gave me the idea to try this with REST.

Here is the base that I am working from in both AutoIT and VBScript.

First, the VBS version, RESTCall.vbs:

```
'======================================================
' NAME: RESTCall.vbs
'
' AUTHOR: Jeff Campbell , Burns and McDonnell
' DATE  : 8/22/2013
'
' COMMENT:
'  Proof of concept testing for using REST API
'  calls to Revit server for addition
'  into the BMCD local copy tool.
'======================================================

Option Explicit

'Declare variable section

Dim restReq
Dim url
Dim userName
Dim userMachine
Dim GUID
Dim outData
Dim StatusCode
Dim CleanGUID
Dim WshNetwork

'Enviromental sets

Set restReq = CreateObject ("Microsoft.XMLHTTP")
Set WshNetwork = WScript.CreateObject("WScript.Network")

'Setup header section

userName = WshNetwork.UserName
MsgBox userName,0,"This is the logged on user"
userMachine = WshNetwork.ComputerName
MsgBox userMachine,0,"This is the computer name"
GUID = GetGuid

'MsgBox GUID,0,"This is my GUID"

GUID = Replace(GUID,"{","")
GUID = Replace (GUID,"}","")
MsgBox GUID,40,"This is me clean GUID"
Set WshNetwork = Nothing

'Create the headers that we will need to make the request

url = "http://.../RevitServerAdminRESTService2013/AdminRESTService.svc/ /Contents"

restReq.open "GET", url, False

restReq.setRequestHeader "User-Name", userName
restReq.setRequestHeader "User-Machine-Name", userMachine
restReq.setRequestHeader "Operation-GUID", GUID

restReq.send("")

outData = restReq.ResponseText
StatusCode = restReq.Status

MsgBox StatusCode,0,"Status"
MsgBox outData,0,"My Data"
Set restReq = Nothing

'------------User Function Section---------------------
'GUID generator

Function GetGuid()
Set restGUID = CreateObject ("Scriptlet.TypeLib")
	GetGuid = Left(CStr(restGUID.Guid), 38)
	Set restGUID = Nothing
End Function
```

I was originally headed down the VBScript path but once I figured out how to use the REST calls there, I applied the same principles in AutoIT and it worked fine for me.

Here is that version, RESTcall.au3:

```
;Includes

#Include <WinAPI.au3>
#include <Array.au3>
#include "JSON.au3"
#include "JSON_Translate.au3"

$restReq = ObjCreate("winhttp.winhttprequest.5.1")
$restGuid = _CreateGuid()

;Dialogs for testing remark out for production use

MsgBox (4096,"My Computer Name",@ComputerName)
MsgBox (4096,"My Name",@UserName)
MsgBox(4096, "Generate Guid", $restGuid)

;REST Request Section

$restReq.Open("GET", "http://.../RevitServerAdminRESTService2013/AdminRESTService.svc/ /Contents", False)

$restReq.setRequestHeader ("User-Name", @UserName)
$restReq.setRequestHeader ("User-Machine-Name", @ComputerName)
$restReq.setRequestHeader ("Operation-GUID", $restGuid)

$restReq.Send()

$oReceived = $restReq.ResponseText
$oStatusCode = $restReq.Status

;Dialog box of status code for troubleshooting

MsgBox(4096, "Status Code", $oStatusCode())

If $oStatusCode == 200 then
  ; The value of 2 overwrites the file if it already exists
  $file = FileOpen("Received.txt", 2)
  FileWrite($file, $oReceived)
  FileClose($file)
  MsgBox(4096, "My test for Nolan", $oReceived ())
EndIf

;-------Section for called functions-------------------

Func _CreateGuid()
  Local $Guid = DllStructCreate($tagGUID)

  $Result = DllCall("OLE32.DLL", "dword", "CoCreateGuid", "ptr", DllStructGetPtr($Guid))
  $Result = _WinAPI_StringFromGUID(DllStructGetPtr($Guid))

  $strresult = StringTrimLeft($Result, 1)
  $strresult = StringTrimRight($strresult, 1)

  Return $strresult
EndFunc
```

This is getting added to a 'copy down' tool that we created in AutoIT for our local copy creation.
The original tool was based on INI files that we keyed around each office file server.
When I moved to Revit Server I simply modified it and continued using the INI files.
This has proven rather cumbersome since we need to track each filename for our projects.
Using some of the information in your blog and an AU presentation from Rob Howarth I decided to make some improvements for our 2014 roll out.
The new copy down tool will use the REST API to generate all of the information needed off of Revit Server.
This eliminates the need for the INI files and streamlines the amount of time it was taking to manage the system.

I am also using it to create various CMD files that we then use to automate some of our other processes.

Many thanks to Jeff for sharing this!