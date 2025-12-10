---
post_number: "1732"
title: "Snoop Appearance Asset"
slug: "snoop_appearance_asset"
author: "Jeremy Tammik"
tags: ['levels', 'revit-api', 'sheets', 'views']
source_file: "1732_snoop_appearance_asset.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1732_snoop_appearance_asset.html"
---

### Forge Picture, Debugging, Snooping Appearances
Today, let's look at the Forge architecture, Revit add-in debug, edit and continue, and yet another RevitLookup enhancement:
- [High-level picture of Forge](#2)
- [Debug and continue in a Revit add-in](#3)
- [Snooping appearance assets](#4)
#### High-Level Picture of Forge
Would you like to quickly understand
the [Forge](https://forge.autodesk.com) architecture,
including all relevant aspects, without getting mired in its nitty-gritty details?
Check out Scott Sheppard's very cool executive overview in
the [Forge high-level picture for software development managers](https://labs.blogs.com/its_alive_in_the_lab/2019/03/whats-so-hot-about-this-forge-thing-the-high-level-picture-for-software-development-managers.html).
![Forge high-level picture](img/forge_high_level_picture.jpg)
#### Debug and Continue in a Revit Add-In
Developers are continuously seeking reliable, efficient development approaches.
Some ways have been described in the past implementing the functionality
to [edit and continue, and debug without restarting](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.49).
This question arose again in the StackOverflow question
asking [why is my DLL still being used by Revit after execution?](https://stackoverflow.com/questions/55256817/why-is-my-dll-still-being-used-by-revit-after-execution).
Konrad Sobon jumped in and pointed out his solution:
> I did a write-up on my blog that explains how you can use the Revit Add-In Manager to achieve the result you are after:
> - [debugging revit add-ins](http://archi-lab.net/debugging-revit-add-ins)
> The difference between this and a standard method of debugging is that Revit loads the DLL using the `LoadFrom` method, locking it up for as long as the Revit.exe process is running, while the Add-In Manager uses the `Load` method that only reads the `byte[]` stream of the DLL which means it remains available, and you can re-build your solution in VS, and reload in Revit without closing it. It does have drawbacks, obviously, so please read the post.
#### Snooping Appearance Assets
In further support of efficient debugging and Revit database exploration, here is
another [RevitLookup](https://github.com/jeremytammik/RevitLookup) enhancement
enabling snooping of appearance assets, based on two pull requests
by [Victor Chekalin](http://www.facebook.com/profile.php?id=100003616852588), aka Виктор Чекалин:
- [#48 – snoop rendering `AssetProperty`](https://github.com/jeremytammik/RevitLookup/pull/48)
- [#49 – pushed the missed files](https://github.com/jeremytammik/RevitLookup/pull/49)
- [#50 – handle `AssetPropertyDoubleArray4d`](https://github.com/jeremytammik/RevitLookup/pull/50)
The description is sweet and simple:
- Snoop rendering `AssetProperty` – `Material` → `AppearanceAssetId` → `GetRenderingAssset`
This is supported by more than a thousand words:
![Snooping appearance assets](img/revitlookup_snoop_appearance_asset_1.png)

![Snooping appearance assets](img/revitlookup_snoop_appearance_asset_2.png)

![Snooping appearance assets](img/revitlookup_snoop_appearance_asset_3.png)

![Snooping appearance assets](img/revitlookup_snoop_appearance_asset_4.png)

![Snooping appearance assets](img/revitlookup_snoop_appearance_asset_5.png)

I integrated Victor's pull requests
in [RevitLookup release 2019.0.0.11](https://github.com/jeremytammik/RevitLookup/releases/tag/2019.0.0.11).
Many thanks to Victor for this useful enhancement!