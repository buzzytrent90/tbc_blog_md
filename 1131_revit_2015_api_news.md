---
post_number: "1131"
title: "Revit 2015 API News – DevDays Online Recording"
slug: "revit_2015_api_news"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'transactions', 'views']
source_file: "1131_revit_2015_api_news.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1131_revit_2015_api_news.html"
---

### Revit 2015 API News – DevDays Online Recording

As
[the Walrus said](http://www.jabberwocky.com/carroll/walrus.html),
now "the time has come ... to talk of many things", including the highlights of the new Revit API functionality, so here we go!

Welcome to my first post on the Revit 2015 API, disregarding the
[RevitLookup migration to Revit 2015](http://thebuildingcoder.typepad.com/blog/2014/04/revitlookup-for-revit-2015.html),
of course.

A couple of posts on upcoming Revit 2015 product functionality were already published in the past week or so, e.g.:

- [Overview of new features by synergiscad](http://synergiscadblog.com/2014/04/02/whats-new-in-autodesk-revit-2015)
- [Feature list by Revit.King](http://revitking.blogspot.ch/2014/03/new-feature-list-for-revit-2015.html)
- [Highlighted features and enhancements by Paradigm shift](http://www.seandburke.com/blog/2014/03/27/whats-new-in-revit-2015/)
- [Screen snapshots by cadline](http://www.cadlinecommunity.co.uk/Blogs/Blog.aspx?ScoId=cf9f3cac-1748-4f84-a025-07968741c658&returnTo=%2fBlogs%2finsider%2fDefault.aspx&returnTitle=Insider%20Blog)
- [Videos by BIMopedia](http://bimopedia.com/2014/03/27/videos-whats-new-in-revit-2015)

I will therefore skip all product related stuff and dive straight into the API instead.

Here is the Revit 2015 DevDays Online recording material, based on the presentations shown at the confidential Autodesk DevDays conferences in and around December 2013.

- Presentation
  [slide deck](http://thebuildingcoder.typepad.com/devday/2013/online/Revit_2015_API_News/Revit_2015_API_News_Slides.pdf) (23 MB)
  [^](file:////a/devdays/2013/online/Revit_2015_API_News_Slides.pdf)
- Presentation
  [recording](http://thebuildingcoder.typepad.com/devday/2013/online/Revit_2015_API_News/index.html) with table of contents navigation (354 MB)
  [^](file:////a/devdays/2013/online/cam/Revit_2015_API_News/index.html)
- Presentation
  [notes](http://thebuildingcoder.typepad.com/devday/2013/online/Revit_2015_API_News/Revit_2015_API_News_Notes.pdf) (198 KB)
  [^](file:////a/devdays/2013/online/Revit_2015_API_News_Notes.pdf)
- [Sample code incl. RVT models](http://thebuildingcoder.typepad.com/devday/2013/online/Revit_2015_API_News/Revit_2015_API_Samples.zip) (45 MB)
  [^](file:////a/devdays/2013/online/Revit_2015_API_Samples.zip)
- [Sample code excl. RVT models](http://thebuildingcoder.typepad.com/devday/2013/online/Revit_2015_API_News/Revit_2015_API_Samples_No_RVT.zip) (1.5 MB)
  [^](file:////a/devdays/2013/online/Revit_2015_API_Samples_No_RVT.zip)

The next thing I want to do is to publish the "What's New" section from the updated Revit API help file RevitAPI.chm, to ensure it is available for and easily found by online searches, as for the past few releases:

- [What's New in the Revit 2010 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2010-api.html)
- [What's New in the Revit 2011 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2011-api.html)
- [What's New in the Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2012-api.html)
- [What's New in the Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html)
- [What's New in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html)

That is always the first place to explore if you have any questions about or problems with your migration or the new functionality provided.

I wish you lots of fun and excitement exploring the new possibilities provided by the Revit 2015 API!

#### Right Click Circle of Commands GUI Design for Revit

Sticking with what we already have for the moment, another interesting this that just came up on the Revit API forum is this discussion thread initiated by Ryan on
[GUI design for Revit](http://forums.autodesk.com/t5/Revit-API/GUI-design-for-Revit/m-p/4944720).

Ryan suggests developing a secondary user-infterface for Revit, like the right-click command in Fusion 360, or Inventor, a wheel (or circle) showing a pre-determined list of commands.

The Revit user interface could look something like this:

![Revit user interface](img/rb_revit_user_interface.jpg)

Commands placed on circle segments:

![Commands placed on circle segments](img/rb_project_circle_screenshot_in_use.jpg)

Interactive command population:

![Interactive command population](img/rb_img_20131031_091527.jpg)

Now Brett of [BRevit](http://brevitb.blogspot.com.au/) presented a
[20-second video of WheelConcept](https://www.youtube.com/watch?v=lYrpJe2484E&feature=youtu.be),
a working implementation of a right-click circle of commands based on the PostCommand API.

In his own words:

I think a lot can be achieved with the postcommand function. Hope you don't mind but i took your idea and gave it a crack....

Firstly i created a form with the wheel as the background. I created a public property on the form which is the id of the command to issue:

```csharp
  Public CommandToIssue As RevitCommandId
```

I then created transparent picture boxes over the various icons. Clicking on a picturebox asigns a command id and closes the form:

```vbnet
  Private Sub PictureBox2\_Click( \_
    ByVal sender As Object, \_
    ByVal e As EventArgs) \_
  Handles PictureBox2.Click
    CommandToIssue \_
      = RevitCommandId.LookupPostableCommandId( \_
        PostableCommand.StructuralColumn)
    Close()
  End Sub
```

Under the forms show event i added code to locate the form at the mouses current position:

```vbnet
  Private Sub WheelForm\_Shown( \_
    ByVal sender As Object, \_
    ByVal e As EventArgs) \_
  Handles Me.Shown
    Location = New System.Drawing.Point( \_
      CInt(MousePosition.X - (Me.Width / 2)), \_
      CInt(MousePosition.Y - (Me.Width / 2)))
  End Sub
```

Finally I setup a command to display the wheel, and post the command:

```vbnet
Option Strict On
Option Explicit On

Imports Autodesk.Revit.Attributes
Imports Autodesk.Revit.UI

Imports BrevitTools.UI.Wheel

<Transaction(TransactionMode.Manual)> \_
Public Class Wheel
  Implements IExternalCommand

  Public Function Execute( \_
    ByVal cmdData As ExternalCommandData, \_
    ByRef message As String, \_
    ByVal elements As Autodesk.Revit.DB.ElementSet) \_
  As Result Implements IExternalCommand.Execute

    Dim form As New WheelForm
    form.ShowDialog()

    If form.CommandToIssue IsNot Nothing Then
      cmdData.Application.PostCommand(
        form.CommandToIssue)
    End If

    Return Result.Succeeded

  End Function

End Class
```

To display the wheel inside Revit I assigned a shotcut key (ww).

Here is link to a
[video of it in action](https://www.youtube.com/watch?v=lYrpJe2484E&feature=youtu.be).

I think there is loads of potential for this. You could easily add code to check what elements are preselected and then display a contextual wheel.

Thank you very much, Brett, for sharing this!