---
post_number: "1385"
title: "External Service"
slug: "external_service"
author: "Jeremy Tammik"
tags: ['csharp', 'filtering', 'geometry', 'levels', 'python', 'references', 'revit-api', 'sheets', 'views']
source_file: "1385_external_service.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1385_external_service.html"
---

### External Services
It's been a while since I talked about External Services.
The External Services framework was initially introduced and used sparingly in
the [Revit 2013 MEP API](http://thebuildingcoder.typepad.com/blog/2012/05/the-revit-2013-mep-api-and-external-services.html).
In Revit 2014,
the [IFC export was moved to an external service](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) and
I presented
the [user-defined MEP calculation sample](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html),
later [moved to GitHub](http://thebuildingcoder.typepad.com/blog/2013/11/user-mep-calculation-sample-on-github.html).
The framework for handling [referenced file resources](http://thebuildingcoder.typepad.com/blog/2014/07/referenced-files-as-a-service.html) is
also implemented as an external service, using an implementation called the ExternalResourceService.
The motivation to continue this discussion now was given by Arnošt Löbel's class SD10752 on Revit External Services at Autodesk University 2015 in Las Vegas.
![Arnošt Löbel](img/372_arnost_lobel_600x520.jpg)
#### Hashmas Hangover Run
Before getting to that, I'll just mention that I joined
the [Hashmas Hangover run](http://www.meetup.com/Basel-Hash/events/226867168) organised by
the [Basel Hash House Harriers](http://www.meetup.com/Basel-Hash) for
a jog around the nice little Swiss town of Liestal last Sunday:
[![Hash House Harriers Basel Likk'mm trail run on December 13 2015](img/hhhbsl_2015-12-13.png)](http://harrier.ch/Likkmm/CH-BaselH3/Events/BaHHH.20151213)
Many thanks to our incomparable hare [Likk'mm](http://harrier.ch/Likkmm), who laid the trail and provided the GPS track.
#### Revit External Services – Make Built-in Features Behave Your Way
The Revit External Services framework enables developers to tailor the behavior of built-in Revit features to a level impossible to achieve with conventional external command-based add-ins. External Services have been used as a base to support customized MEP (mechanical, electrical, and plumbing) calculations, access externally stored data, replace built-in export/import filters, and many other Revit features. As Revit continues to grow there are new external services introduced with every major release, which gives developers new opportunities to provide their solutions on top of the Revit core functionality. Unfortunately, this feature has not been widely used by the yet, possibly due to the lack of comprehensive documentation, which is exactly the gap this class will attempt to fill. We explain the framework's main ideas, clearly illustrate the most anticipated uses and present a hands-on coding approach using a simple MEP friction calculator as an example.
Learning objectives:
- Learn how to identify all the parts that make up the Framework of External Services
- Discover the framework's main workflows, expectations, and limitations
- Learn how to find, organize, and use built-in external servers and services
- Learn how to design and implement a custom server for a published service (e.g., a MEP friction calculator)
Below are Jeremy's session notes taken during the presentation, followed by the official [handout and class material](#6).
#### Session Notes
This framework was introduced in Revit two years ago.
There is scant documentation for it.
Here is the agenda:
- Five-mile view, philosophy and design.
- Classes, implementation, workflows.
- Code samples.
It could be used, e.g., to replace a spell checker. Note that this is a purely hypothetical example for the audience to better understand the workflow. Spell-checking in Revit is not really implemented as an external service.
In a nutshell: delegate a process to an external application for redefinition, enabling feature componentising and behaviour customisation.
The ApplicationInitialized event was implemented two years back explicitly so that external servers can register after all external services have been registered, i.e., after all OnStartup calls have completed.
Currently, we are renovating Revit inside out and adding a large number of services.
We implemented two sample applications just for this class:
- [Server explorer](#4)
- [Revit MEP calculation](#5)
Note that these samples currently register their service in external commands.
This is totally unrealistic and only intended for demonstration purposes.
A real external server would register its service during start-up instead.
Note that the Revit SDK samples are not designed to be perfect and fully fleshed out, just demonstrate the basic code using the API.
#### Server Explorer
The first sample is the Server Explorer.
It lists all external servers and the services they provide.
Here are the results of running the server explorer sample in Revit 2016 SP2:
![Server Explorer Sample](img/sd10752_revit_external_service_explorer.png)
MEP calculations are one of the most important uses of external services.
As you can see from the output above, there are currently four servers for duct fitting pressure drop calculation.
#### Revit MEP Calculation
The second example is a Revit MEP calculation external service sample for a pressure drop calculation.
It was implemented specifically for this AU class and, as mentioned, uses external commands to add and remove the calculator. A proper calculator will be implemented as an ExternalDBApplication with no external commands involved.
It currently just demonstrates the principles and returns hardcoded dummy calculation results.
#### Handout and Class Material
The two sample applications discussed above are included in
the [zip file containing the SD10752 class material](/a/doc/au/2015/doc2/sd10752_revit_external_services_arnost_lobel_samples.zip).
Jeremy's note to self:
the Revit 2014 [UserMepCalculation](https://github.com/jeremytammik/UserMepCalculation) sample
is now superseded by this one and should be replaced.
Here is Arnošt's [handout document](/a/doc/au/2015/doc2/sd10752_revit_external_services_arnost_lobel_handout.pdf).
The slide deck is very similar.
Many thanks to Arnošt for preparing and presenting all this important material!