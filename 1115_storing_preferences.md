---
post_number: "1115"
title: "Storing Revit Add-in Settings"
slug: "storing_preferences"
author: "Jeremy Tammik"
tags: ['csharp', 'references', 'revit-api', 'views', 'windows']
source_file: "1115_storing_preferences.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1115_storing_preferences.html"
---

### Storing Revit Add-in Settings

Here is another tip from the Revit API discussion forum that seems worthwhile cleaning up and making easy to find, on
[how to store plugin preferences](http://forums.autodesk.com/t5/Revit-API/How-to-store-plugin-Preferences/m-p/4844497),
raised and discussed by
[Dimi](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/1951637),
[peterjegan](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/1090898) and
[ollikat](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/774564):

**Question:** I am trying to find a way to store user preferences for my plugin.

These preferences are file paths, import options etc. so that the next time the user runs the plugin, all the choices he made the fist time he used the plugin stay the same.

Are there any guidelines how to achieve this, or API functionality that helps?

**Answer:** If the settings you wish to store are Revit project specific, you can save them in the RVT document in
[Extensible Storage](http://thebuildingcoder.typepad.com/blog/2011/04/extensible-storage.html) via
the Revit API. This would fit a scenario where a user wants to have different settings in different projects. Here are some more
[extensible storage topics](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.23).

If your settings contain general add-in related information that has no direct relationship with the Revit project, it might be better to use the dedicated Visual Studio C# feature made exactly for this purpose:
[Using settings in C#](http://msdn.microsoft.com/en-us/library/aa730869(v=vs.80).aspx).

This is very easy and effective system for storing add-in data.

Some people who want more control develop their own system, but I suggest starting here.

**Response:** The C# Visual Studio User Settings system suggested above worked fine.

Nevertheless, in my case, it's even easier, since I am storing user settings for Windows Forms entries.

You can easily associate user settings with Windows Form components inside the Form Designer:
[How to: Create Application Settings Using the Designer](http://msdn.microsoft.com/en-us/library/wabtadw6%28v=vs.110%29.aspx).

This means even less code for reading and writing.

The only caveat I found is that you have to call save before closing the form, otherwise the settings will be lost:

```csharp
Properties.Settings.Default.Save();
```