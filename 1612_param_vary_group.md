---
post_number: "1612"
title: "Param Vary Group"
slug: "param_vary_group"
author: "Jeremy Tammik"
tags: ['elements', 'parameters', 'revit-api', 'sheets', 'transactions', 'views', 'windows']
source_file: "1612_param_vary_group.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1612_param_vary_group.html"
---

### Setting Parameter Varies Between Groups
We already looked at the topic of setting the `SetAllowVaryBetweenGroups` flag on a shared parameter
in Scott Conover's [parameter definition overview](http://thebuildingcoder.typepad.com/blog/2016/12/parameter-definition-overview.html).
The setting was introduced in the [Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html),
cf. \*Parameter variance among group instances\*.
Now Miroslav Schonauer raised it again, asking:
\*\*Question:\*\* Is the following option for group behaviour of shared param \*instance\* bindings exposed to API?
![SetAllowVaryBetweenGroups](img/vary_between_groups_1.png)
If yes, that solves all the issues :-)
If not – the default outcome seems to be \*aligned per group type\*.
Is there any way to set \*can vary by group instance\* (what I need) the default for API-created bindings?
Later: I found
this [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [creating a project parameter with \*values can vary by group instance\* selected](https://forums.autodesk.com/t5/revit-api-forum/create-project-parameter-with-quot-values-can-vary-by-group/m-p/5939455)
which explains that it is kind-of possible.
The problem remains that `SetAllowVaryBetweenGroups` is available only on `InternalDefinition`, while my programmatically created shared param has `ExternalDefinition`.
That thread explains that getting the binding after it has been created (i.e., in a 'Step 2') does return `InternalDefinition`, so this method can be used.
Can someone at least confirm that there is nothing simpler to do than the above 2-step process?
\*\*Answer:\*\* Yes. You need to do the two-step process:
- Register the shared parameter
- Find the internal definition
- Set the appropriate value for this property
The easiest way to go from one to the other:
- [SharedParameterElement.Lookup(GUID)](http://www.revitapidocs.com/2018.1/4dce82de-7495-523a-c8d4-4b3fc709e85e.htm)
- [ParameterElement.GetDefinition()](http://www.revitapidocs.com/2018.1/ec9b3cd3-4379-6eb8-7c7d-c220ba03f359.htm)
\*\*Response:\*\* I implemented this method to handle the setting of the `SetAllowVaryBetweenGroups` flag:
```csharp
///
/// Helper method to control `SetAllowVaryBetweenGroups`
/// option for instance binding param
/// summary>
static void SetInstanceParamVaryBetweenGroupsBehaviour(
Document doc,
Guid guid,
bool allowVaryBetweenGroups = true )
{
try // last resort
{
SharedParameterElement sp
= SharedParameterElement.Lookup( doc, guid );
// Should never happen as we will call
// this only for \*existing\* shared param.
if( null == sp ) return;
InternalDefinition def = sp.GetDefinition();
if( def.VariesAcrossGroups != allowVaryBetweenGroups )
{
// Must be within an outer transaction!
def.SetAllowVaryBetweenGroups( doc, allowVaryBetweenGroups );
}
}
catch { } // ideally, should report something to log...
}
```
It assumes that `guid` comes from a known shared parameter.
Further good news: this can be called not only immediately after programmatically binding a new shared param, but also to \*silently\* change this specific setting for an \*existing\* shared parameter.
For example, we all typically have our own helper methods to get-or-create a shared parameter binding, cf., e.g., my method
to [add a category to a shared parameter binding](http://thebuildingcoder.typepad.com/blog/2012/04/adding-a-category-to-a-shared-parameter-binding.html).
Here is a code snippet providing enough to get the gist of how the above can be used (ignore my helper classes and error handling):
```csharp
// Assumes outer transaction
public static Parameter GetOrCreateElemSharedParam(
Element elem,
string paramName,
string grpName,
ParameterType paramType,
bool visible,
bool instanceBinding,
bool userModifiable,
Guid guid,
bool useTempSharedParamFile,
string tooltip = "",
BuiltInParameterGroup uiGrp = BuiltInParameterGroup.INVALID,
bool allowVaryBetweenGroups = true )
{
try
{
// Check if existing
Parameter param = elem.LookupParameter( paramName );
if( null != param )
{
// NOTE: If you don't want forcefully setting
// the "old" instance params to
// allowVaryBetweenGroups =true,
// just comment the next 3 lines.
if( instanceBinding && allowVaryBetweenGroups )
{
SetInstanceParamVaryBetweenGroupsBehaviour(
elem.Document, guid, allowVaryBetweenGroups );
}
return param;
}
// If here, need to create it (my custom
// implementation and classes…)
BindSharedParamResult res = BindSharedParam(
elem.Document, elem.Category, paramName, grpName,
paramType, visible, instanceBinding, userModifiable,
guid, useTempSharedParamFile, tooltip, uiGrp );
if( res != BindSharedParamResult.eSuccessfullyBound
&& res != BindSharedParamResult.eAlreadyBound )
{
return null;
}
// Set AllowVaryBetweenGroups for NEW Instance
// Binding Shared Param
if( instanceBinding )
{
SetInstanceParamVaryBetweenGroupsBehaviour(
elem.Document, guid, allowVaryBetweenGroups );
}
// If here, binding is OK and param seems to be
// IMMEDIATELY available from the very same command
return elem.LookupParameter( paramName );
}
catch( Exception ex )
{
System.Windows.Forms.MessageBox.Show(
string.Format(
"Error in getting or creating Element Param: {0}",
ex.Message ) );
return null;
}
}
```
I added Miro's method `SetInstanceParamVaryBetweenGroupsBehaviour`
to [The Building Coder samples release 2018.0.134.11](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.134.11) in
the module [CmdCreateSharedParams.cs L441-L470](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCreateSharedParams.cs#L441-L470).
Many thanks to Miro for raising this issue and sharing his approach to solve it!