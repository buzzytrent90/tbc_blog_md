---
post_number: "0230"
title: "Unrotate North"
slug: "unrotate_north"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'python', 'revit-api', 'selection', 'views']
source_file: "0230_unrotate_north.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0230_unrotate_north.html"
---

### Unrotate North

**Update:** A better algorithm than the one presented below is provided by the discussion on the
[project location](http://thebuildingcoder.typepad.com/blog/2010/01/project-location.html).

I spent this weekend on the
[Rigi Mountain](http://en.wikipedia.org/wiki/Rigi) together
with seventy other people from the alpine club.
Saturday it rained quite hard most of the day, unfortunately, but a couple of us still went on a very nice hike from Rigi Scheidegg across to and over the top of Rigi Hochflueh with a beautiful view down on the
[Lucerne Lake](http://en.wikipedia.org/wiki/Lake_Lucerne) and the
[Rütli Meadow](http://en.wikipedia.org/wiki/R%C3%BCtli).
In the evening, we also tested how many people can be shoehorned into a wood-heated hot tub outside in snow and rain, among other interesting pastimes.
Sunday I went climbing with Angela.
Considering the rain from the day before and the night and it still being quite cloudy on Sunday, we were maybe the only two crazy enough to even try it.
It involved the same walk from Scheidegg to Hochflueh, then searching for the climbing area, doing the climbs, and, last but not least, getting back again safely.
The climbing area is called
[Thedys Gärtli](http://www.kley.ch/hansjoerg/2004/2004_4_1.html)
and is on the Hochflue slabs close to the summit, so we could all the time enjoy the same beautiful view as the day before.
We were extremely pleased to be able to do two very nice climbs, among the easier ones, Männertrü (4c) and Fürlinie (5c).
Although we knew when we started the latter one that we would get back rather late, we actually ended up descending the mountain in total darkness and spending most of the night trying to get back to civilisation.
Or, rather, all of the night, since I missed the last train out of there.

Anyway, meanwhile, our inexorable exploration of the Revit API continues, and here is another question to address which expands on our analysis of the calculation of the
[azimuth](http://thebuildingcoder.typepad.com/blog/2008/10/azimuth.html)
or Revit elements:

**Question:** I am using the Element.Location property to query the X and Y coordinate values of an element.
I also make use of the project location and project angle.
I have a problem if the Project North has been rotated.
How can I determine the 'real' X and Y coordinates of an element if the Project North has been rotated?

**Answer:** Ok, I see what you mean.

To dive into this a little bit deeper, I implemented a new Building Coder sample command CmdUnrotateNorth to calculate the coordinates of the original unrotated element location for you.
It transforms the element location back to the original coordinates to cancel effect of rotating the project north
by simply determining the project north rotation angle 'a' and applying a similar transformation to the element coordinates like this:

```
  Transform t = Transform.get_Rotation(
    XYZ.Zero, XYZ.BasisZ, a );
```

To simplify testing, I implemented a helper method GetElementLocation which defines a location point for any given element having a non-null Location property:
```csharp
bool GetElementLocation(
out XYZ p,
Element e )
{
p = XYZ.Zero;
bool rc = false;
Location loc = e.Location;
if( null != loc )
{
LocationPoint lp = loc as LocationPoint;
if( null != lp )
{
p = lp.Point;
rc = true;
}
else
{
LocationCurve lc = loc as LocationCurve;
Debug.Assert( null != lc,
"expected location to be either point or curve" );
p = lc.Curve.get\_EndPoint( 0 );
rc = true;
}
}
return rc;
}
```

In the main method, I check that one single element is selected and then list its location point as returned by the API, or rather by this method.
I then iterate over all
project locations, print each one's the project north angle and list the element location point rotated around the origin by that angle.

I tested this by selecting an element and running the command with it in its original unrotated position, which returns:

```
Selected element location: (-23.64,1.64,0)
Angle between project north and true north: 0 degrees
Unrotated element location: (-23.64,1.64,0)
```

After applying a rotation to the project north by calling Manage > Project Location > Position > Rotate Project North > Align selected line or plane, invoking the same command again returns the following:

```
Selected element location: (-21.66,9.62,0)
Angle between project north and true north: 20 degrees
Unrotated element location: (-23.64,1.64,0)
```

As you can see, the element location returned by the API is different, but the original value can easily be obtained by applying the rotation described.

Here is the code of the entire mainline of the sample command in its Execute method:
```python
Application app = commandData.Application;
Document doc = app.ActiveDocument;
ElementSet els = doc.Selection.Elements;
if( 1 != els.Size )
{
message = "Please select a single element.";
}
else
{
ElementSetIterator it = els.ForwardIterator();
it.MoveNext();
Element e = it.Current as Element;
XYZ p;
if( !GetElementLocation( out p, e ) )
{
message
= "Selected element has no location defined.";
Debug.Print( message );
}
else
{
string msg
= "Selected element location: "
+ Util.PointString( p );
double pna = 0;
foreach( ProjectLocation location
in doc.ProjectLocations )
{
ProjectPosition projectPosition
= location.get\_ProjectPosition( XYZ.Zero );
pna = projectPosition.Angle;
msg +=
"\nAngle between project north and true north: "
+ Util.AngleString( pna );
Transform t = Transform.get\_Rotation(
XYZ.Zero, XYZ.BasisZ, pna );
msg +=
"\nUnrotated element location: "
+ Util.PointString( t.OfPoint( p ) );
Util.InfoMsg( msg );
}
}
}
return IExternalCommand.Result.Failed;
```

Here is
[version 1.1.0.50](zip/bc11050.zip)
of the complete Building Coder sample source code and Visual Studio solution including the new command.