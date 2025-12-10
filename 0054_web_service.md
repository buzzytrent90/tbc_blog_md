---
post_number: "0054"
title: "Web Service"
slug: "web_service"
author: "Jeremy Tammik"
tags: ['elements', 'references', 'revit-api', 'windows']
source_file: "0054_web_service.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0054_web_service.html"
---

### Web Service

More than half way through our west European DevDays tour, we have had exciting days with lots of meetings, presentations, discussions, questions and answers in London, Gteborg and Frankfurt.
Today we are in Milan, with the next stop in Paris planned for tomorrow.
Meanwhile, here is a question related to the discussion between Dave and
[Ed Pitt](http://www.cadsmart.net)
in the post on
[using namespaces](http://thebuildingcoder.typepad.com/blog/2008/12/using-namespaces.html)
a couple of days ago:

Question: I'm developing an application for Revit Architecture 2009 using Visual Studio 2005 with RevitAPI.dll that interacts with a web service on the client side. In it, a client DLL developed in Visual Studio 2005 interacts with a web service to download and upload Revit projects from a server. The communication between client and server is secured and uses the MTOM or Message Transmission Optimization Mechanism technology.
The client DLL works well when it is included as a reference in a stand-alone .NET project such as a WindowsApplicationForm or console application, the authentification succeeds and files can be downloaded from and uploaded to the server. However, when the client is included in the Revit project, the call to the server to download or upload projects fails. It returns an exception from the server and cannot access the services, although it is using the same user id and password as previously. In debug mode on the server side, I can see that the request coming from the Revit application is quite different from the .NET application one.
What could be the explanation for this? Are there any built-in security issues in Revit Architecture that might cause this behaviour?

Reply from Miroslav Schonauer and Saikat Bhattacharya:
You might try creating a Revit.exe.config file and fine tuning its contents.
The problem is possibly in some setting required for the particular web service which in the standalone application is either assumed by default or configured via the App.exe.config file. The solution might be to create a Revit.exe.config file in the Revit.exe install folder. Unlike acad.exe.config, this one does not come with the install since no .NET fine-tuning is required by Revit. Of course it is virtually impossible to say exactly *what* needs to be fine tuned.
One project we saw had an issue related to authentication that was solved by adding a complex element to the Revit.exe.config file. Here is an example of a skeleton configuration file:

```
<configuration>
  <system.serviceModel>
    <bindings>
      ...project and web service specific info
    </bindings>
    <client>
      ... project and web service specific info
    </client>
  </system.serviceModel>
</configuration>
```

Even though this may not completely resolve the current issue,
it is definitely useful information for some applications and a good point to start exploring from.