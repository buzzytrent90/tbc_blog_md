---
post_number: "1650"
title: "Scale Icon Image"
slug: "scale_icon_image"
author: "Jeremy Tammik"
tags: ['revit-api', 'rooms', 'sheets']
source_file: "1650_scale_icon_image.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1650_scale_icon_image.html"
---

### Scaling a Bitmap for the Large and Small Image Icons
Every time I created a ribbon button, I was faced with the task of creating appropriately scaled icons for it to populate the `PushButton` large and small image icon properties `LargeImage` and `Image`.
They seem to expect a 32 x 32 and 16 x 16 icon, respectively.
I finally solved that once and for all by implementing a couple of methods to perform automatic bitmap scaling:
- [BitmapImageToBitmap](#3) – convert a `BitmapImage` to `Bitmap`
- [BitmapToBitmapSource](#4) – convert a `Bitmap` to a `BitmapSource`
- [ResizeImage](#5) – resize an image to the specified width and height
- [ScaledIcon](#6) – return a scaled down icon of desired size for Revit ribbon button
- [Usage sample](#7) – putting them together
Here they are one by one:
#### BitmapImageToBitmap
```csharp
///
/// Convert a BitmapImage to Bitmap
/// summary>
static Bitmap BitmapImageToBitmap(
BitmapImage bitmapImage )
{
//BitmapImage bitmapImage = new BitmapImage(
// new Uri("../Images/test.png", UriKind.Relative));
using( MemoryStream outStream = new MemoryStream() )
{
BitmapEncoder enc = new BmpBitmapEncoder();
enc.Frames.Add( BitmapFrame.Create( bitmapImage ) );
enc.Save( outStream );
Bitmap bitmap = new Bitmap( outStream );
return new Bitmap( bitmap );
}
}
```
#### BitmapToBitmapSource
```csharp
[System.Runtime.InteropServices.DllImport( "gdi32.dll" )]
public static extern bool DeleteObject( IntPtr hObject );
///
/// Convert a Bitmap to a BitmapSource
/// summary>
static BitmapSource BitmapToBitmapSource( Bitmap bitmap )
{
IntPtr hBitmap = bitmap.GetHbitmap();
BitmapSource retval;
try
{
retval = Imaging.CreateBitmapSourceFromHBitmap(
hBitmap, IntPtr.Zero, Int32Rect.Empty,
BitmapSizeOptions.FromEmptyOptions() );
}
finally
{
DeleteObject( hBitmap );
}
return retval;
}
```
#### ResizeImage
```csharp
///
/// Resize the image to the specified width and height.
/// summary>
/// The image to resize.param>
/// The width to resize to.param>
/// The height to resize to.param>
/// The resized image.returns>
static Bitmap ResizeImage(
Image image,
int width,
int height )
{
var destRect = new System.Drawing.Rectangle(
0, 0, width, height );
var destImage = new Bitmap( width, height );
destImage.SetResolution( image.HorizontalResolution,
image.VerticalResolution );
using( var g = Graphics.FromImage( destImage ) )
{
g.CompositingMode = CompositingMode.SourceCopy;
g.CompositingQuality = CompositingQuality.HighQuality;
g.InterpolationMode = InterpolationMode.HighQualityBicubic;
g.SmoothingMode = SmoothingMode.HighQuality;
g.PixelOffsetMode = PixelOffsetMode.HighQuality;
using( var wrapMode = new ImageAttributes() )
{
wrapMode.SetWrapMode( WrapMode.TileFlipXY );
g.DrawImage( image, destRect, 0, 0, image.Width,
image.Height, GraphicsUnit.Pixel, wrapMode );
}
}
return destImage;
}
```
#### ScaledIcon
`ScaledIcon` simply calls the three helper methods defined above to return a scaled version of the input image:
```csharp
///
/// Scale down large icon to desired size for Revit
/// ribbon button, e.g., 32 x 32 or 16 x 16
/// summary>
static BitmapSource ScaledIcon(
BitmapImage large_icon,
int w,
int h )
{
return BitmapToBitmapSource( ResizeImage(
BitmapImageToBitmap( large_icon ), w, h ) );
}
```
#### Usage Sample
Within the external application `PopulatePanel` method, simply read the embedded resource icon image and apply `ScaledIcon` to it to populate the large and small image properties with appropriately scaled images:
```csharp
BitmapImage bmi = new BitmapImage( new Uri(
"icons/cmdx.png", UriKind.Relative ) );
PushButton pb = p.AddItem( new PushButtonData(
"Command", "Command", path, "CmdX" ) ) as PushButton;
pb.ToolTip = "Do something fantastic";
pb.LargeImage = ScaledIcon( bmi, 32, 32 );
pb.Image = ScaledIcon( bmi, 16, 16 );
```
![Embedded icon resource](img/roomedit_2014_embedded_icon_resource.png)