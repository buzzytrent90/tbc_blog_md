---
post_number: "1678"
title: "Purge Unused"
slug: "purge_unused"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'rooms', 'sheets', 'transactions', 'vbnet', 'views']
source_file: "1678_purge_unused.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1678_purge_unused.html"
---

### Purge Unused using Performance Adviser
We repeatedly looked at ways to detect and purge unused elements.
A list of some previous discussions of the topic was given last time we looked
at [purge and detecting an empty view](http://thebuildingcoder.typepad.com/blog/2017/11/purge-and-detecting-an-empty-view.html).
Matt Taylor, associate and CAD developer at [WSP](https://www.wsp.com),
was [the first to congratulate](http://thebuildingcoder.typepad.com/blog/2018/08/ten-years-anniversary-and-revit-api-with-mvvm-wpf-and-winform.html#comment-4053631853)
on [The Building Coder's ten-year anniversary](http://thebuildingcoder.typepad.com/blog/2018/08/ten-years-anniversary-and-revit-api-with-mvvm-wpf-and-winform.html).
He now adds something really special to celebrate this:
- [Purge Unused using the Performance Adviser](#2)
- [PurgeTool.vb implements `GetPurgeableElements`](#3)
- [PurgeUnused.vb External Command](#4)
![Broomstick](img/broomstick.png)
#### Purge Unused using the Performance Adviser
I’m sharing with you a new discovery of mine.
Apparently, nobody has previously publicly discovered a simple and effective way of purging all unused elements.
I now found one:
I have successfully used
the [Performance Adviser](http://help.autodesk.com/view/RVT/2019/ENU/?guid=Revit_API_Revit_API_Developers_Guide_Advanced_Topics_Performance_Adviser_html) to
do a similar job to the native `Purge Unused` command.
Please refer to
my [RevitPurgeUnused GitLab repository](https://gitlab.com/MattTaylor/RevitPurgeUnused).
While the code will compile back to Revit 2012, it actually throws an `InternalException` for versions 2012-2016 (in my experience).
It doesn’t do a perfect job (e.g., it doesn’t purge materials and material assets), but it is very, very good, and quite fast.
I also added a note of my solution to some of the existing threads on this topic in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160):
- [Purge Unused Via the API](https://forums.autodesk.com/t5/revit-api-forum/purge-unused-via-the-api/m-p/8229573)
- [CF-1201 \*Purge unused objects\*](https://forums.autodesk.com/t5/revit-api-forum/cf-1201-purge-unused-objects/m-p/8229574)
Very many thanks to Matt for sharing this solution to one of the top developer wish list items!
#### PurgeTool.vb implements GetPurgeableElements
Here is Matt's VB.NET code
in [PurgeTool.vb](https://gitlab.com/MattTaylor/RevitPurgeUnused/blob/master/PurgeTool.vb),
defining the long-sought-after `GetPurgeableElements` method:

```
# Region "Imported Namespaces"
Imports System
Imports System.Collections.Generic
Imports Autodesk.Revit.DB
# End Region

Public Class PurgeTool
  '''
  ''' The guid of the 'Project contains unused families and types' PerformanceAdviserRuleId.
  '''
  Const PurgeGuid As String = "e8c63650-70b7-435a-9010-ec97660c1bda"

  '''
  ''' Get all purgeable elements.
  ''' Intended for Revit 2017+ as versions up to and including Revit 2016 throw an InternalException.
  '''
  '''
  '''
  ''' True if successful.
  Shared Function GetPurgeableElements(doc As Document, ByRef purgeableElementIds As ICollection(Of ElementId)) As Boolean
    purgeableElementIds = New List(Of ElementId)()

    Try
      'create a new list of rules.
      Dim ruleIds As IList(Of PerformanceAdviserRuleId) = New List(Of PerformanceAdviserRuleId)
      Dim ruleId As PerformanceAdviserRuleId = Nothing
      'find the intended rule.
      If GetPerformanceAdvisorRuleId(PurgeGuid, ruleId) Then
        'add the rule to the new list.
        ruleIds.Add(ruleId)
      Else
        'cannot find rule.
        Return False
      End If
      'execute our chosen rule only.
      Dim failureMessages As IList(Of FailureMessage) = PerformanceAdviser.GetPerformanceAdviser().ExecuteRules(doc, ruleIds)
      If failureMessages.Count > 0 Then
        'If there are any purgeable elements, we should have a failure message.
        'the failure message should have a collection of failing elements - set to our byref collection
        purgeableElementIds = failureMessages.Item(0).GetFailingElements
      End If
      'no errors - return true.
      Return True
    Catch ex As Autodesk.Revit.Exceptions.InternalException
      'this exception gets thrown a lot in earlier versions of Revit - up to and including Revit 2016.

    End Try
    'likely thrown an internal exception
    Return False
  End Function

  '''
  ''' Find a PerformanceAdviserRuleId with a guid that matches a supplied guid.
  '''
  '''
  '''
  ''' true if successful, along with the rule as a byref.
  Private Shared Function GetPerformanceAdvisorRuleId(ByVal guidStr As String, ByRef ruleId As PerformanceAdviserRuleId) As Boolean
    ruleId = Nothing
    Dim guid As Guid = New Guid(guidStr)
    For Each rule As PerformanceAdviserRuleId In PerformanceAdviser.GetPerformanceAdviser().GetAllRuleIds
      'check if the rule Id matches our rule guid
      If rule.Guid.Equals(guid) Then
        'it does - set rule to our byref object
        ruleId = rule
        Return True
      End If
    Next
    'failed to find the rule matching our guid.
    Return Nothing
  End Function
End Class
```

#### PurgeUnused.vb External Command
The result is used like this in the external command defined
by [PurgeUnused.vb](https://gitlab.com/MattTaylor/RevitPurgeUnused/blob/master/PurgeUnused.vb):

```
  Dim purgeableElements As ICollection(Of ElementId) = Nothing
  If PurgeTool.GetPurgeableElements(doc, purgeableElements) AndAlso purgeableElements.Count > 0 Then
    Using transaction As New Transaction(doc, "Purge Unused")
      transaction.Start()
      doc.Delete(purgeableElements)
      transaction.Commit()
      Return Result.Succeeded
    End Using
  Else
    Return Result.Failed
  End If
```