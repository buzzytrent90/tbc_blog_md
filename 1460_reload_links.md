---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:16.072753'
original_url: https://thebuildingcoder.typepad.com/blog/1460_reload_links.html
post_number: '1460'
reading_time_minutes: 4
series: structural
slug: reload_links
source_file: 1460_reload_links.md
tags:
- levels
- references
- revit-api
- sheets
- structural
title: The Building Coder
word_count: 793
---

### Automatically Reload Links After Migration
Today, my colleagues Michael Brian Lee and Miroslav Schonauer share a solution for links that are not automatically reloaded after migrating the model to Revit 2016.
#### Question
I implemented a batch tool to migrate Revit Server 2015 models to Revit Server 2016.
Danny Polkinhorn’s SaveAsNewCentral code was a great starting point, and the tool is successfully migrating the models.
The workflow for the migration is to open 2015 model detached from central with all worksets closed, which starts an automatic upgrade process, then save to the 2016 server.
The assumption was that links will be taken care of automatically.
To verify the migration and upgrade I perform the following steps:
- Open model in Revit Server 2016 with all worksets closed.
- Go to Manage Links.
- Check Saved Path – it seems correct.
- Click on the Reload button.
That displays the following error saying that the linked file cannot be found:
![Link not found](img/reload_links_1.jpeg)
I then click on Reload From, navigate to the exactly the same path as shown in the Saved Path, and the link is loaded.
Any idea of why the Reload does not find the model, even if the path is correct?
It is possible that the automatic upgrade changed the GUID of the models?
How can I get around this?
At the moment I think I'll need to implement and run another batch tool once all models are migrated.
The second tool could reload all links using `RevitLinkLoadResult.LoadFrom` and then sync to central.
Would that work?
Is there any other way around this?
Second question:
In some of my tests, I see some strange characters on some of the server paths:
![Strange characters](img/reload_links_2.png)
What might be causing that, please?
#### Answer
I suspect that this is an artefact of how link resolution is achieved with Revit Server.
Although, from the UI, it looks as if it is resolving by a path, in reality, it is actually resolving by a unique GUID that corresponds to the central model. This allows us to transparently repath a link if it is moved to a new folder or server, or if it is renamed. However, the trade-off is that it cannot automatically repath to a new model replacing an old one, even if the name and location are the same. That’s because each new model, when saved to the server for the first time, is equipped with a new unique GUID.
#### Solution
Thank you for the official confirmation, Mike – it is good to know we are not doing anything fundamentally wrong in our scripts and Revit Server code.
Further good news: we managed to implement a programmatic solution for this Revit Server workflow issue.
Basically, we prototyped a custom add-in command that now fully simulates the manual UI solution, i.e., it automates selecting 'Reload from….' in Manage Links and browsing to basically the same location as the 'Saved Path' shown does.
It took a bit of time to find the right classes as there are no less than four levels of indirection to get from `RevitLinkType` to the 'Saved Path' and then use that in the call to `LoadFrom`.
Here are the most important steps:
```csharp
// Loop all RVT Links
foreach( RevitLinkType typeLink in fecLinkTypes )
{
// ...
// Skip1 - not IsFromRevitServer
if( !typeLink.IsFromRevitServer() )
{
//…
continue;
}
// Skip2 - not ExternalFileReference
// 99% it would already skip above as
// RevitServer MUST be ExternalFileReference,
// but leave just in case...
ExternalFileReference er = typeLink.GetExternalFileReference();
if( er == null )
{
// ...
continue;
}
// If here, we can cache ModelPath related
// info and show to user regardless if we skip
// on next checks or not....
ModelPath mp = er.GetPath();
string userVisiblePath = ModelPathUtils
.ConvertModelPathToUserVisiblePath( mp );
// Skip3 - if ModelPath is NOT Server Path
// 99% redundant as we already checked raw
// RevitLinkType for this, but keep
// just in case...
if( !mp.ServerPath )
{
// ...
continue;
}
// Skip4 - if NOT "NOT Found" problematic one
// there is nothing to fix
if( er.GetLinkedFileStatus()
!= LinkedFileStatus.NotFound )
{
// ...
continue;
}
// Skip5 - if Nested Link (can’t (re)load these!)
if( typeLink.IsNestedLink )
{
// ...
continue;
}
// If here, we MUST offer user to "Reload from..."
// ...
RevitLinkLoadResult res = null;
try
{
// This fails for problematic Server files
// since it also fails on "Reload" button in
// UI (due to the GUID issue in the answer)
//res = typeLink.Reload();
// This fails same as above :-(!
//res = typeLink.Load();
// This WORKS!
// Basically, this is the equivalent of UI
// "Reload from..." + browsing to the \*same\*
// Saved path showing in the manage Links
// dialogue.
// ToDo: Check if we need to do anything
// special with WorksetConfiguration?
// In tests, it works fine with the
// default c-tor.
ModelPath mpForReload = ModelPathUtils
.ConvertUserVisiblePathToModelPath(
userVisiblePath );
res = typeLink.LoadFrom( mpForReload,
new WorksetConfiguration() );
Util.InfoMsg( string.Format(
"Result = {0}", res.LoadResult ) );
}
catch( Exception ex )
{
// ...
}
}
```
Many thanks to Miro for solving and sharing this solution!