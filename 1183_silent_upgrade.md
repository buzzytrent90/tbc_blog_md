---
post_number: "1183"
title: "Upgrading Family Files Silently, Part 2"
slug: "silent_upgrade"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'parameters', 'references', 'revit-api', 'transactions', 'windows']
source_file: "1183_silent_upgrade.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1183_silent_upgrade.html"
---

### Upgrading Family Files Silently, Part 2

Last week, I provided some suggestions for
[upgrading family files silently](http://thebuildingcoder.typepad.com/blog/2014/07/upgrading-family-files-silently.html),
i.e. suppressing the warning messages displayed for every family file that requires updating when being loaded into a project, requiring a user confirmation for each one:

1. Use a [Revit file updater](http://lmgtfy.com/?q=revit+file+updater),
   e.g. the
   [file upgrader on Autodesk labs](http://labs.blogs.com/its_alive_in_the_lab/2011/07/updated-file-upgrader-for-revit-now-available.html)
   provided by the ADN team.
2. Handle the pop-up messages yourself using one of the techniques for
   [handling dialogues and failures](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32).
3. [Determine the Revit version](http://thebuildingcoder.typepad.com/blog/2013/01/basic-file-info-and-rvt-file-version.html) used
   to save the family files before trying to load them.

The developer tried out these three suggestions and we will take a look at the results, followed by a reiteration of a different observation, a warm welcome to my new colleague Jaime, and congratulations on his first blog post:

- [Revit file updater](#2.1)
- [Handling the dialogue](#2.2)
- [Determining the Revit file version](#2.3)
- [Each ribbon button requires a separate external command](#3)
- [Welcome, Jaime, and congratulations on your first post!](#4)

#### 1. Revit File Updater

a. I tried looking for the add-in at the Autodesk lab link, but the page is no longer available.

b. I searched a bit and saw that
[the file updater add-in was moved to Autodesk Exchange](http://labs.blogs.com/its_alive_in_the_lab/2013/07/graduatedretired-adn-plugins-of-the-month.html).
I looked, but couldn't find it there either.

c. I did find add-ins made by others, but haven't looked at any of them.

d. It would be nice if the sample code were still around somewhere, so that we can see how to do it correctly if we plan to upgrade the family files before importing.

#### 2. Handling the Dialogue

The dialogue we're trying to close/hide is this:

![Upgrade message](img/upgrade_message.png)

It appears when we use doc.LoadSymbol and the family file needs upgrading.
It goes away after upgrading without user input, but it does pop up again if we're loading symbols from many files.

I tried using the DialogBoxShowing event, but I think this is one of those dialogues that do not broadcast that event.

I tried using the Failure API by creating a class implementing IFailuresPreprocesser and using the method call

```csharp
  transaction.SetFailureHandlingOptions( options );
```

Unfortunately, this didn't work as expected.
The call to IFailuresPreprocessor.PreprocessFailures did happen (the breakpoint was captured), but it only gets called when the transaction is committed:

```csharp
  // The dialogue appears in this line:

  doc.LoadFamilySymbol( familyfilePath,
    typeName, out symbol );

  // The failure preprocesser is called here:

  transaction.commit();
```

I haven't tried the Windows API method to close the window, but I think I'll only use that as a last resort only.

#### 3. Determining the Revit File Version

The BasicFileInfo method worked wonderfully.
I was able to get the Revit version the family files were saved in.

In conclusion, it would be nice if the LoadFamilySymbol would provide an overload something like

```csharp
  doc.LoadFamilySymbol( familyFilePath,
    typeName, ignorePrompts, out symbol );
```

For now I'll use BasicFileInfo to gather information about the saved version of the family files we're going to load into the document and inform the user that some warning messages will popup because some files needs to be upgraded.
If the ADN Revit file updater is still around, perhaps we can even ask the user if they would like the family files to be upgraded automatically too.

**Answer:** Here is the complete
[file upgrader source code](zip/FileUpgrader_2014-07-16.zip),
Visual Studio solution and add-in manifest.

Sorry to hear that the other methods did not help, and thank you for trying them out.

I agree that dismissing the dialogue via the Windows API is best left as a last resort.

Would you like to share a code snippet showing how you use the BasicFileInfo as you describe?

I do not think there is any sample code of using that on the net yet.

Probably, the file upgrader approach could be combined with that to fully automate the upgrade process.

It might be possible to use the BasicFileInfo to determine which files need upgrading, display the list in a message box to the user, ask for confirmation to upgrade, and then run the upgrader automatically before loading the resulting updated models.

**Response:** I used this to get the BasicFileInfo:

```csharp
  var familyFilePath = "FamilyFile.rfa";

  var fileInfo = BasicFileInfo.Extract(
    familyFilePath );
```

The file info provides two properties of interest.

- fileInfo.IsSavedInCurrentVersion returns true if the version of the family file is the same as the Revit version.
- fileInfo.SavedInVersion returns a string with the full Revit name and version.
  For example, for a family file saved in Revit 2015, it returns "Autodesk Revit Architecture 2015".

Thanks for including the file updater code.
It's good reference.
From looking at the code, each family file is opened and saved in the current version.
I'll try to implement it that way.
And yes, we'll need to ask the user if they would like to upgrade the files.
It's possible they might not want to.

Many thanks for this interesting and fruitful discussion!

#### Each Ribbon Button Requires a Separate External Command

**Question:** I would like to implement only one single external command and let it handle all ribbon button clicks for my external application. The motivation for this is to increase or decrease the number of ribbon buttons at Revit startup depending on the state of other, external resources. How can my external command determine which ribbon button triggered its Execute method? For instance, can I determine the label of the clicked button?

**Answer:** Neither does the Revit API support this, nor can I think of any other method to achieve it.

The best way to go is to define a separate external command for each button.

If you wish, you can then call a common handler for all commands, calling one single method from each of the different Execute methods, with an additional parameter specifying which button was clicked.

For more detailed background information and suggestions, please refer to this discussion on
[implementing a single command for multiple buttons](http://thebuildingcoder.typepad.com/blog/2012/11/happy-bhai-dooj.html#2).

**Addendum** by
[Guy Robinson](http://www.redbolts.com):
Couldn't you just set the buttons to be enabled or visible to achieve the desired effect? This setting can be updated from an external command whenever a service becomes available or unavailable.

#### Welcome, Jaime, and Congratulations on Your First Post!

We have a new member in the ADN DevTech team:
[Jaime Rosales Duque](http://adndevblog.typepad.com/aec/jaime-rosales.html) joined
us from a different department in Autodesk.
He was part of the BIM 360 Glue team and worked on add-ins for various Autodesk products, including Revit, Navisworks, AutoCAD and its verticals.

Jaime is also a certified NYC bartender and passionate photographer.
He relates these hobbies to his coding passion, saying that a well done, tasty cocktail and an expressive and unique picture can be related to a well written, maintainable and scalable piece of code   :-)

Jaime now published his first blog post on
[FamilyType values and aerial pictures](http://adndevblog.typepad.com/aec/2014/07/familytypes-value-aerial-pictures.html).

Congratulations, Jaime, and thank you very much!