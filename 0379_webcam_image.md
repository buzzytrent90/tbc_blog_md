---
post_number: "0379"
title: "Grabbing an Internet Webcam Image"
slug: "webcam_image"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'levels', 'python', 'revit-api', 'schedules', 'views', 'windows']
source_file: "0379_webcam_image.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0379_webcam_image.html"
---

### Grabbing an Internet Webcam Image

I am in full tilt preparing for the
[AEC DevCamp](http://thebuildingcoder.typepad.com/blog/2010/03/devcamp-devlabs-and-updated-api-training-schedule.html) conference
([web site](http://devcamps2010.autodeskevents.net/index.cfm?event=cms.page&id=SID3FA5EEFA001D080A3456040FDD1BD93C)) in
Boston next week, June 7-9, and the
[DevLabs](http://thebuildingcoder.typepad.com/blog/2010/03/devcamp-devlabs-and-updated-api-training-schedule.html#devlabs) in Waltham following immediately afterwards
([register](http://usa.autodesk.com/adsk/servlet/item?id=6703509&siteID=123112&cname=DevLab (AEC), Waltham, Jun 10 2010, 201039)).

One of my sessions at the DevCamp discusses migration to and advanced use of the new Revit 2011 API features.
Since some of the new features will be covered by other presentations held by members of the development team, I picked an area that was off their radar and ended up choosing the
[Idling event](http://thebuildingcoder.typepad.com/blog/2010/04/idling-event.html) that I recently talked about.
I mentioned that it provides some exciting new possibilities, since it allows more interaction with external applications and goes one step closer towards providing
[asynchronous access](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html) to the Revit API.

Looking for an example of using this event, I pondered various potential providers of external graphical data that changes over time.
One obvious source of live images is an Internet webcam.

The idea of displaying a live image in the Revit model also has the added advantage of allowing me to try my hand at using the new analysis visualisation framework and setting up an own spatial field primitive to display pixel data on the face of a Revit model element.
We looked at one such example discussing the DistanceToSurfaces SDK sample making use of the
[dynamic model update](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html#2) mechanism.

#### Reading the Matterhorn Webcam JPEG Image Data

Before I dive into the Revit specific use of this, let me explain how I retrieve the image data that I plan to use from the Internet, since I spent some time myself exploring that issue.
It took me a while to discover that all I have to do to access a webcam image is establish what URL to use and then read the byte data from it to obtain the jpeg data to display.

For instance, one of the first webcam candidates that sprang to mind was the
[Matterhorn](http://en.wikipedia.org/wiki/Matterhorn),
probably the best-know Swiss mountain.
One Matterhorn webcam image is provided by the URL
<http://www.ggb.ch/webcam.php>.
Exploring the HTML information provided on that page, I discovered that the webcam image itself is read from the URL
<http://www.ggb.ch/webcam.php?e_1__getimage=1>.

Here is a short Python script that shows how easy it is to read this image data from the URL and create a JPEG file named 'a.jpg' from it:
```python
import urllib
def get\_html\_content( url ):
f = urllib.urlopen( url )
s = f.read()
f.close()
return s
def main():
url = 'http://www.ggb.ch/webcam.php?e\_1\_\_getimage=1'
data = get\_html\_content( url )
f = open( 'a.jpg', 'wb' )
f.write( data )
f.close()
main()
```

#### Grabbing the Matterhorn Webcam Image in .NET

Thanks to the powerful libraries provided by the .NET framework, reading the Matterhorn image data in .NET is not much less succinct than the minimalistic Python script.
This code also adds a call to Process.Start to test display the result in the default image viewer:
```csharp
using System.Diagnostics;
using System.Drawing;
using System.Drawing.Imaging;
using System.Net;
using System.IO;

namespace GrabJpg
{
  class Program
  {
    const string \_url =
      "http://www.ggb.ch/webcam.php?e\_1\_\_getimage=1";

    static void Main( string[] args )
    {
      using( WebClient client = new WebClient() )
      {
        byte[] data = client.DownloadData( \_url );

        using( Image img = Image.FromStream(
          new MemoryStream( data ) ) )
        {
          string filename = "a.jpg";
          img.Save( filename, ImageFormat.Jpeg );
          Process.Start( filename ); // test display
        }
      }
    }
  }
}
```

To ensure that all possible required information is available, here is
[GrabJpg.zip](zip/GrabJpg.zip)
containing the source code and Visual Studio solution for this sample.

#### Picadilly Circus is More Dynamic

After looking at the Matterhorn image a couple of times and noticing that it does not change as much as I would like, since its main use it to show what the current weather is like up there, I looked around for something a little more dynamic.

My next candidate was
[Picadilly Circus](http://en.wikipedia.org/wiki/Piccadilly_Circus) in London, since that tends to have some people and vehicles moving around it.
I found a webcam displaying an image of it which required a slightly more convoluted path to access the final underlying image data, since its URL is embedded within a tag named 'snapshoturl' in an XML data stream.
Anyway, the image data URL that one ends up with is
<http://cameras.camvista.com/webcams/cam2_Piccadilly.jpg>.

Here is a Python script accessing that image data, creating a JPEG file named 'a.jpg' from it, and starting up the default image viewer to display it on the screen:
```python
import popen2, urllib
def get\_html\_content( url ):
f = urllib.urlopen( url )
s = f.read()
f.close()
return s
def main():
url = 'http://www.camvista.com/xml/10086.xml'
data = get\_html\_content( url )
# data is an xml file pointing to the jpg url
print data
tag = 'snapshoturl'
i = data.find( '<' + tag + '>' )
print i
if -1 < i:
i = i + len(tag) + 2
print i
url = data[i:]
print url
i = url.find( '</' + tag + '>' )
print i
url = url[:i]
print url
data = get\_html\_content( url )
f = open( 'a.jpg', 'wb' )
f.write( data )
f.close()
popen2.popen2( 'a.jpg' )
main()
```

A typical view of Picadilly Circus from this webcam on an early Thursday morning before too many people start milling around looks like this:

![Webcam image of Picadilly Circus](img/webcam_picadilly.jpg)

#### Conversion to Greyscale Image

Another issue that I need to consider in order to display a bitmap image using the Revit analysis visualisation framework is that the latter is not designed to display RGB data.
In order to use it to display a colour image, one option would be to calculate a palette of selected colours based on the most common pixel colours in the given image, and then assign each display point in the Revit model one of the palette colours.
Another possibility, which seems a lot simpler to me initially, is to convert the colour image to a greyscale one, and then use values between zero and one for the analysis data to feed into the AVF spatial field primitive.

One method that I found to convert a bitmap to a greyscale image using the .NET framework is to make use of the FormatConvertedBitmap class.
Unfortunately, it cannot be initialised directly from the Image instance that we created from the JPEG image data stream.
It does provide a constructor taking a BitmapSource argument, though, and I discovered that I can create such an instance from the Image that I have using the following method:
```csharp
static BitmapSource ConvertBitmapToBitmapSource(
  Bitmap bmp )
{
  return System.Windows.Interop.Imaging
    .CreateBitmapSourceFromHBitmap(
      bmp.GetHbitmap(),
      IntPtr.Zero,
      Int32Rect.Empty,
      BitmapSizeOptions.FromEmptyOptions() );
}
```

With that method in place, I can pass the image data through the following conversion sequence to feed the format conversion:

1. byte[] from URL using DownloadData- MemoryStream from byte[] using constructor- Image from MemoryStream using Image.FromStream- Bitmap from Image using GetThumbnailImage- BitmapSource from Bitmap using ConvertBitmapToBitmapSource- Set FormatConvertedBitmap.Source using BitmapSource

I end up with the following little console application mainline to read the webcam image data from the specified URL, convert it to a greyscale image, save that to a JPEG file, and display it for test purposes:
```csharp
const string \_url =
  "http://www.ggb.ch/webcam.php?e\_1\_\_getimage=1";

static void Main( string[] args )
{
  using( WebClient client = new WebClient() )
  {
    byte[] data = client.DownloadData( \_url );

    using( Image img = Image.FromStream(
      new MemoryStream( data ) ) )
    {
      string filename = "a.jpg";
      img.Save( filename, ImageFormat.Jpeg );

      // test display original:

      Process.Start( filename );

      // scale down to half size in each direction:

      int w = img.Width >> 1;
      int h = img.Height >> 1;

      Bitmap bitmap = new Bitmap(
        img.GetThumbnailImage( w, h, null,
          IntPtr.Zero ) );

      FormatConvertedBitmap grey
        = new FormatConvertedBitmap();

      grey.BeginInit();

      grey.Source
        = ConvertBitmapToBitmapSource( bitmap );

      grey.DestinationFormat
        = PixelFormats.Gray32Float;

      grey.EndInit();

      JpegBitmapEncoder encoder
        = new JpegBitmapEncoder();

      encoder.Frames.Add(
        BitmapFrame.Create( grey ) );

      encoder.QualityLevel = 25;

      string filename2 = "a2.jpg";

      FileStream file = new FileStream(
        filename2, FileMode.Create,
        FileAccess.Write );

      encoder.Save( file );
      file.Close();

      // test display greyscale:

      Process.Start( filename2 );
    }
  }
}
```

Here is
[GrabGreyJpg.zip](zip/GrabGreyJpg.zip)
containing the complete source code and Visual Studio solution for this console application.

With these preparations in place, we are all set to return to Revit and make use of these image grabbing methods to implement an Idling event handler to paint a real-time view onto a face of any arbitrary building element.
For instance, wouldn't it be neat to create a building model whose windows display a real-time view of some beautiful natural park?