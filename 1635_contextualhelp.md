---
post_number: "1635"
title: "Contextualhelp"
slug: "contextualhelp"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'sheets', 'transactions', 'windows']
source_file: "1635_contextualhelp.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1635_contextualhelp.html"
---

### Contextual Help Best Practice
Dragos Turmac of the Revit development team solved
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) questions
on [contextual help not working from within a command](https://forums.autodesk.com/t5/revit-api-forum/contextual-help-not-working-from-within-a-command/m-p/7842882) and
on [F1 help for add-in only](https://forums.autodesk.com/t5/revit-api-forum/f1-help-for-addin-only/m-p/7522730) by
explaining the current best practice to implement online help:
\*\*Question:\*\* I'm trying to add contextual help for all commands (press F1 to go to a location).
It's working fine when the tooltip is shown for my commands, but, when the add-in form is open, pressing F1 opens the Autodesk knowledge site for Revit instead.
I looked at the documentation
on [Ribbon panels and controls](https://knowledge.autodesk.com/search-result/caas/CloudHelp/cloudhelp/2017/ENU/Revit-API/files/GUID-1547E521-59BD-4819-A989-F5A238B9F2B3-htm.html) and this part:
> The `ContextualHelp` class has a `Launch` method that can be called to display the help topic specified by the contents of this `ContextualHelp` object at any time, the same as when the F1 key is pressed when the control is active. This allows the association of help topics with user interface components inside dialogs created by an add-in application.
This leads me to believe that one should be able to press F1 from within a command and get the correct contextual help.
What am I missing?
\*\*Question 2:\*\* This seems to be a repeat of my original request
for [F1 help for add-in only](https://forums.autodesk.com/t5/revit-api-forum/f1-help-for-addin-only/m-p/7522730):
> I am adding Help to my add-in and want the F1 key to call my help file (CHM) for the whole form, not just for the control that has focus. I am using the following code which is working fine. My problem is that F1 triggers Revit's online help after calling mine. Is there a way to prevent this keystroke going on to Revit?
```csharp
protected override bool ProcessCmdKey(
ref Message msg,
Keys keyData )
{
// F1 for help on any control
if( keyData == Keys.F1 )
{
HelpClick();
// This keystroke was handled, don't
// pass to the control with the focus
return true;
}
return base.ProcessCmdKey( ref msg, keyData );
}
```
\*\*Answer:\*\* I used the contextual help functionality a couple of times in the past and never encountered the behaviour you describe:
- [Multi-Version Add-in](http://thebuildingcoder.typepad.com/blog/2012/07/multi-version-add-in.html)
- [What's New in the Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html) > Contextual help support
- [Using Balloon Tips in Revit](http://thebuildingcoder.typepad.com/blog/2014/03/using-balloon-tips-in-revit.html)
- [From Hack to App – OBJ Mesh Import to DirectShape](http://thebuildingcoder.typepad.com/blog/2015/02/from-hack-to-app-obj-mesh-import-to-directshape.htm...)
Unfortunately, all of these date rather far back, so I cannot guarantee that they behave as expected nowadays.
Happily, Dragos solved the issue by explaining the up-to-date solution and best practice implementing online help:
An F1 key hit in a modal C# form is normally caught using the `HelpRequested` control event, like this:
```csharp
// in designer
this.HelpRequested += new System.Windows.Forms
.HelpEventHandler( this.OnHelpRequested );
// in form's code
private void OnHelpRequested(
object sender,
HelpEventArgs hlpevent )
{
// Do stuff here, whatever you wish.
// Once this event is called, Revit no longer follows
Help.ShowPopup( this, "test help",
new Point( somewhere.x, somewhere.y ) );
}
```
If the above event is not subscribed to, Revit opens its own help.
If it is, Revit does nothing.
Attached a small add-in
in [contextual_help_test_addin.zip](zip//a/case/sfdc/12806669/contextual_help_test_addin.zip) that
only shows a Windows form on which an F1 press will show up a test popup help and Revit does nothing about it.
It implements a start-up command and a test dialogue.
Here is the complete code for the command:
```csharp
using Autodesk.Revit.UI;
using Autodesk.Revit.DB;
using Autodesk.Revit.Attributes;
namespace TestAddin
{
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
public class StartUpCmd : IExternalCommand
{
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
TestDlg testDlg = new TestDlg();
testDlg.ShowDialog();
return Result.Succeeded;
}
}
}
```
The C# portion of the test dialogue implementation looks like this:
```csharp
using System.Drawing;
using System.Windows.Forms;
namespace TestAddin
{
public partial class TestDlg : Form
{
public TestDlg()
{
InitializeComponent();
}
private void OnHelpRequested(
object sender,
HelpEventArgs hlpevent )
{
Help.ShowPopup( this, "test help",
new Point( Top, Left ) );
}
}
}
```
\*\*Response:\*\* I can confirm that I used your original snippet to call my context help routine, and all worked perfectly.
Many thanks to Dragos for this very clear explanation and succinct sample!
![Help](img/help.png)