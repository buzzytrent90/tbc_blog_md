---
post_number: "1482"
title: "The Building Coder"
slug: "material_asset_texture"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'levels', 'python', 'references', 'revit-api', 'sheets', 'views']
source_file: "1482_material_asset_texture.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1482_material_asset_texture.html"
---

### Material Asset Textures, Forge Webinar Recordings
Today is the last day before my one-week vacation, directly followed non-stop by
the [ISEPBIM](https://www.facebook.com/ISEPBIM) Forge and BIM workshops at [ISEP](http://www.isep.ipp.pt), Porto University,
the [RTC Revit Technology Conference Europe](http://www.rtcevents.com/rtc2016eur) in Porto, and
the [Forge Accelerator](http://autodeskcloudaccelerator.com) in Munich.
So come the end of the month I will probably be in dire need of another vacation :-)
Anyway, before leaving, here is a note about another recording of the Forge webinar series published for your convenience and pleasure, and a happy resolution of a recent material asset texture related Revit API issue:
- [Forge webinar series and BIM360 recordings](#2)
- [Listing material asset textures and sub-textures](#3)
#### Forge Webinar Series and BIM360 Recordings
The [Forge webinar series](http://autodeskforge.devpost.com/details/webinars) continues.
Yesterday, Mikako Harada presented
an [Introduction to BIM360](http://adndevblog.typepad.com/cloud_and_mobile/2016/10/introduction-to-bim-360.html).
As always, API keys to try it out yourself can be obtained
from [developer.autodesk.com](https://developer.autodesk.com), which also hosts
the [BIM360 documentation](https://developer.autodesk.com/en/docs/bim360/v1).
In this context, you might also be interested in
the [Forge DevCon recording on BIM 360](https://www.youtube.com/watch?v=cOQEyI-EMAQ).
Here are the recordings and documentation pointers for all the topics covered so far:
- September 20
– [Introduction to Autodesk Forge and the Autodesk App Store](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-autodesk-forge-and-the-autodesk-app-store.html)
- September 22
– [Introduction to OAuth and Data Management API](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-oauth-and-data-management-api.html)
– on [OAuth](https://developer.autodesk.com/en/docs/oauth/v2/overview)
and [Data Management API](https://developer.autodesk.com/en/docs/data/v2/overview), providing token-based authentication, authorization and a unified and consistent way to access data across A360, Fusion 360, and the Object Storage Service.
- September 27
– [Introduction to Model Derivative API](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-model-derivative-api.html)
– on the [Model Derivative API](https://developer.autodesk.com/en/docs/model-derivative/v2/overview) that enables users to represent and share their designs in different formats and extract metadata.
- September 29
– [Introduction to Viewer](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-viewer-api.html)
– the [Viewer](https://developer.autodesk.com/en/docs/viewer/v2/overview), formerly part of the 'View and Data API', is a WebGL-based JavaScript library for 3D and 2D model rendering of CAD models from seed files, e.g., [AutoCAD](http://www.autodesk.com/products/autocad/overview), [Fusion 360](http://www.autodesk.com/products/fusion-360/overview), [Revit](http://www.autodesk.com/products/revit-family/overview) and many other formats.
- October 4
– [Introduction to Design Automation](http://adndevblog.typepad.com/cloud_and_mobile/2016/10/introduction-to-design-automation.html)
– the [Design Automation API](https://developer.autodesk.com/en/docs/design-automation/v2/overview), formerly known as 'AutoCAD I/O', enables running scripts on native CAD design files such as `DWG`,
cf. [Albert's tutorials](https://github.com/szilvaa/acadio-tutorials).
- October 6
– [Introduction to BIM360](http://adndevblog.typepad.com/cloud_and_mobile/2016/10/introduction-to-bim-360.html) and
the [Forge DevCon recording on BIM 360](https://www.youtube.com/watch?v=cOQEyI-EMAQ)
– [BIM360](https://developer.autodesk.com/en/docs/bim360/v1/overview) enables apps to integrate with BIM360 to extend its capabilities in the construction ecosystem.
For code samples on any of these, please refer to the Forge Platform samples on GitHub
at [Developer-Autodesk](https://github.com/Developer-Autodesk),
optionally adding a filter, e.g., like this for `Design.automation`: [...Developer-Autodesk?query=Design.automation](https://github.com/Developer-Autodesk?query=Design.automation).
The Forge Webinar series continues during the remainder of
the [Autodesk App Store Forge and Fusion 360 Hackathon](http://autodeskforge.devpost.com) until the end of October.
Next is session 7, an Introduction to Fusion 360 Client API:
- October 11 – [Fusion 360 Client API](http://help.autodesk.com/view/NINVFUS/ENU/?guid=GUID-A92A4B10-3781-4925-94C6-47DA85A4F65A) – an integrated CAD, CAM, and CAE tool for product development, built for the new ways products are designed and made.
- October 13 – Q&A on all APIs.
- October 20 – Q&A on all APIs.
- October 27 – Submitting a web service app to Autodesk App store.
Quick access links:
- For API keys, go to [developer.autodesk.com](https://developer.autodesk.com)
- For code samples, go to [github.com/Developer-Autodesk](https://github.com/Developer-Autodesk)
Feel free to contact us at [forgehackathon@autodesk.com](mailto:forgehackathon@autodesk.com) at any time with any questions.
#### Listing Material Asset Textures and Sub-Textures
An issue concerning access to material asset sub-textures was raised in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
on [Revit 2017 API bug report: wrong texture export ](http://forums.autodesk.com/t5/revit-api-forum/revit-2017-api-bug-report-wrong-texture-export/m-p/6566378) and
resolved by the development team:
\*\*Question:\*\* I work with Revit API and, seemingly, encountered a bug with asset content.
While exporting from `.rvt` project a material with the following texture characteristics:
![Material texture](img/material_texture.png)
In other words, textured material with a checker basic procedural texture and both 2 checkers as its sub-textures.
Exploring this through Python, I saw the following situation:
![Material texture asset code](img/material_texture_asset_code.png)
This can be described as follows: the first slot `(asset["checker_color1"])` was exported correctly, since the method:

```
  asset["checker_color1"].GetAllConnectedProperties()
```

returned an attached asset, but the second slot was exported incorrectly, because the corresponding method threw an exception and the `checker_color2` position is empty, although, as you can see above, there also must be an asset there.
\*\*Answer:\*\* I logged the issue \*REVIT-99286 [API: sub-texture missing in material asset -- 12174739]\* with our development team for further exploration.
They reply:
I cannot reproduce this issue in Revit 2016. I added layers of checker to a material's texture and with the API extraction I can see each layer being found and readable. I do not have the customer model, nor did I experiment with the debugger syntax indicated.
I suspect it's more likely the developer has a typo in his code rather than anything else.
I created a sample model with an embedded `ShowMaterialInfo` macro set to read from the checker material which is saved within it.
You can scroll through the output and see the multiple layers of connected assets and their nested properties.
I hope this helps.
I extracted the development team code from their macro and added it
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
module [CmdGetMaterials.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdGetMaterials.cs#L64-L652)
in [release 2017.0.130.4](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.130.4), cf.
the [diff](https://github.com/jeremytammik/the_building_coder_samples/compare/2017.0.130.3...2017.0.130.4).
By the way, this code pulls in two new namespaces that were not previously used:
```csharp
using System.ComponentModel;
using Autodesk.Revit.Utility;
```
Here is the complete material asset texture listing code including a couple of helper class definitions:
```csharp
#region List Material Asset Sub-Texture
///
/// A description of a property consists of a name, its attributes and value
/// here AssetPropertyPropertyDescriptor is used to wrap AssetProperty
/// to display its name and value in PropertyGrid
/// summary>
internal class AssetPropertyPropertyDescriptor : PropertyDescriptor
{
#region Fields
///
/// A reference to an AssetProperty
/// summary>
private AssetProperty m_assetProperty;
///
/// The type of AssetProperty's property "Value"
/// summary>m
private Type m_valueType;
///
/// The value of AssetProperty's property "Value"
/// summary>
private Object m_value;
#endregion
#region Properties
///
/// Property to get internal AssetProperty
/// summary>
public AssetProperty AssetProperty
{
get { return m_assetProperty; }
}
#endregion
#region override Properties
///
/// Gets a value indicating whether this property is read-only
/// summary>
public override bool IsReadOnly
{
get
{
return true;
}
}
///
/// Gets the type of the component this property is bound to.
/// summary>
public override Type ComponentType
{
get
{
return m_assetProperty.GetType();
}
}
///
/// Gets the type of the property.
/// summary>
public override Type PropertyType
{
get
{
return m_valueType;
}
}
#endregion
///
/// Public class constructor
/// summary>
/// the AssetProperty which a AssetPropertyPropertyDescriptor instance describesparam>
public AssetPropertyPropertyDescriptor( AssetProperty assetProperty )
: base( assetProperty.Name, new Attribute[0] )
{
m_assetProperty = assetProperty;
}
#region override methods
///
/// Compares this to another object to see if they are equivalent
/// summary>
/// The object to compare to this AssetPropertyPropertyDescriptor. param>
/// returns>
public override bool Equals( object obj )
{
AssetPropertyPropertyDescriptor other = obj as AssetPropertyPropertyDescriptor;
return other != null && other.AssetProperty.Equals( m_assetProperty );
}
///
/// Returns the hash code for this object.
/// Here override the method "Equals", so it is necessary to override GetHashCode too.
/// summary>
/// returns>
public override int GetHashCode()
{
return m_assetProperty.GetHashCode();
}
///
/// Resets the value for this property of the component to the default value.
/// summary>
/// The component with the property value that is to be reset to the default value.param>
public override void ResetValue( object component )
{
}
///
/// Returns whether resetting an object changes its value.
/// summary>
/// The component to test for reset capability.param>
/// true if resetting the component changes its value; otherwise, false.returns>
public override bool CanResetValue( object component )
{
return false;
}
/// G
/// Determines a value indicating whether the value of this property needs to be persisted.
/// summary>
/// The component with the property to be examined for persistence.param>
/// true if the property should be persisted; otherwise, false.returns>
public override bool ShouldSerializeValue( object component )
{
return false;
}
///
/// Gets the current value of the property on a component.
/// summary>
/// The component with the property for which to retrieve the value.param>
/// The value of a property for a given component.returns>
public override object GetValue( object component )
{
Tuple typeAndValue = GetTypeAndValue( m_assetProperty, 0 );
m_value = typeAndValue.Item2;
m_valueType = typeAndValue.Item1;
return m_value;
}
private static Tuple GetTypeAndValue( AssetProperty assetProperty, int level )
{
Object theValue;
Type valueType;
//For each AssetProperty, it has different type and value
//must deal with it separately
try
{
if( assetProperty is AssetPropertyBoolean )
{
AssetPropertyBoolean property = assetProperty as AssetPropertyBoolean;
valueType = typeof( AssetPropertyBoolean );
theValue = property.Value;
}
else if( assetProperty is AssetPropertyDistance )
{
AssetPropertyDistance property = assetProperty as AssetPropertyDistance;
valueType = typeof( AssetPropertyDistance );
theValue = property.Value;
}
else if( assetProperty is AssetPropertyDouble )
{
AssetPropertyDouble property = assetProperty as AssetPropertyDouble;
valueType = typeof( AssetPropertyDouble );
theValue = property.Value;
}
else if( assetProperty is AssetPropertyDoubleArray2d )
{
//Default, it is supported by PropertyGrid to display Double []
//Try to convert DoubleArray to Double []
AssetPropertyDoubleArray2d property = assetProperty as AssetPropertyDoubleArray2d;
valueType = typeof( AssetPropertyDoubleArray2d );
theValue = GetSystemArrayAsString( property.Value );
}
else if( assetProperty is AssetPropertyDoubleArray3d )
{
AssetPropertyDoubleArray3d property = assetProperty as AssetPropertyDoubleArray3d;
valueType = typeof( AssetPropertyDoubleArray3d );
theValue = GetSystemArrayAsString( property.Value );
}
else if( assetProperty is AssetPropertyDoubleArray4d )
{
AssetPropertyDoubleArray4d property = assetProperty as AssetPropertyDoubleArray4d;
valueType = typeof( AssetPropertyDoubleArray4d );
theValue = GetSystemArrayAsString( property.Value );
}
else if( assetProperty is AssetPropertyDoubleMatrix44 )
{
AssetPropertyDoubleMatrix44 property = assetProperty as AssetPropertyDoubleMatrix44;
valueType = typeof( AssetPropertyDoubleMatrix44 );
theValue = GetSystemArrayAsString( property.Value );
}
else if( assetProperty is AssetPropertyEnum )
{
AssetPropertyEnum property = assetProperty as AssetPropertyEnum;
valueType = typeof( AssetPropertyEnum );
theValue = property.Value;
}
else if( assetProperty is AssetPropertyFloat )
{
AssetPropertyFloat property = assetProperty as AssetPropertyFloat;
valueType = typeof( AssetPropertyFloat );
theValue = property.Value;
}
else if( assetProperty is AssetPropertyInteger )
{
AssetPropertyInteger property = assetProperty as AssetPropertyInteger;
valueType = typeof( AssetPropertyInteger );
theValue = property.Value;
}
else if( assetProperty is AssetPropertyReference )
{
AssetPropertyReference property = assetProperty as AssetPropertyReference;
valueType = typeof( AssetPropertyReference );
theValue = "REFERENCE"; //property.Type;
}
else if( assetProperty is AssetPropertyString )
{
AssetPropertyString property = assetProperty as AssetPropertyString;
valueType = typeof( AssetPropertyString );
theValue = property.Value;
}
else if( assetProperty is AssetPropertyTime )
{
AssetPropertyTime property = assetProperty as AssetPropertyTime;
valueType = typeof( AssetPropertyTime );
theValue = property.Value;
}
else
{
valueType = typeof( String );
theValue = "Unprocessed asset type: " + assetProperty.GetType().Name;
}
if( assetProperty.NumberOfConnectedProperties > 0 )
{
String result = "";
result = theValue.ToString();
TaskDialog.Show( "Connected properties found", assetProperty.Name + ": " + assetProperty.NumberOfConnectedProperties );
IList properties = assetProperty.GetAllConnectedProperties();
foreach( AssetProperty property in properties )
{
if( property is Asset )
{
// Nested?
Asset asset = property as Asset;
int size = asset.Size;
for( int i = 0; i < size; i++ )
{
AssetProperty subproperty = asset[i];
Tuple valueAndType = GetTypeAndValue( subproperty, level + 1 );
String indent = "";
if( level > 0 )
{
for( int iLevel = 1; iLevel <= level; iLevel++ )
indent += " ";
}
result += "\n " + indent + "- connected: name: " + subproperty.Name + " | type: " + valueAndType.Item1.Name +
" | value: " + valueAndType.Item2.ToString();
}
}
}
theValue = result;
}
}
catch
{
return null;
}
return new Tuple( valueType, theValue );
}
///
/// Sets the value of the component to a different value.
/// For AssetProperty, it is not allowed to set its value, so here just return.
/// summary>
/// The component with the property value that is to be set. param>
/// The new value.param>
public override void SetValue( object component, object value )
{
return;
}
#endregion
///
/// Convert Autodesk.Revit.DB.DoubleArray to Double [].
/// For Double [] is supported by PropertyGrid.
/// summary>
/// the original Autodesk.Revit.DB.DoubleArray param>
/// The converted Double []returns>
private static Double[] GetSystemArray( DoubleArray doubleArray )
{
double[] values = new double[doubleArray.Size];
int index = 0;
foreach( Double value in doubleArray )
{
values[index++] = value;
}
return values;
}
private static String GetSystemArrayAsString( DoubleArray doubleArray )
{
double[] values = GetSystemArray( doubleArray );
String result = "";
foreach( double d in values )
{
result += d;
result += ",";
}
return result;
}
}
///
/// supplies dynamic custom type information for an Asset while it is displayed in PropertyGrid.
/// summary>
public class RenderAppearanceDescriptor : ICustomTypeDescriptor
{
#region Fields
///
/// Reference to Asset
/// summary>
Asset m_asset;
///
/// Asset's property descriptors
/// summary>
PropertyDescriptorCollection m_propertyDescriptors;
#endregion
#region Constructors
///
/// Initializes Asset object
/// summary>
/// an Asset objectparam>
public RenderAppearanceDescriptor( Asset asset )
{
m_asset = asset;
GetAssetProperties();
}
#endregion
#region Methods
#region ICustomTypeDescriptor Members
///
/// Returns a collection of custom attributes for this instance of Asset.
/// summary>
/// Asset's attributesreturns>
public AttributeCollection GetAttributes()
{
return TypeDescriptor.GetAttributes( m_asset, false );
}
///
/// Returns the class name of this instance of Asset.
/// summary>
/// Asset's class namereturns>
public string GetClassName()
{
return TypeDescriptor.GetClassName( m_asset, false );
}
///
/// Returns the name of this instance of Asset.
/// summary>
/// The name of Assetreturns>
public string GetComponentName()
{
return TypeDescriptor.GetComponentName( m_asset, false );
}
///
/// Returns a type converter for this instance of Asset.
/// summary>
/// The converter of the Assetreturns>
public TypeConverter GetConverter()
{
return TypeDescriptor.GetConverter( m_asset, false );
}
///
/// Returns the default event for this instance of Asset.
/// summary>
/// An EventDescriptor that represents the default event for this object,
/// or null if this object does not have events.returns>
public EventDescriptor GetDefaultEvent()
{
return TypeDescriptor.GetDefaultEvent( m_asset, false );
}
///
/// Returns the default property for this instance of Asset.
/// summary>
/// A PropertyDescriptor that represents the default property for this object,
/// or null if this object does not have properties.returns>
public PropertyDescriptor GetDefaultProperty()
{
return TypeDescriptor.GetDefaultProperty( m_asset, false );
}
///
/// Returns an editor of the specified type for this instance of Asset.
/// summary>
/// A Type that represents the editor for this object. param>
/// An Object of the specified type that is the editor for this object,
/// or null if the editor cannot be found.returns>
public object GetEditor( Type editorBaseType )
{
return TypeDescriptor.GetEditor( m_asset, editorBaseType, false );
}
///
/// Returns the events for this instance of Asset using the specified attribute array as a filter.
/// summary>
/// An array of type Attribute that is used as a filter. param>
/// An EventDescriptorCollection that represents the filtered events for this Asset instance.returns>
public EventDescriptorCollection GetEvents( Attribute[] attributes )
{
return TypeDescriptor.GetEvents( m_asset, attributes, false );
}
///
/// Returns the events for this instance of Asset.
/// summary>
/// An EventDescriptorCollection that represents the events for this Asset instance.returns>
public EventDescriptorCollection GetEvents()
{
return TypeDescriptor.GetEvents( m_asset, false );
}
///
/// Returns the properties for this instance of Asset using the attribute array as a filter.
/// summary>
/// An array of type Attribute that is used as a filter.param>
/// A PropertyDescriptorCollection that
/// represents the filtered properties for this Asset instance.returns>
public PropertyDescriptorCollection GetProperties( Attribute[] attributes )
{
return m_propertyDescriptors;
}
///
/// Returns the properties for this instance of Asset.
/// summary>
/// A PropertyDescriptorCollection that represents the properties
/// for this Asset instance.returns>
public PropertyDescriptorCollection GetProperties()
{
return m_propertyDescriptors;
}
///
/// Returns an object that contains the property described by the specified property descriptor.
/// summary>
/// A PropertyDescriptor that represents the property whose owner is to be found. param>
/// Asset objectreturns>
public object GetPropertyOwner( PropertyDescriptor pd )
{
return m_asset;
}
#endregion
///
/// Get Asset's property descriptors
/// summary>
private void GetAssetProperties()
{
if( null == m_propertyDescriptors )
{
m_propertyDescriptors = new PropertyDescriptorCollection( new AssetPropertyPropertyDescriptor[0] );
}
else
{
return;
}
//For each AssetProperty in Asset, create an AssetPropertyPropertyDescriptor.
//It means that each AssetProperty will be a property of Asset
for( int index = 0; index < m_asset.Size; index++ )
{
AssetProperty assetProperty = m_asset[index];
if( null != assetProperty )
{
AssetPropertyPropertyDescriptor assetPropertyPropertyDescriptor = new AssetPropertyPropertyDescriptor( assetProperty );
m_propertyDescriptors.Add( assetPropertyPropertyDescriptor );
}
}
}
#endregion
}
public void ShowMaterialInfo( Document doc )
{
// Find material
FilteredElementCollector fec = new FilteredElementCollector( doc );
fec.OfClass( typeof( Material ) );
String materialName = "Checker"; // "Copper";//"Prism - Glass - Textured"; // "Parking Stripe"; // "Copper";
// "Carpet (1)";// "Prism - Glass - Textured";// "Parking Stripe"; // "Prism 1";// "Brick, Common" ;// "Acoustic Ceiling Tile 24 x 48"; // "Aluminum"
Material mat = fec.Cast().First( m => m.Name == materialName );
ElementId appearanceAssetId = mat.AppearanceAssetId;
AppearanceAssetElement appearanceAsset = doc.GetElement( appearanceAssetId ) as AppearanceAssetElement;
Asset renderingAsset = appearanceAsset.GetRenderingAsset();
RenderAppearanceDescriptor rad
= new RenderAppearanceDescriptor( renderingAsset );
PropertyDescriptorCollection collection = rad.GetProperties();
TaskDialog.Show( "Total properties", "Properties found: " + collection.Count );
//
string s = ": Material Asset Properties";
TaskDialog dlg = new TaskDialog( s );
dlg.MainInstruction = mat.Name + s;
s = string.Empty;
List orderableCollection = new List( collection.Count );
foreach( PropertyDescriptor descr in collection )
{
orderableCollection.Add( descr );
}
foreach( AssetPropertyPropertyDescriptor descr in
orderableCollection.OrderBy( pd => pd.Name ).Cast() )
{
object value = descr.GetValue( rad );
s += "\nname: " + descr.Name
+ " | type: " + descr.PropertyType.Name
+ " | value: " + value;
}
dlg.MainContent = s;
dlg.Show();
}
public void ListAllAssets(UIApplication uiapp)
{
AssetSet assets = uiapp.Application.get_Assets( AssetType.Appearance );
TaskDialog dlg = new TaskDialog( "Assets" );
String assetLabel = "";
foreach( Asset asset in assets )
{
String libraryName = asset.LibraryName;
AssetPropertyString uiname = asset["UIName"] as AssetPropertyString;
AssetPropertyString baseSchema = asset["BaseSchema"] as AssetPropertyString;
assetLabel += libraryName + " | " + uiname.Value + " | " + baseSchema.Value;
assetLabel += "\n";
}
dlg.MainInstruction = assetLabel;
dlg.Show();
}
#endregion // List Material Asset Sub-Texture
```
I hope you are glad to hear that no issues were detected and that you can make good use of this handy sample code for further explorations.
Many thanks to Scott Conover for putting it together!
I wish you a happy and fruitful week!