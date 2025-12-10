---
post_number: "1766"
title: "Khl Snoop"
slug: "khl_snoop"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'python', 'references', 'revit-api', 'sheets', 'windows']
source_file: "1766_khl_snoop.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1766_khl_snoop.html"
---

### New PC, KLH Snoop, Annotations and Vertex Handling
I have been very busy and motivated indeed setting up a new computer this week.
Nonetheless, I was able to keep going full steam with Revit API related issues as well:
- [Setting up a new MacBook](#2)
- [KLH Engineers RevitDeveloperTools snooping tool](#3)
- [Pulling text from annotation tags](#4)
- [Vertex handling](#5)
- [The true meaning of pizza](#6)
![Baby lizard on MacBook](img/baby_lizard_on_macbook_1008.jpg)
#### Setting up a New MacBook
I picked up my new computer on Monday, a MacBook Pro.
Migration worked very well so far, installation was easier than ever before, and I am already almost completely done moving over.
These components are now up and running:
- Google Chrome
- Pulse Secure
- Microsoft Office
- Outlook credentials
- Komodo Edit
- iterm2
- Parallels
- Windows 10
- Visual Studio 2017
- Revit 2020
- VeraCrypt and osxfuse
- Skype
- VLC
- Firefox and downloadhelper
- git
- Personal Komodo editor scripts
- Created and set up new GitHub SSH key
- Migrated personal disk data to new machine
- Restored bash profile
- Python 3.7.4
- [pip](https://pypi.org/project/pip) – `pip3 install --upgrade pip`
- [Python markdown](https://pypi.python.org/pypi/Markdown): `pip3 install Markdown`
- [git-lfs](https://git-lfs.github.com) – [installation](https://help.github.com/en/articles/installing-git-large-file-storage)
- Internet browser bookmarks
A few security related issues remain to do:
- Desktop Duo for VPN access
- Replacing
my [obsolete TrueCrypt encryption system](https://www.comparitech.com/blog/information-security/truecrypt-is-discoutinued-try-these-free-alternatives)
I am writing this blog on the new system, with significant help and inspiration provided by my baby lizard friend presented in the image above.
By chance, now that I almost finished, I also just read this helpful article
on [how to set up your new MacBook for coding](https://www.freecodecamp.org/news/how-to-set-up-a-brand-new-macbook) by Amber Wilkie
on [freeCodeCamp.org](https://www.freecodecamp.org).
I have already completed many of the steps she suggests; still, a few are new and useful for me as well.
#### KLH Engineers RevitDeveloperTools Snooping Tool
More Revit API related, John D'Alessandro, Software Engineer at [KLH Engineers, PSC](http://www.klhengrs.com) shared a powerful new Revit database snooping utility, explaining:
> We at KLH Engineers decided to roll our own Revit Snoop tool a while ago and we thought we’d show it to you so you can check it out.
![KLH Snoop](img/klh_snoop_screenshot.png)
> We released it as a class library on our GitHub so that others can use it in their own tools and solutions.
> We’re also accepting issues and PRs, so the more people use the tool the better it’ll get. Let me know if you have any feedback on it or questions.
> The repo is at [KLH Engineers RevitDeveloperTools](https://github.com/klhengineers/RevitDeveloperTools).
> It has some similarities and overlap with [RevitLookup](https://github.com/jeremytammik/RevitLookup), and adds a number of features not available there.
> The README includes a feature list comparing the two snooping tools:
![KLH Snoop feature list](img/klh_snoop_feature_list.png)
Many thanks to John and KLH for implementing and generously sharing this powerful new snooping tool!
#### Pulling Text from Annotation Tags
One quick note on a small issue that was not discussed in
the public [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) seems
worthwhile sharing here:
\*\*Question:\*\* I'm trying to pull the text from the annotation tags on a drawing using the Revit API.
Is that possible?
How?
\*\*Answer:\*\* Yes, this is definitely possible.
You can use a filtered element collector to collect all the annotation tags, and then access their text via their properties.
To retrieve the tags, you need to determine what class they have. I assume they
are [IndependentTag class instances](https://www.revitapidocs.com/2020/e52073e2-9d98-6fb5-eb43-288cf9ed2e28.htm).
The property to access their text is presumably
the [TagText property](https://www.revitapidocs.com/2020/8e297dee-920d-f620-6198-0bed494e3f04.htm).
You can install and use [RevitLookup](https://github.com/jeremytammik/RevitLookup) yourself
to validate these assumptions of mine in your own specific model.
If these assumptions are correct, the final solution might look something like this in C#:
```csharp
#region Pull Text from Annotation Tags
///
/// Return the text from all annotation tags
/// in a list of strings
/// summary>
List PullTextFromAnnotationTags(
Document doc )
{
FilteredElementCollector tags
= new FilteredElementCollector( doc )
.OfClass( typeof( IndependentTag ) );
return new List( tags
.Cast()
.Select(
t => t.TagText ) );
}
#endregion // Pull Text from Annotation Tags
```
For future reference, I also added this code to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) in
the [CmdCollectorPerformance.cs module](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs#L1223-L1240)
Finally, you can use the built-in Revit macro IDE (integrated development environment) to convert this code from C# to Python, if you like.
#### Vertex Handling
Another recent recurring question on vertex handling may also be of general interest:
\*\*Question:\*\* When processing meshes or other collections of triangles, is there any way to know if one vertex is shared by multiple triangles?
Thus, the final data does not have to contain duplicate vertices and can reuse the existing vertex indices in order to save data size.
\*\*Answer:\*\* I have often used strategies similar to the following when collecting mesh data:
- Implement a comparison operator for the mesh vertex coordinate data.
- Store all mesh vertices in a dedicated dictionary using the mesh vertex coordinates as keys, e.g., `Dictionary`.
The `int` is not really important, just the keys.
I often use the `ìnt` value to count the number of vertices encountered at each location.
- For each new mesh vertex received, check whether it is already listed in the dictionary.
If so, increment its count; else, add a new entry for it.
The comparison operator needs to accommodate an appropriate amount of [fuzz](https://thebuildingcoder.typepad.com/blog/2017/12/project-identifier-and-fuzzy-comparison.html#3),
cf. this [dimensioning application](https://thebuildingcoder.typepad.com/blog/2018/12/rebars-in-host-net-framework-and-importance-of-fuzz.html#4).
For some examples, look at `GetCanonicVertices`
for [tracking element modification](https://thebuildingcoder.typepad.com/blog/2016/01/tracking-element-modification.html) and
this [post on the `XYZ` class](https://thebuildingcoder.typepad.com/blog/2017/08/birthday-post-on-the-xyz-class.html#3).
For yet more examples and use cases, you can search The Building Coder for
- `XyzComparable`
- `XyzEqualityComparer`
- `XyzProximityComparer`
#### The True Meaning of Pizza
I am writing this visiting my friend Dani in Verbania, Italy.
Hence, this little note in closing:
Did you ever wonder why pizza is called pizza?
Well, pretty obviously, it simply represents the formula used by a hungry mathematician to determine its volume from its radius `z` and thickness `a`:

```
  Volume = π · z 2 · a = pi · z · z · a
```

![Pizza volume formula](img/pizza.jpg)

Pizza volume formula V = pi·z·z·a