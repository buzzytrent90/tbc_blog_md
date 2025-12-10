---
post_number: "1558"
title: "Right Project Loc"
slug: "right_project_loc"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'sheets']
source_file: "1558_right_project_loc.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1558_right_project_loc.html"
---

### Finding the Right Project Location
Here is an explanation of the Revit project location elements and selecting the right one to use.
#### Question
The sample project 'rac_basic_sample_project.rvt' for Revit 2016 defines project location elements: 'Internal 21748' and 'Project 111428'.
![Project locations](img/project_location_1.png)
The active Project Location is 'Internal 21748'.
![Active project location](img/project_location_2.png)
However, the values in 'Project 111428' are correct and can be used to transform the coordinates, whereas the values in 'Internal 21748' are nonsense.
That leads to the following questions:
1. Why is the active project location 'Internal 21748' and not 'Project 111428'?
2. How can I determine which of the two to use when retrieving all project location elements, given that I cannot rely on the active project location?
3. What is the difference between these two? Why do they exist? Are there always exactly two project locations in a Revit project?
#### Answer
In the case you describe, the `Document.ProjectLocations` property will return only one single `ProjectLocation` element – it will hide the location named "Project'.
I have to explain a little bit about how Revit works internally here. A normal Revit model has two SiteLocations – one tied to the shared coordinates, and one tied to the project coordinates. Each one starts with one `ProjectLocation`. Those are the two ProjectLocations you mention – 'Internal' for the shared coordinates, and 'Project' for the project coordinates.
The API does not expose the project ones directly, because they are confusing and not particularly helpful for normal use, as you've seen.
If you created more ProjectLocations via the Location/Weather/Site dialog on the Manage tab, they'd share the same SiteLocation as the Internal location.
As far as coordinates are concerned – assuming you have a ProjectLocation `projLoc` and you execute the following:
```csharp
ProjectPosition pos = projLoc.GetProjectPosition( XYZ.Zero );
TaskDialog.Show( "Revit", pos.EastWest );
```
This will tell you the relationship between the project coordinates and shared coordinates when that location is active. For example, in the little test I did to check my understanding, it says 18, and my project base point is 18 feet east of my survey point.
I don't know what the 'Project' ProjectLocation's coordinates mean in this circumstance, because it would be comparing to itself. Possibly it's a transform between Revit's internal origin and the project coordinates.
If `Document.ProjectLocations` does actually return both ProjectLocations in Revit 2016, you should simply ignore the 'Project' one. It's part of the project coordinates and isn't actually a ProjectLocation in the sense of the locations you see in Location/Weather/Site.
This information is given in terms of the 2018 API. It should mostly apply to Revit 2016 as well. I think the code internals are the same. You might need to make small trivial changes, such as use `get_ProjectPosition()` instead of `GetProjectPosition()` or similar.
Many thanks to Diane Christoforo for this illuminating explanation!