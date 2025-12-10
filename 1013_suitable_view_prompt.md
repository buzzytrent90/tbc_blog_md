---
post_number: "1013"
title: "Set a Suitable View for Family Instance Placement"
slug: "suitable_view_prompt"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'revit-api', 'vbnet', 'views']
source_file: "1013_suitable_view_prompt.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1013_suitable_view_prompt.html"
---

### Set a Suitable View for Family Instance Placement

Matt Taylor of [WSP](http://www.wspgroup.com) raised
an issue related to the use of the PromptForFamilyInstancePlacement method, now that API access is also possible in the project browser view.

He also went right ahead to suggest and implement a solution:

**Question:** One of the new features of the Revit 2014 API is the possibility to select elements within the Project Browser view.

If the Project Browser view is active, selecting a ribbon control that uses PromptForFamilyInstancePlacement will fail because the user will try to pick in the active drawing view to place the family instance.

According to the help for PromptForFamilyInstancePlacement: "Users are not permitted to change the active view during this placement operation (the operation will be completed)."

If there is only one active UIView, it's easy to just set it to active prior to calling PromptForFamilyInstancePlacement. If more than one view is open, how can I get the last active drawing view (prior to the Project Browser)?

Short of implementing a ViewActivated event (seems like overkill), or prompting the user to click in a view before using PromptForFamilyInstancePlacement (not always required, inconsistent across 2011/2012/2013), what can be done?

**Answer:** It should be possible and not an all-to-big effort to implement a ViewActivated event handler and simply make a note of each view activated. Then, before calling PromptForFamilyInstancePlacement, revert to the last valid one for that operation. Would that solve the issue for you?

**Response:** Here is the
[zipped view activated solution](zip/mt_suitable_view.zip),
mainly copied from your blog’s example, though in VB.NET.

It was substantially simpler than I envisaged – an hour of work to implement.

In OnStartup, we instantiate a dictionary mapping each document to the last valid PromptForFamilyInstancePlacement view and subscribe to the view activated event:

```vbnet
  ''' <summary>
  ''' Map document title to last valid view element id
  ''' </summary>
  Public Shared viewHash As Hashtable

  Public Function OnStartup( \_
    ByVal application As UIControlledApplication) \_
  As Result Implements IExternalApplication.OnStartup

    Try
      viewHash = New Hashtable

      AddHandler application.ViewActivated,
        AddressOf OnViewActivated

      Return Result.Succeeded

    Catch ex As Exception

      Return Result.Failed
    End Try

  End Function
```

In the view activated event handler, we simply make a note of the last view activated for each document, provided it is a valid view to call the PromptForFamilyInstancePlacement in, i.e. not a system browser or project browser view:

```vbnet
  Private Shared Function ViewDescription( \_
    ByVal v As DB.View) As String

    Return String.Format(
      "view '{0}' in document '{1}'",
      v.Name, v.Document.Title)

  End Function

  Private Sub OnViewActivated( \_
    ByVal sender As Object,
    ByVal e As UI.Events.ViewActivatedEventArgs)

    Dim docTitle As String = e.Document.Title
    Dim vPrevious As DB.View = e.PreviousActiveView
    Dim vCurrent As DB.View = e.CurrentActiveView
    If Not (vCurrent.ViewType = ViewType.SystemBrowser \_
            OrElse vCurrent.ViewType = ViewType.ProjectBrowser) Then
      If viewHash.ContainsKey(docTitle) Then
        viewHash.Item(docTitle) = vCurrent.Id
      Else
        viewHash.Add(docTitle, vCurrent.Id)
      End If
    End If

    Dim s As String = If(
      (vPrevious Is Nothing),
      "no view at all",
      "previous " + ViewDescription(vPrevious))

    Debug.Print(
      String.Format(
        "Switching from {0} to new {1}.",
        s, ViewDescription(vCurrent)))

  End Sub
```

With that information available, we can always switch to the last valid view before calling the PromptForFamilyInstancePlacement method, e.g. like this:

```vbnet
  ' According to the help for
  ' PromptForFamilyInstancePlacement:
  ' "Users are not permitted to change the active
  ' view during this placement operation (the
  ' operation will be completed)."
  ' Here's the fix, which only needs to work for 2014.
  ' (With previous versions, the ribbon items are
  ' greyed out if the project browser is active.)

  Dim actView As DB.View = doc.ActiveView

  If actView.ViewType = DB.ViewType.SystemBrowser \_
    OrElse actView.ViewType = DB.ViewType.ProjectBrowser Then

    ' Get stored view id.

    If AppCommand.viewHash.ContainsKey(doc.Title) Then

      Dim prevValidViewId As DB.ElementId \_
        = AppCommand.viewHash.Item(doc.Title)

      Dim elem As DB.Element = doc.GetElement(
        prevValidViewId)

      If elem IsNot Nothing Then
        Dim view As DB.View = TryCast(elem, DB.View)
        If view IsNot Nothing Then

          ' TODO: Check that view is valid for the
          ' placement of this type of family symbol,
          ' otherwise get another view.
          ' (Likely to be the case, as the user
          ' will normally click a ribbon item as
          ' part of the context of what they're doing.)

          docUI.ActiveView = view

        End If
      End If

    End If
  End If

  docUI.PromptForFamilyInstancePlacement(
    familySymbol)
```

Many thanks to Matt for pointing this out and providing such a nice solution!

I'll add a couple of non-Revit-API-related notes:

#### Autodesk University Brazil

[Autodesk University Brazil](http://aubrasil.autodesk.com) is taking place in São Paulo on October 10, 2013.
Explore the program, mark your calendar and register at [aubrasil.autodesk.com](http://aubrasil.autodesk.com).

#### Building Performance Analysis Certificate Program

Autodesk launched the
[Building Performance Analysis (BPA) Certificate Program](http://sustainabilityworkshop.autodesk.com/bpac).

This free online course for architecture and engineering students teaches the building science fundamentals for designing high-performance buildings.
Through self-paced online tutorials, quizzes and Autodesk software exercises, the BPA Certificate Program gives students the skills to help drive an industry-wide transition to performance-based sustainable design.
It provides:

- Seven modules that include climate analysis, sun path studies, building massing and orientation, solar radiation analysis, wind analysis, and more.
- A clear 'introduction to software' section within each of the seven modules.
- Small clusters of content and quizzes that students can complete in short amounts at a time.
- Case-based examples and questions.
- Content focused on energy fundamentals and modeling with detailed Revit models.
- Application of Revit-based tools including Revit, Vasari and Green Building Studio.

This program takes an estimated 20 hours to complete.
Following successful completion, students are issued a certificate and a badge that they can place on their resume, LinkedIn profile or portfolio.
Students will improve and prove their fluency in sustainable building design strategies and tools.

#### Factory Design Mobile App

The
[Factory Design Mobile](https://itunes.apple.com/us/app/autodesk-factory-design/id657389678?ls=1&mt=8),
available on the Apple App Store, enables iPad users to create and review factory layouts in the field.

It presents a 2D layout environment to conceptualize designs, adjust them based on field observations and collaborate with partners.

Besides being an independent conceptual design tool for factory layouts, FDM also supports synchronization with existing designs.
You can review your design live on the factory floor and synchronize changes from the floor to the final design, including updating both 2D and 3D representations of the factory.