---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.3
content_type: qa
optimization_date: '2025-12-11T11:44:16.406450'
original_url: https://thebuildingcoder.typepad.com/blog/1621_return_failure_info.html
post_number: '1621'
reading_time_minutes: 9
series: general
slug: return_failure_info
source_file: 1621_return_failure_info.md
tags:
- csharp
- elements
- filtering
- levels
- parameters
- revit-api
- sheets
- transactions
- views
- walls
title: Return Failure Info
word_count: 1750
---

### Gathering and Returning Failure Information
Here is a nice solution shared
by [Mastjaso](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1058186) based on significant help
from [Rpthomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [returning failure information to command](https://forums.autodesk.com/t5/revit-api-forum/return-failure-information-to-command/m-p/7695676):
\*\*Question:\*\* Is there a way to return failure information to your main command when creating a transaction?
My program is doing some error logging and creating an error report at the end of it, and I'd like to include some information from specific types of Revit failures (i.e., include if multiple instances were created in the same place).
I looked over the failure SDK samples, forum and blog posts but have not yet come across, nor can think of a way to accomplish this. Implementing a preprocessor and passing it to the transaction is simple enough, but how do I get that information back to my main program? Since I'm not passing the preprocessor in using `ref`, I can't save it to a property on the preprocessor for accessing after and the transaction object itself doesn't appear to contain any information about failures that occur.
Do I have to write to a file on disk with my preprocessor and then read that back with the tail end of my main program?
As far as I can tell, the general stage for this process is as follow:
- Addin creates a transaction.
- Addin calls `Transaction.Commit` to end the transaction.
- The failure preprocessor is executed.
- Failure processing event is flagged, and all failure processing event handlers are run.
- Failure processor is run.
- The transaction is either committed or rolled back by Revit.
- The addin will now execute the line of code that comes after `Transaction.Commit`.
My issue is that I want Revit to continue with its standard failure processing, and I just want the information about those failures. From the perspective of my add-in, I can call a transaction and pass it a custom preprocessor class; however, this doesn't appear to be a real object, just a method that gets run with a set return value. I could also hook up a preprocessor event handler, but again, there's no simple way to get information from my event handler, back into my main program.
Here's the preprocessor example from the ErrorHandling SDK sample:
```csharp
Transaction transaction = new Transaction(
m_doc, Warning_FailurePreproccessor_OverlappedWall );
FailureHandlingOptions options = transaction
.GetFailureHandlingOptions();
FailurePreproccessor preproccessor
= new FailurePreproccessor();
options.SetFailuresPreprocessor( preproccessor );
transaction.SetFailureHandlingOptions( options );
transaction.Start();
Line line = Line.CreateBound( new XYZ( -10, 0, 0 ),
new XYZ( -20, 0, 0 ) );
Wall wall1 = Wall.Create( m_doc, line, level1.Id, false );
Wall wall2 = Wall.Create( m_doc, line, level1.Id, false );
m_doc.Regenerate();
transaction.Commit();
```
So, I can pass in a preprocessor object to handle the preprocessing where I can access the failure errors.
However, the preprocessor object just looks like this:
```csharp
///
/// Implements the interface IFailuresPreprocessor
/// summary>
public class FailurePreproccessor : IFailuresPreprocessor
{
///
/// This method is called when there have been
/// failures found at the end of a transaction
/// and Revit is about to start processing them.
/// summary>
/// The Interface class
/// that provides access to the failure information. param>
/// returns>
public FailureProcessingResult PreprocessFailures(
FailuresAccessor failuresAccessor )
{
}
}
```
With just a single method (`PreprocessFailures`) that presumably gets called by Revit, with a set return value to Revit. So, my issue is here, how do I get information from this failure preprocessing method, back into my main program? The preprocessor's return value gets returned to Revit, not my addin, and if I want to implement the `IFailuresPreprocessor` interface I can't pass in a `ref` or `out` variable to the preprocessor to write to, as far as I can tell.
\*\*Answer:\*\* The implementation of the `PreprocessFailures` method has an argument `failuresAccessor` which is used for reviewing and dealing with failure messages.
The below is an example implementation. Failure messages have to be dealt with before you can commit a transaction either by resolving them or deleting them so there are generally no messages left after transaction is committed. You can get information out, but you would need to create your own class to store the information. `IFailurePreprocessor` is more for resolving the messages beforehand.
If you were passing API objects `ByRef` or holding onto them you'd likely get errors due to those objects being out of context.
```vbnet
Private Class FailurePP
Implements IFailuresPreprocessor
Private FailureList As New List(Of String)
Public Function PreprocessFailures(failuresAccessor As FailuresAccessor) As FailureProcessingResult Implements IFailuresPreprocessor.PreprocessFailures
For Each item As FailureMessageAccessor In failuresAccessor.GetFailureMessages
FailureList.Add(item.GetDescriptionText)
Dim FailDefID As FailureDefinitionId = item.GetFailureDefinitionId
If FailDefID = BuiltInFailures.GeneralFailures.DuplicateValue Then
failuresAccessor.DeleteWarning(item)
End If
Next
Return FailureProcessingResult.ProceedWithCommit
End Function
Public Sub ShowDialogue()
Dim SB As New Text.StringBuilder
For Each item As String In FailureList
SB.AppendLine(item)
Next
TaskDialog.Show("Some info", SB.ToString)
End Sub
End Class
Private Function FailureEx(ByVal commandData As Autodesk.Revit.UI.ExternalCommandData,
ByRef message As String, ByVal elements As Autodesk.Revit.DB.ElementSet)
If commandData.Application.ActiveUIDocument Is Nothing Then Return Result.Succeeded Else
Dim IntUIDoc As UIDocument = commandData.Application.ActiveUIDocument
Dim IntDoc As Document = IntUIDoc.Document
Dim F_PP As New FailurePP
Using tx As New Transaction(IntDoc, "Marks")
Dim Ops As FailureHandlingOptions = tx.GetFailureHandlingOptions
Ops.SetFailuresPreprocessor(F_PP)
tx.SetFailureHandlingOptions(Ops)
If tx.Start = TransactionStatus.Started Then
Dim FEC As New FilteredElementCollector(IntDoc)
Dim ECF As New ElementCategoryFilter(BuiltInCategory.OST_StructuralColumns)
Dim Jx As List(Of Element) = FEC.WherePasses(ECF).WhereElementIsNotElementType.ToElements
If Jx.Count >= 2 Then
For I = 0 To 1
Jx(I).Parameter(BuiltInParameter.ALL_MODEL_MARK).Set("Duplicate Mark")
Next
End If
tx.Commit()
End If
End Using
F_PP.ShowDialogue()
Return Result.Succeeded
End Function
```
\*\*Response:\*\* The preprocessor doesn't necessarily need to resolve or delete them, though, since if it leaves them they'll just be handled by the failure processor, which I believe by default is those Revit popup boxes. I was looking at `failuresAccessor` to get the failure messages, but how do I get that information back into my program?
You mention that I would need to create my own class to store the information, but I'm confused as to how the method or object I pass would ever be able to get information back out into my main command.
\*\*Answer:\*\* They'll continue to a further stage of the failure framework but the point of the preprocessor is to deal with them so they don't arrive at that stage.
In the code above, I got the failure message out as a string, I could have also obtained the failure ID, the name of the transaction amongst other things. I'm not sure what information you are after?
The code below doesn't require you to cancel or delete warnings, but this depends on severity of warnings. The old Revit 2012 document states that returning `FailureProcessingResult.Continue` rolls back the transaction, but I find for the example below that not to be the case (since parameters have been changed). Probably this depends on failure severity. The warnings of duplicate mark still exist in the document, but the API objects are no more.
The API objects are not directly accessible after transaction commit, so you have to mirror the information within them in your own class.
The other two parts of the Failure API framework are probably not suitable for what you describe since they have no direct relation to your `IExternalcommand`, i.e., the `IFailuresProcessingEvent` is for everything called for all transactions, and the `IFailuresProcessor` is to replace the dialogue for the whole Revit session.
```vbnet
Private Class FailurePP
Implements IFailuresPreprocessor
Private FailureList As New List(Of String)
Public Function PreprocessFailures(failuresAccessor As FailuresAccessor) As FailureProcessingResult Implements IFailuresPreprocessor.PreprocessFailures
For Each item As FailureMessageAccessor In failuresAccessor.GetFailureMessages
FailureList.Add(item.GetDescriptionText)
Dim FailDefID As FailureDefinitionId = item.GetFailureDefinitionId
If FailDefID = BuiltInFailures.GeneralFailures.DuplicateValue Then
'failuresAccessor.DeleteWarning(item)
End If
Next
Return FailureProcessingResult.Continue
End Function
Public Sub ShowDialogue()
Dim SB As New Text.StringBuilder
For Each item As String In FailureList
SB.AppendLine(item)
Next
TaskDialog.Show("Some info", SB.ToString)
End Sub
End Class
```
\*\*Response:\*\* Thanks for your replies! The code does indeed work perfectly. I had a fundamental misunderstanding of how objects behaved in C# and had missed that you were calling `ShowDialogue` from outside the `PreprocessFailures` method.
In case anyone happens to come across this, I ported the code to C# and figure I may as well share it:
```csharp
///
/// Collect all failure message description strings.
/// summary>
class MessageDescriptionGatheringPreprocessor : IFailuresPreprocessor
{
List FailureList { get; set; }
public MessageDescriptionGatheringPreprocessor()
{
FailureList = new List();
}
public FailureProcessingResult PreprocessFailures(
FailuresAccessor failuresAccessor )
{
foreach( FailureMessageAccessor fMA
in failuresAccessor.GetFailureMessages() )
{
FailureList.Add( fMA.GetDescriptionText() );
FailureDefinitionId FailDefID
= fMA.GetFailureDefinitionId();
//if (FailDefID == BuiltInFailures
// .GeneralFailures.DuplicateValue)
// failuresAccessor.DeleteWarning(fMA);
}
return FailureProcessingResult.Continue;
}
public void ShowDialogue()
{
string s = string.Join( "\r\n", FailureList );
TaskDialog.Show( "Post Processing Failures:", s );
}
}
[Transaction( TransactionMode.Manual )]
class CmdFailureGatherer
{
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiApp = commandData.Application;
Document activeDoc = uiApp.ActiveUIDocument.Document;
MessageDescriptionGatheringPreprocessor pp
= new MessageDescriptionGatheringPreprocessor();
using( Transaction t = new Transaction( activeDoc ) )
{
FailureHandlingOptions ops
= t.GetFailureHandlingOptions();
ops.SetFailuresPreprocessor( pp );
t.SetFailureHandlingOptions( ops );
t.Start( "Marks" );
IList specEqu
= new FilteredElementCollector( activeDoc )
.OfCategory( BuiltInCategory.OST_SpecialityEquipment )
.WhereElementIsNotElementType()
.ToElements();
if( specEqu.Count >= 2 )
{
for( int i = 0; i < 2; i++ )
specEqu[i].get_Parameter(
BuiltInParameter.ALL_MODEL_MARK ).Set(
"Duplicate Mark" );
}
t.Commit();
}
pp.ShowDialogue();
return Result.Succeeded;
}
}
```
Many thanks to Mastjaso and Rpthomas108 for putting this together!
I added it
to [The Building Coder Samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2018.0.135.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.135.1)
in the new module
[CmdFailureGatherer.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdFailureGatherer.cs).
In that command, I generate a warning by creating two overlapping walls:
```csharp
// Generate an 'duplicate wall' warning message:
Element level = new FilteredElementCollector( doc )
.OfClass( typeof( Level ) )
.FirstElement();
Line line = Line.CreateBound( XYZ.Zero, 10 \* XYZ.BasisX );
Wall wall1 = Wall.Create( doc, line, level.Id, false );
Wall wall2 = Wall.Create( doc, line, level.Id, false );
```
During the command execution, this displays the standard Revit warning message, which is not suppressed:
![Standard Revit failure message](img/failure_message_dlg.png)
At the end of the command, the failure message descriptions have all been collected and are reported in a separate task dialogue:
![Failure message report](img/failure_message_report.png)