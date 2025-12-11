---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: news
optimization_date: '2025-12-11T11:44:14.238004'
original_url: https://thebuildingcoder.typepad.com/blog/0599_vb_line_continuation.html
post_number: 0599
reading_time_minutes: 2
series: general
slug: vb_line_continuation
source_file: 0599_vb_line_continuation.htm
tags:
- csharp
- elements
- revit-api
- transactions
- vbnet
- views
title: Implicit Line Continuation in VB 2010
word_count: 466
---

### Implicit Line Continuation in VB 2010

An new feature that is of interesting to us VB Revit add-in developers is the
[implicit line continuation](http://msdn.microsoft.com/en-us/library/865x40k4.aspx) that
was added in Visual Basic 2010.

In prior versions of VB, all statements that continue beyond the end of the line have to be explicitly marked using an underscore '\_' character.
Since some of the Revit API method signatures force rather long lines, this can become a bit overwhelming.
This is what it might look like in VB 2008:
```vbnet
''' <summary>
''' Hello World #2 - simplified without full namespace.
''' </summary>
<Transaction(TransactionMode.Automatic)> \_
Public Class HelloWorldSimple
  Implements IExternalCommand

  Public Function Execute( \_
    ByVal commandData As ExternalCommandData, \_
    ByRef message As String, \_
    ByVal elements As ElementSet) \_
    As Result \_
    Implements IExternalCommand.Execute

    TaskDialog.Show("My Dialog Title", "Hello World Simple!")
    Return Result.Succeeded

  End Function

End Class
```

With the new implicit line continuation feature, many of the underscores can be omitted in VB 2010:
```vbnet
<Transaction(TransactionMode.Automatic)>
Public Class HelloWorldSimple
  Implements IExternalCommand

  Public Function Execute(
    ByVal commandData As ExternalCommandData,
    ByRef message As String,
    ByVal elements As ElementSet) \_
    As Result \_
    Implements IExternalCommand.Execute

    TaskDialog.Show("My Dialog Title", "Hello World Simple!")
    Return Result.Succeeded

  End Function

End Class
```

Note that two of the underscores still remain;
the implicit line continuation is only available after certain syntax elements, such as after a comma or an opening parenthesis, or before a closing parenthesis.

Here is an interview with Doug Rothaus of the Visual Studio User Education team describing it:

#### Updated Visual Basic Add-in Wizard

Here is an updated VB version of the Visual Studio
[Revit 2012 add-in wizard](http://thebuildingcoder.typepad.com/blog/2011/04/visual-studio-add-in-wizards-for-revit-2012.html) making
use of the implicit line continuation:

- [TemplateRevitArch2012AddinVbJt\_3.zip](zip/TemplateRevitArch2012AddinVbJt_3.zip) – copy to

  [My Documents]\Visual Studio 2010\Templates\ProjectTemplates\Visual Basic

The only change I made was to remove the line continuation underscores wherever possible.

For the C# version, please refer to the
[original post](http://thebuildingcoder.typepad.com/blog/2011/04/visual-studio-add-in-wizards-for-revit-2012.html).

#### Visual Studio Line Length Guide Line

Another complementary Visual Studio feature of special interest to all of us who have to limit the line length of our source code due to one reason or another is the
[guide-line for Visual Studio](http://through-the-interface.typepad.com/through_the_interface/2011/06/a-guide-line-for-visual-studio.html) pointed
out yesterday by Augusto and Kean.

Since I have the same issue as Kean, having to limit my source code line length strictly to fit into the blog post format, I installed it as described and verified that it works well.