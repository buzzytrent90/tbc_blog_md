---
post_number: "1629"
title: "Change Text Colour"
slug: "change_text_colour"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'parameters', 'revit-api', 'selection', 'sheets', 'transactions', 'views', 'windows']
source_file: "1629_change_text_colour.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1629_change_text_colour.html"
---

### Changing Text Colour
Lately, I have been spending more time participating in the interesting discussions in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) than writing blog posts.
Occasionally, I grab some new code snippet worth preserving for posterity and add it
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).
Even more occasionally, I actually test it :-)
I did so now for the trivial task of changing text colour.
Here are some notes on that and a nice `node.js` web scraping tutorial:
- [Changing text colour via the text note type](#2)
- [Web scraping using `node.js`](#3)
#### Changing Text Colour via the Text Note Type
This topic came up when Rudi [@Revitalizer](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1103138) Honke
added a comment on colours in the thread
on [changing colour of labels within tags family per tag instance](https://forums.autodesk.com/t5/revit-api-forum/changing-color-of-labels-within-tags-family-per-tag-instance/m-p/7794532), saying,
> colour parameters are of type integer, but the raw value may be difficult to read for the user.
He explained how to read them in the earlier thread
on [how to change text colour](https://forums.autodesk.com/t5/revit-api-forum/how-to-change-text-color/td-p/2567672):
\*\*Question:\*\* I am using the `ColorDialog` control from Visual Studio 2008 to select a colour, and then I retrieve the RGB components in three variables:
- ColorComponentRed
- ColorComponentGreen
- ColorComponentBlue
I want to assign this colour to some text, but the following code is not working:
```vbnet
Dim colorparam As Parameter
colorparam = elem.ObjectType.Parameter(
BuiltInParameter.TEXT_COLOR)
Dim app As New Autodesk.Revit.Creation.Application()
Dim color As Autodesk.Revit.Color = app.NewColor()
color.Red = ColorComponentRed
color.Green = ColorComponentGreen
color.Blue = ColorComponentBlue
colorparam.Set(color)
```
Can anybody tell me what I am doing wrong and how to do it properly, please?
\*\*Answer:\*\* Hi.
```csharp
private int GetRevitTextColorFromSystemColor(
System.Drawing.Color color )
{
return (int) color.R \* (int) Math.Pow( 2, 0 )
+ (int) color.G \* (int) Math.Pow( 2, 8 )
+ (int) color.B \* (int) Math.Pow( 2, 16 );
}
```
I know that 2^0 is just 1, so you can simplify that to:
```csharp
private int GetRevitTextColorFromSystemColor(
System.Drawing.Color color )
{
return (int) color.R
+ (int) color.G \* (int) Math.Pow( 2, 8 )
+ (int) color.B \* (int) Math.Pow( 2, 16 );
}
```
Then, for example:
```csharp
int color = GetRevitTextColorFromSystemColor(
yourSystemColor );
textNoteType.get_Parameter(
BuiltInParameter.LINE_COLOR )
.Set( color );
```
Also note that starting with Revit 2017, the Revit API provides a `ColorSelectionDialog`.
You can use its `SelectedColor` property to get a Revit colour instead of a system colour.
Richard [@RPTHOMAS108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas added some 'quite convenient extension methods' to that:
```vbnet
Public Function AsRGB(ByVal Parameter As Parameter) As Byte()
Dim I As Integer = Parameter.AsInteger
Dim Red As Byte = I Mod 256
I = I \ 256
Dim Green As Byte = I Mod 256
I = I \ 256
Dim Blue As Byte = I Mod 256
Return New Byte(2) {Red, Green, Blue}
End Function
Public Function AsParameterValue(ByVal Color As Color) As Integer
Return Color.Red + (256 \* Color.Green) + (65536 \* Color.Blue)
End Function
Public Function AsParameterValue(ByVal Color As Windows.Media.Color) As Integer
Return Color.R + (256 \* Color.G) + (65536 \* Color.B)
End Function
Public Function AsParameterValue(ByVal Color As System.Drawing.Color) As Integer
Return Color.R + (256 \* Color.G) + (65536 \* Color.B)
End Function
```
Many thanks to Rudi and Richard for the helpful solutions!
I added Rudi's utility function
to [The Building Coder samples Util.cs module](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs#L455-L488) like
this:
```csharp
#region Colour Conversion
///
/// Revit text colour parameter value stored as an integer
/// in text note type BuiltInParameter.LINE_COLOR.
/// summary>
public static int ToColorParameterValue(
int red,
int green,
int blue )
{
return red + (green << 8) + (blue << 16);
}
///
/// Revit text colour parameter value stored as an integer
/// in text note type BuiltInParameter.LINE_COLOR.
/// summary>
public static int GetRevitTextColorFromSystemColor(
System.Drawing.Color color )
{
return ToColorParameterValue( color.R, color.G, color.B );
}
#endregion // Colour Conversion
```
To test it, I added an additional transaction step
to [CmdNewTextNote](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdNewTextNote.cs#L196-L216):
```csharp
using( Transaction t = new Transaction( doc ) )
{
t.Start( "Change Text Colour" );
int color = Util.ToColorParameterValue(
255, 0, 0 );
Element textNoteType = doc.GetElement(
txNote.GetTypeId() );
Parameter param = textNoteType.get_Parameter(
BuiltInParameter.LINE_COLOR );
// Note that this modifies the existing text
// note type for all instances using it. If
// not desired, use Duplicate() first.
param.Set( color );
t.Commit();
}
```
As the comment says, this modifies the text note type and consequently all text note instances referring to that type.
![Text note colour](img/text_note_color.png)
#### Web Scraping Using Node.js
On a different tack, if you are interested in `node.js` and web scraping, here is a very pleasant and super clear 27-minute step by step introduction to basic [web scraping with `node.js`](https://www.youtube.com/watch?v=eUYMiztBEdY):

It implements a minimal app live using the `request-promise` and `cheerio` libraries to gather and correlate data from two web sites.