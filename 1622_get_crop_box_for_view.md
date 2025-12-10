---
post_number: "1622"
title: "Get Crop Box For View"
slug: "get_crop_box_for_view"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'revit-api', 'sheets', 'transactions', 'vbnet', 'views']
source_file: "1622_get_crop_box_for_view.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1622_get_crop_box_for_view.html"
---

### Efficiently Retrieve Crop Box for Given View
Konrads Samulis shared a very nice solution to retrieve the crop box for a given view using a highly efficient parameter filter in
his [comment](http://thebuildingcoder.typepad.com/blog/2013/09/rotating-a-plan-view.html#comment-3734421721)
on [rotating a plan view](http://thebuildingcoder.typepad.com/blog/2013/09/rotating-a-plan-view.html).
In his own words:
In digging up this old thread, I found something quite curious in the API in 18.1, that I'm not sure was there before.
The method of using a temporary transaction (with rollback) to find the element id of the crop box was taking a very long time on a large model, so I did a bit of digging to see how I could improve it.
I noticed that in the built-in parameter `ID_PARAM` of the crop box contains the element id of the view it's in.
E.g., the crop box 'points' to the id of the view it is in using `ID_PARAM`.
That got me thinking... why not use a parameter filter to retrieve it?
So... in VB.NET (because I'm an old school vber):
```vbnet
Public Function GetELementIDOfCropBox(activeView As View, revDoc As Document) As ElementId
Dim provider As ParameterValueProvider = New ParameterValueProvider(New ElementId(CInt(BuiltInParameter.ID_PARAM)))
Dim ruleID_PARAM As FilterElementIdRule = New FilterElementIdRule(provider, New FilterNumericEquals(), activeView.Id)
Dim filter As ElementParameterFilter = New ElementParameterFilter(ruleID_PARAM)
Dim collector = New FilteredElementCollector(revDoc).WherePasses(filter).ToElementIds().Except(New List(Of ElementId)(New ElementId() {activeView.Id}))
Return collector.FirstOrDefault
End Function
```
This can all be stuffed into one single statement:
```csharp
Public Function GetELementIDOfCropBoxShortened(activeView As View, revDoc As Document) As ElementId
Return New FilteredElementCollector(revDoc)
.WherePasses(New ElementParameterFilter(
New FilterElementIdRule(
New ParameterValueProvider(
New ElementId(CInt(BuiltInParameter.ID_PARAM))),
New FilterNumericEquals(),
activeView.Id)))
.ToElementIds()
.Except(New List(Of ElementId)(New ElementId() {activeView.Id})).FirstOrDefault
End Function
```
Much like your original, I had to exclude the element id of the view, as it also returns its own id from `ID_PARAM`.
Hope that this may help someone – I'm sure the Dynamo people who come here for inspiration will want to know this as well...
Many thanks to Konrads for sharing this nice efficient solution!
I ported it to C# and added it
to [The Building Coder Samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2018.0.135.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.135.2) and
the extensive collection of filtered element examples in
the [module CmdCollectorPerformance.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs):
```csharp
///
/// Return element id of crop box for a given view.
/// The built-in parameter ID_PARAM of the crop box
/// contains the element id of the view it is used in;
/// e.g., the crop box 'points' to the view using it
/// via ID_PARAM. Therefore, we can use a parameter
/// filter to retrieve all crop boxes with the
/// view's element id in that parameter.
/// summary>
ElementId GetCropBoxFor( View view )
{
ParameterValueProvider provider
= new ParameterValueProvider( new ElementId(
(int) BuiltInParameter.ID_PARAM ) );
FilterElementIdRule rule
= new FilterElementIdRule( provider,
new FilterNumericEquals(), view.Id );
ElementParameterFilter filter
= new ElementParameterFilter( rule );
return new FilteredElementCollector( view.Document )
.WherePasses( filter )
.ToElementIds()
.Where( a => a.IntegerValue
!= view.Id.IntegerValue )
.FirstOrDefault();
}
```
![3D view section box](img/3d_view_section_box.png)