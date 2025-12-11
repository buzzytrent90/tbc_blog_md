---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.024789'
original_url: https://thebuildingcoder.typepad.com/blog/1437_bcf_manual_addin.html
post_number: '1437'
reading_time_minutes: 5
series: general
slug: bcf_manual_addin
source_file: 1437_bcf_manual_addin.md
tags:
- csharp
- references
- revit-api
- sheets
title: Bcf Manual Addin
word_count: 1098
---

### BCF API and Manually Setting Up an Add-In
The time is overdue to migrate
the [Visual Studio Revit Add-In Wizards](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.20) to
Revit 2017.
This time, instead of a simple flat migration like in previous years, I decided to [set up a new Visual Studio C# .NET Revit add-in project manually](#3), by hand, completely from scratch, just to see whether anything significant changed since I did that last, and to ensure that the wizard is really using all the required Visual Studio settings.
I tried it out in by implementing a [buildingSMART](http://www.buildingsmart-tech.org) [BCF API sample client](#2).
Basically, that requires the following steps:
- Create a new pure Visual Studio class library
- Rename `Class1` to `Command`
- Add references to the Revit API assemblies
- Implement an external command
- Implement an add-in manifest
- Define an add-in GUID
- Implement a post-build event to auto-install the add-in
#### The BCF API and a Sample Client
To try out these steps, I made use of a brand new sample project that I implemented in response to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread inquiring
about [support for the BCF API](http://forums.autodesk.com/t5/revit-api/support-for-bcf-api/m-p/6282780):
\*\*Question:\*\* Can anyone answer if Revit supports the BCF API to allow for third party integrations?
\*\*Answer:\*\* I searched the Internet for 'bcf api' and found
the [BuildingSMART BCF-API GitHub repository](https://github.com/BuildingSMART/BCF-API).
Is that what you are talking about?
Being a REST API, you can use it perfectly well from your own .NET add-ins.
The BCF API GitHub repo points to a C# client,
the [BcfApiExampleClient](https://github.com/rvestvik/BcfApiExampleClient).
You can try integrating that very code into a Revit add-in and see how it goes.
Out of interest, I took a slightly deeper look.
I see that BCF stands for [BIM Collaboration Format](http://iug.buildingsmart.org/resources/abu-dhabi-iug-meeting/IDMC_017_1.pdf).
I also see that the C# sample can easily be integrated into a Revit add-in.
\*\*Question:\*\* Does the Revit API create the JSON for BCF or would we be required to create it ourselves via the add-in?
\*\*Answer:\*\* I was not aware of BCF until you raised this question, and I am pretty sure that Revit does nothing to support it.
If it is BCF specific, you will probably have to create it with your own custom add-in.
I created a new project to try out converting the BcfApiExampleClient sample to a Revit add-in,
[RvtBcfApiExampleClient](https://github.com/jeremytammik/RvtBcfApiExampleClient).
It is a pure and simple REST API client.
I had some issues getting both the original sample and my Revit version up and running.
They were resolved over the week-end, however, and both the stand-alone sample client and the new Revit add-in version now work fine.
Here are some screen snapshots running the Revit add-in version:
The first step is the log in to the BIM-IT server:
![BIM-IT login](img/BcfApiExampleClient_login_bim_it.png)
As usual, this authorisation process returns an access token:
![BIM-IT access token](img/BcfApiExampleClient_authorisation.png)
Finally, the sample simply lists all existing projects, of which I possess exactly zero:
![BIM-IT project list](img/BcfApiExampleClient_project_list.png)
If you are interested in diving deeper into the BCF API, please note
the [comment by Georg Dangl](https://github.com/rvestvik/BcfApiExampleClient/issues/2),
saying, "in December we had another Hackathon in Munich where Veni Lillkall created
a [small sample app in C#](https://github.com/BIMit/BCF-Hackathon-Munich/tree/Team_C%23).
That might give you some ideas how to use the BCF API itself."
#### The Kayak Framework – an Easy Way to Speak HTTP with .NET
One interesting aspect of the BCF-API sample is its use
of [the Kayak framework – an easy way to speak HTTP with .NET](http://dotnetslackers.com/articles/aspnet/The-Kayak-Framework-An-easy-way-to-speak-HTTP-with-NET.aspx),
published in 2010 by [Benjamin van der Veen](http://dotnetslackers.com/community/members/bvanderveen.aspx),
including [Kayak sample code](http://dotnetslackers.com/code/KayakCMSExample.zip):
- Kayak is a lightweight HTTP server for the CLR, and the Kayak Framework is a utility for mapping HTTP requests to C# method invocations. With Kayak, you can skip the bulk, hassle, and overhead of IIS and ASP.NET. Kayak enables you to do more with less syntax, and is easy to configure to work in any way you care to dream up.
- Kayak is a simple server with an easy-to-use request framework. It automatically maps HTTP verb/path combinations to your methods, deserialises arguments to invocations of those methods from the query string or JSON request body, and serialises the return values as JSON. It's behaviour is configurable, yet simple to use and understand thanks to its limited scope.
#### Manual Setup of a Revit Add-In
As said, I used the BCF API client sample project to record exactly how to manually set up a Visual Studio C# .NET Revit add-in from scratch.
Here are the detailed individual steps I performed for this, with links to the corresponding commits in the GitHub repository:
- [Added pure Visual Studio class library](https://github.com/jeremytammik/RvtBcfApiExampleClient/commit/7fb713de0efb0940191ba20dba7b7b08220c7e62)
- [Renamed Class1 to Command and edited properties](https://github.com/jeremytammik/RvtBcfApiExampleClient/commit/4a98cd78157be9e36abda9b9b85a3156d8fed911)
- [Added Revit API assemblies and implemented external command](https://github.com/jeremytammik/RvtBcfApiExampleClient/commit/ccd0c0f6469425c059567e3b26b75acbc37602b2)
- [Added add-in manifest](https://github.com/jeremytammik/RvtBcfApiExampleClient/commit/1f9f74e83af6cdbbea4541ab03cdbe15a6868754)
- [Added add-in manifest](https://github.com/jeremytammik/RvtBcfApiExampleClient/commit/667a7d8b51292ae3a84bbf59df8d7d2ad9a64886)
- [Added GUID using guidize.exe](https://github.com/jeremytammik/RvtBcfApiExampleClient/commit/4982403505a9baf44e46e36cf5b500d3375af7b9)
- [Implemented post-build event to auto-install add-in for Revit to find](https://github.com/jeremytammik/RvtBcfApiExampleClient/commit/9101faa26d071d6247714c6f4af94171a9814f52)
- [Successfully tested](https://github.com/jeremytammik/RvtBcfApiExampleClient/commit/1912b9586bcb2ec0219f89bc0fc4d8aa98992bd9)
The result is [RvtBcfApiExampleClient release 2017.0.0.0](https://github.com/jeremytammik/RvtBcfApiExampleClient/releases/tag/2017.0.0.0), which is just a naked Revit 2017 C# .NET add-in skeleton.
The final working version presented above after fixing the initial problems with the login process is captured
in [release 2017.0.0.4](https://github.com/jeremytammik/RvtBcfApiExampleClient/releases/tag/2017.0.0.4).
As said, next I will use the generated files to update
the [Visual Studio Revit add-in wizard](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.20) for Revit 2017.