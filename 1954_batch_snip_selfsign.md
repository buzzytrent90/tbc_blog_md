---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: qa
optimization_date: '2025-12-11T11:44:17.122841'
original_url: https://thebuildingcoder.typepad.com/blog/1954_batch_snip_selfsign.html
post_number: '1954'
reading_time_minutes: 12
series: general
slug: batch_snip_selfsign
source_file: 1954_batch_snip_selfsign.md
tags:
- doors
- elements
- family
- filtering
- levels
- parameters
- references
- revit-api
- sheets
- views
- windows
title: Batch Snip Selfsign
word_count: 2440
---

### Parameters, Snippets, Batch Mode and Self-Signing
Picking up some specially interesting topics from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) and
elsewhere:
- [Revit 2023 parameters service cloud](#2)
- [Roll your own verified publisher](#3)
- [Reset the unsigned add-in security warning](#3b)
- [Remove the code signing certificate](#3c)
- [Revit API code snippet repository](#4)
- [Batch processing and monitoring progress](#5)
#### Revit 2023 Parameters Service Cloud
In Revit 2023, a new cloud-based parameters service is under evaluation.
Learn all about it in the two-and-a-half-minute video
on [Revit 2023 Parameters Service](https://youtu.be/cRz7kQz88mA).
> Manage your parameters library more efficiently across projects and locations using the Parameters Service with Revit and the Autodesk Construction Cloud...
It’s currently in a preview phase with no API, yet.
API support will presumably be coming along after the evaluation phase.
#### Roll Your Own Verified Publisher
Are you getting tired of struggling with the Revit add-in security warning about an unsigned add-in saying, \*the publisher of this add-in could not be verified\*?
Konrad Sobon of [archi+lab](https://archi-lab.net) explains how
to [create a self-signed code signing certificate](https://archi-lab.net/creating-a-self-signed-code-signing-certificate) for yourself for free:
> ... *[detailed explanation]* ... That’s it!
That should create your PFX file that you can now use with signtool, and code sign your Revit plugins for free!
This self-signed code signing certificate won’t expire for another 17 years so you should be good to go for a while.
Now, be aware of the fact that this self-signed code signing certificate is not the same as one issued to you by a 3rd party.
I guess the level of “trust” here would be a little different, but in this particular case, I don’t think it matters to me.
I am fed up with paying money to companies that have just atrocious customer support.
If you are using these code signing certificates literally to just sign Revit plugins, then there is no reason to obtain one from a 3rd party and pay a hefty price for it on top of all the hoops that they will make you jump through.
I hope this helps some of the AEC development community out there save some money and time.
Thank you very much, Konrad for digging in so deep and sharing this very helpful approach!
For the sake of completeness, you can also check out
Konrad's previous post on [code signing of your Revit plug-ins](http://archi-lab.net/code-signing-of-your-revit-plug-ins),
the main [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [Code signing of Revit Addins](https://forums.autodesk.com/t5/revit-api-forum/code-signing-of-revit-addins/m-p/5981560)
and previous articles on this by The Building Coder:

* [What's New in the Revit 2017 API – Code signing of Revit Addins](http://thebuildingcoder.typepad.com/blog/2016/04/whats-new-in-the-revit-2017-api.html#2.4)
* [How Does Code Signing of Revit Add-Ins Increase Security?](http://thebuildingcoder.typepad.com/blog/2016/09/trusted-signature-motivation-and-fishing.html#2)
* [Revit Add-In Code Signing YAML](https://thebuildingcoder.typepad.com/blog/2020/09/code-signing-preview-and-element-type-predicates.html#2) to automate code signing in CI

#### Reset the Unsigned Add-In Security Warning
If, on the contrary, you wish to see the message, and accidentally disabled it, Eatrevitpoopcad's discovery how
to [reset the unsigned add-in security warning](https://forums.autodesk.com/t5/revit-api-forum/reset-the-unsigned-add-in-security-warning/td-p/11090662) will
come in useful:
\*\*Question:\*\* Does anyone know how to reset the security warning Revit gives you when loading unsigned add-ins?
After you click \*Always Load\* and you never see it again?
Let's say I pressed \*Always Load\* and then changed my mind and would like to keep seeing that message:
![Unsigned add-in security warning message](img/unsigned_addin_warning_message.png "Unsigned add-in security warning message")
\*\*Answer:\*\* I found it in the registry!
Here is how to reset code signing warnings:
- Go to START
- Enter REGEDIT
- Go to the following registry key:
\*HKEY_CURRENT_USER\SOFTWARE\Autodesk\Revit\Autodesk Revit 2022\CodeSigning\*
You will see a bunch of GUID value names with data values 0 or 1.
Each one is for a different plugin you codesigned.
Select them all and delete them.
This is for Revit 2022; for other versions of Revit, go up a folder and pick the Revit version for which you would like to clear code signing entries.
Many thanks to Eatrevitpoopcad for sharing this!
#### Remove the Code Signing Certificate
Matthew Taylor adds:
If you want to completely remove the code signing certificate, you can follow these steps:
- From a cmd prompt, run MMC, the Microsoft Management Console.
You may also type MMC after bringing up the Windows ‘Start’ menu.
- File > Add/Remove Snap-in... > Certificates > Add > My user account > Okay.
- Navigate to Certificates - Current User > Trusted Publishers > Certificates.
- You should now see a list of signatures.
- Proceed with extreme caution.
- To delete a certificate, just right-click > Delete.
- To close the console: File > Close > No.
Thank you, Matt!
#### Revit API Code Snippet Repository
Maycon Freitas, architect, Dynamo, Revit API Developer and Forge enthusiast at [Blossom Consult](https://www.blossomconsult.com),
shares a new collection of Revit API code snippets and invites the community to join in:
> I'm creating the [RevitAPISnippets GitHub repository](https://github.com/mayconrfreitas/RevitAPISnippets) to
share Revit API code snippets with our Revit developer community.
If you're interested in contributing somehow, I would really appreciate.
The idea is to create an open source project to help developers to improve coding performance.
More about this project in the two-and-a-half-minute video
on [RevitAPISnippets: 170+ code lines in 2 minutes (Revit API)](https://youtu.be/moD7CYUkJHw).
#### Batch Processing and Monitoring Progress
Questions on batch processing BIMs come up on a regular basis, and surfaced again discussing
a [way to check if family is corrupt](https://forums.autodesk.com/t5/revit-api-forum/way-to-check-if-family-is-corrupt/m-p/11174180) with
[Phillip Miller](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/311888) and
Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas:
\*\*Question:\*\* I have a simple addon that iterates over all the families in a document and exports them to a folder amongst other things, using `fam.SaveAs(path, options)`.
All works as expected until the very rare case when it hits a family that is corrupt.
This doesn't happen often, but it does happen.
When it hits such a family there are no warning messages, doesn't crash, just hangs.
My question: Is there a way that I can check for this situation or a way to skip on to the next family?
\*\*Answer:\*\* I don't think so.
You can't cancel the process either so also no possibility of setting a time limit. Only thing you could do is create a logging system where the last item that caused the issue along with those already exported up to that point can be skipped on the next run of the Add-in.
For these kind of issues, all you can really do from an API developer perspective is get end user to open a support case with Autodesk regarding the attached project file (or you could open one).
A brief review of the journal indicates an issue with many short curves.
Regarding end user support case they can easily replicate the issue outside of the API by attempting to edit the family in the UI.
It seems to be getting stuck in a regeneration loop and may eventually resolve.
I wouldn't classify it as document corruption because that tends to have an abrupt end, i.e., file would not open and error message would be shown in UI.
Seems to be an issue with how Revit responds to a certain circumstance that the file causes, i.e., problem with Revit.
\*\*Response:\*\* Thanks for your reply.
It was what I thought but just wanted to make sure I wasn't missing anything.
This is meant to be a totally automated process with no user interaction so is a bit of a shame I can't get around this catch and move on.
What I will have to do is set a timer and if there is a long period of time opening a family, pop up a message box of some sort to notify the user of the issue when they get back to their workstation.
\*\*Answer:\*\* Should also mention the `ProgressChangedEventArgs.Cancel` of `Application.ProgressChanged`.
This reports the progress of the little green bar that appears in the bottom left.
This has a cancel button next to it that is available when opening a family.
So, it may be possible to cancel via that if you spot a recurring pattern of progress as appears to happen with this family, e.g., how many times it goes to 25% after 50% etc.
ProgressChangedEventArgs.Caption may also report `Regenerating`, so you could also count those and set limits on that.
This all hinges on if ProgressChangedEventArgs.Cancel works, i.e., it works in the UI immediately for the family in question.
Below is an example that seems to work well for English Language Revit.
When you call cancel on the event it throws an exception for the calling method so you can catch that and move onto the next item. The below works by counting the occurrences of "Regenerating", a count of around 2200 is about right for that family (when it starts to loop).
You could as an alternative use the progress Position property but to get a percentage you need to divide the Position property by the UpperRange property. This approach doesn't seem as good as the regen caption approach because the percentages encountered are more random and not a good indicator of a pattern of repetition (would have to store more in an array to check against).

```
  Private IntRegenCount As Integer = 0

  Private Sub ProgressChanged(a As Object, e As Autodesk.Revit.DB.Events.ProgressChangedEventArgs)
    If e.Caption = "Regenerating" Then
      IntRegenCount += 1
    End If
    If IntRegenCount > 2200 Then
      'IntRegenCount = 0
      e.Cancel()
    End If
  End Sub

  Public Function Obj_220516d(commandData As ExternalCommandData, ByRef message As String, elements As ElementSet) As Result
    Dim IntUIApp As UIApplication = commandData.Application
    Dim IntUIDoc As UIDocument = commandData.Application.ActiveUIDocument
    Dim IntDoc As Document = IntUIDoc.Document

    AddHandler commandData.Application.Application.ProgressChanged, AddressOf ProgressChanged
    Const NM As String = "Vantage Metro Series Hinged Door Jamb"

    Dim FEC As New FilteredElementCollector(IntDoc)
    Dim ECF As New ElementClassFilter(GetType(Family))
    Dim F As Family = FEC.WherePasses(ECF) _
      .Where(Function(x) x.Name = NM) _
      .Cast(Of Family) _
      .FirstOrDefault

     Dim FDoc As Document = Nothing

    Try
      FDoc = IntDoc.EditFamily(F)
    Catch ex As Exception
      Return Result.Cancelled
    Finally
      RemoveHandler commandData.Application.Application.ProgressChanged, AddressOf ProgressChanged
    End Try

    Return Result.Succeeded

  End Function
```

This below non-language dependant example appears to work also.
In below I attempt to edit the problem family before moving on to a second family in the project file that does open.
Again, you have to look at the numbers for variables N and T below.
The limit of T below often occurs first and seems to give adequate time allowance to edit problem family.
For either case, you have to handle the DialogBoxShowing event to cancel the dialogue that appears after cancelling the progress changed event.

```
  Private IntProgressLog As Integer() = New Integer(20) {}

  Private Sub ProgressChanged(s As Object, e As Autodesk.Revit.DB.Events.ProgressChangedEventArgs)
    Dim ULim As Double = e.UpperRange
    Dim Pos As Double = e.Position
    Dim Perc As Integer = ((Pos / ULim) * 20)
    IntProgressLog(Perc) += 1

    Dim N As Integer = IntProgressLog.Max
    Dim T As Integer = IntProgressLog.Sum
    If N > 2000 OrElse T > 15000 Then
      e.Cancel()
    End If
  End Sub
  Private Sub MsgBoxShowing(s As Object, e As Autodesk.Revit.UI.Events.DialogBoxShowingEventArgs)
    Dim IntUIApp As UIApplication = s
    IntProgressLog = New Integer(20) {} 'reset the variable for next family
    e.OverrideResult(2)
  End Sub

  Public Function Obj_220516d(commandData As ExternalCommandData, ByRef message As String, elements As ElementSet) As Result
    Dim IntUIApp As UIApplication = commandData.Application
    Dim IntUIDoc As UIDocument = commandData.Application.ActiveUIDocument
    Dim IntDoc As Document = IntUIDoc.Document

    AddHandler IntUIApp.Application.ProgressChanged, AddressOf ProgressChanged
    AddHandler IntUIApp.DialogBoxShowing, AddressOf MsgBoxShowing

    Dim NM As String() = New String(1) {"Vantage Metro Series Hinged Door Jamb", "_ Callout Head"}

    Dim FDoc As Document = Nothing
    For i = 0 To 1
      Dim K As Integer = i
      Dim FEC As New FilteredElementCollector(IntDoc)
      Dim ECF As New ElementClassFilter(GetType(Family))
      Dim F As Family = FEC.WherePasses(ECF) _
        .Where(Function(x) x.Name = NM(K)) _
        .Cast(Of Family) _
        .FirstOrDefault

      Try
        FDoc = IntDoc.EditFamily(F)
      Catch ex As Exception
      End Try
      If FDoc IsNot Nothing Then
        Debug.WriteLine("Edit of: " & NM(i))
      End If
    Next

    RemoveHandler IntUIApp.Application.ProgressChanged, AddressOf ProgressChanged
    RemoveHandler IntUIApp.DialogBoxShowing, AddressOf MsgBoxShowing

    Return Result.Succeeded
  End Function
```

\*\*Answer 2:\*\* Following up on some of your initial thoughts, I would assume that this can be totally automated after all, like this:
- Run an external Windows app that monitors progress and can kill Revit.exe by terminating the process via native OS calls
- Loop through all the families and call SaveAs on each, logging progress to an external file (or wherever you like)
- The log file helps keep track of where you are and where you need to proceed next
- If the external file has not been updated for a while (a minute? ten minutes?), kill Revit.exe and restart with the next family to export, logging and skipping the corrupt one
Sound feasible? It is a pretty standard approach to run a batch process.
Revit is not designed for batch processing dozens of files, let alone hundreds or thousands, so crashes of all kinds are to be expected, perfectly normal, and need to be handled gracefully by batch processing workflows.
Check out [The Building Coder 'batch' category](https://thebuildingcoder.typepad.com/blog/batch).
\*\*Response:\*\* Thank you so much.
You have gone way above and beyond with providing examples etc, and I thank you for that.
I'm back working on this project next week and I will let you know how I get on.
Many thanks to Phillip for raising the issue and Richard for his indefatigable help.
![Programming progress](img/programming_progress_joke.png "Programming progress")