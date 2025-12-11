---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.9
content_type: code_example
optimization_date: '2025-12-11T11:44:14.341247'
original_url: https://thebuildingcoder.typepad.com/blog/0656_unofficial_param.html
post_number: '0656'
reading_time_minutes: 12
series: general
slug: unofficial_param
source_file: 0656_unofficial_param.htm
tags:
- csharp
- doors
- elements
- family
- geometry
- parameters
- python
- revit-api
- rooms
- selection
- sheets
- views
- walls
title: Unofficial Parameters and BipChecker
word_count: 2493
---

### Unofficial Parameters and BipChecker

I climbed the
[Nadelhorn](http://www.mountwiki.com/wiki/view/Nadelhorn) last
weekend, the highest point in Switzerland for me so far, and the second highest point I have ever been, actually, 4387 m.
Except for airplanes, of course.
Here is a picture of the panorama from the Nadehorn summit with Monte Rosa, Dom, Matterhorn, Mont Blanc, etc.

![Nadelhorn summit panorama](file:////j/photo/jeremy/2011/2011-09-23_nadelhorn/56.jpg)

Here are some
[more pictures](https://picasaweb.google.com/104316998805199988071/Nadelhorn4327mSaisonabschlussMitJeremy?authuser=0&authkey=Gv1sRgCJnTodeiktKZ4wE) of
our tour.

Meanwhile, on the Revit API side, Victor Chekalin, or Виктор Чекалин, raised a number of pertinent issues lately.
One of them is this question in the
[comment](http://thebuildingcoder.typepad.com/blog/2008/11/exploring-element-parameters.html?cid=6a00e553e168978833015391c90d6e970b#comment-6a00e553e168978833015391c90d6e970b) on
parameters:

**Question:** In my project I find parameters on a particular element that do not appear in the Element.Parameters collection.
In
[exploring element parameters](http://thebuildingcoder.typepad.com/blog/2008/11/exploring-element-parameters.html),
you wrote: "Sometimes an element will have other parameters attached to it as well, which are not listed in the 'official' collection returned by the Parameters property.
To explore all parameters attached to an element, one can make use of the built-in parameter checker."
And nothing more.
Can you explain the meaning of these parameters?
In my case I need to get the length of any element if length exists.
I wrote the
[following code](http://pastebin.com/GgKFmxdD) for it:
```csharp
private Double? GetElementLength( Element element )
{
  var lengthParam
    = Enum.GetValues( typeof( BuiltInParameter ) )
      .OfType<BuiltInParameter>()
      .Where( p => p.ToString().Contains( "LENGTH" ) )
      .Select( param => element.get\_Parameter( param ) )
      .Where( param => param != null )
      .FirstOrDefault();

  if( lengthParam != null )
  {
    var lengthValue = lengthParam.AsDouble();
    return lengthValue;
  }

  return null;
}
```

At first in BuiltInParameters I find all parameters whose name contains the word "LENGTH".
Then I check for each if this BuiltInParameter exists in element.
If it exists, return the value of this parameter.
The problem: there are sometimes no length parameters in Element.Parameters, but I can still retrieve such a parameter via Element.get\_Parameter.
Of course I can get the length of the element by enumerating all parameters and checking the parameter name for length, but I'm curious about these "strange invisible" parameters.

**Answer:** Yes, there may be lots of parameters attached to elements that do not appear in the 'official' Element.Parameters collection.

You can retrieve them using any of the standard Element.Parameter property overloads.

#### The Element.Parameter Property

While we are at it, here are a few observations of strangeness on the Parameter property itself:

- Yes, it is a property, and still, yes, it takes an argument.- Yes, it does not exist in C# per se, because it is called get\_Parameter there instead.- Yes, these two observations may be connected, even mutually dependent.

The complete list of arguments accepted by the overloads of the Parameter property or get\_Parameter method is:

- BuiltInParameter: Retrieves a parameter from the element given a parameter id.- Definition: Retrieves a parameter from the element based on its definition.- Guid: Retrieves a parameter from the element given a GUID for a shared parameter.- String: Retrieves a parameter from the element given its name.

The Definition and name string can be provided and used for any parameter.
The BuiltInParameter enumeration value or 'parameter id' as it is called above only works for built-in parameters, and the GUID only for shared ones, so they work for mutually exclusive situations.

So if the parameter you are looking for is named "Length", you can retrieve it using that name.

I would recommend you **not to do so**, however, because this makes you code language dependent.

Better is to determine exactly which built-in parameter corresponds to it and use that instead.

Another issue with the approach you describe above is that for a complex element, there may be several different built-in parameters whose enumeration name contains the substring "LENGTH", and they may have completely different meanings.
It is even not completely clear what you yourself mean by 'length' of an element.

#### Snooping and the Built-in Parameter Checker

Here is a code snippet with a loop that shows how to retrieve all parameters from an element. Actually, it may not retrieve all, but it mostly retrieves more than the official Parameters collection contains:
```csharp
Array bips = Enum.GetValues(typeof(BuiltInParameter));
Parameter p;
foreach (BuiltInParameter a in bips)
{
try
{
p = elem.get\_Parameter(a);
}
catch { }
}
```

This kind of loop to retrieve a value for each and every built-in parameter enumeration value is also used by the RevitLookup 'Built-In Enum Snoop' in the Snoop Parameters dialogue and my own built-in parameter checker.
Here are some explorations I made in the past in which I discussed the use of these tools and provided the source code for my built-in parameter checker:

- [Exploring element parameters](http://thebuildingcoder.typepad.com/blog/2008/11/exploring-element-parameters.html)- [Locked dimensioning](http://thebuildingcoder.typepad.com/blog/2009/02/locked-dimensioning.html)- [Deeper parameter exploration](http://thebuildingcoder.typepad.com/blog/2009/04/deeper-parameter-exploration.html)- [Door marks](http://thebuildingcoder.typepad.com/blog/2009/09/door-marks.html)- [Room occupancy](http://thebuildingcoder.typepad.com/blog/2009/11/room-occupancy.html)- [Title block of sheet](http://thebuildingcoder.typepad.com/blog/2009/11/title-block-of-sheet.html)- [Pre-, post- and pick selection](http://thebuildingcoder.typepad.com/blog/2010/05/pre-post-and-pick-select.html)- [Determining sheet size](http://thebuildingcoder.typepad.com/blog/2010/05/determine-sheet-size.html)

Since I do find my built-in parameter checker a very handy tool I decided to simplify access to and installation of it by extracting the code from the obsolete Revit API Introduction Labs.

Besides simply displaying the list of 'unofficial' element parameter values, like the RevitLookup 'Built-In Enum Snoop' functionality, this tool provides the following additional advantages:

- For elements than cannot be selected on the screen, prompt to enter the element id to select them instead.- For elements with a type, prompt whether to display type or instance parameter data.- List the results in a data grid view.- Display the built-in parameter enumeration value and the user visible parameter name side by side.- Display both the internal database value and the user visible string value of real-valued data.- Sort the results by the values of any column.- Copy the entire list of data to the clipboard.

The ability to sort by any column makes it much easier to find the records you are interested in, since you can sort by and then locate by either built-in parameter enumeration value, user visibly name, type, string or database value, or even read-write status.

Extracting and cleaning up this code also provided a welcome opportunity to make use of the generic CanHaveTypeAssigned and GetTypeId element methods.
Previously, the tool only supported displaying the type parameters for family instances.
Now, this functionality should work for all elements that can have a type.
The updated code of the external command Execute mainline below includes both the old family instance code and the implementation of the new generic functionality:
```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  // Select element:

  Element e
    = Util.GetSingleSelectedElementOrPrompt(
      uidoc );

  if( null == e )
  {
    return Result.Cancelled;
  }
  bool isSymbol = false;
  string family\_name = string.Empty;

  // For a family instance, ask user whether to
  // display instance or type parameters; in a
  // similar manner, we could add dedicated
  // switches for Wall --> WallType,
  // Floor --> FloorType etc. ...

  if( e is FamilyInstance )
  {
    FamilyInstance inst = e as FamilyInstance;
    if( null != inst.Symbol )
    {
      string symbol\_name = Util.ElementDescription(
        inst.Symbol, true );

      family\_name = Util.ElementDescription(
        inst.Symbol.Family, true );

      string msg = string.Format( \_type\_prompt,
        "is a family instance" );

      if( !Util.QuestionMsg( msg ) )
      {
        e = inst.Symbol;
        isSymbol = true;
      }
    }
  }
  else if( e.CanHaveTypeAssigned() )
  {
    ElementId typeId = e.GetTypeId();
    if( null == typeId )
    {
      Util.InfoMsg( "Element can have a type,"
        + " but the current type is null." );
    }
    else if( ElementId.InvalidElementId == typeId )
    {
      Util.InfoMsg( "Element can have a type,"
        + " but the current type id is the"
        + " invalid element id." );
    }
    else

    {
      Element type = doc.get\_Element( typeId );

      if( null == type )
      {
        Util.InfoMsg( "Element has a type,"
          + " but it cannot be accessed." );
      }
      else
      {
        string msg = string.Format( \_type\_prompt,
          "has an element type" );

        if( !Util.QuestionMsg( msg ) )
        {
          e = type;
          isSymbol = true;
        }
      }
    }
  }

  // Retrieve parameter data:

  SortableBindingList<ParameterData> data
    = new SortableBindingList<ParameterData>();

  {
    WaitCursor waitCursor = new WaitCursor();

    Array bips = Enum.GetValues(
      typeof( BuiltInParameter ) );

    int n = bips.Length;
    Parameter p;

    foreach( BuiltInParameter a in bips )
    {
      try
      {
        p = e.get\_Parameter( a );

        if( null != p )
        {
          string valueString =
            (StorageType.ElementId == p.StorageType)
              ? Util.GetParameterValue2( p, doc )
              : p.AsValueString();

          data.Add( new ParameterData( a, p,
            valueString ) );
        }
      }
      catch( Exception ex )
      {
        Debug.Print(
          "Exception retrieving built-in parameter {0}: {1}",
          a, ex );
      }
    }
  }

  // Display form:

  string description
    = Util.ElementDescription( e, true )
    + ( isSymbol
      ? " Type"
      : " Instance" );

  using( BuiltInParamsCheckerForm form
    = new BuiltInParamsCheckerForm(
      description, data ) )
  {
    form.ShowDialog();
  }
  return Result.Succeeded;
}
```

For the rest of the implementation, please refer to
[BipChecker01.zip](zip/BipChecker01.zip) containing
the full source code, add-in manifest, and Visual Studio solution for BipChecker version 2012.0.1.0.

**Response:** I understood that parameters may not appear in the Element.Parameters collection, and also how to get them via the Element.get\_Parameter method. I still do not understand why Autodesk developers make some parameters hidden from user.
I guess that these hidden parameters are system related and used in system methods.
Is that it?

I find it difficult to know whether parameter is present in the Element.Parameters collection or not.
I've found only one way to check it:
```csharp
bool IsParameterInCollection( Parameter parameter )
{
  foreach( Parameter p
    in parameter.Element.Parameters )
  {
    if( p.IsShared
      != \_parameter.IsShared )
    {
      return false;
    }

    if( ( p.Definition as
        InternalDefinition ).BuiltInParameter
      == ( \_parameter.Definition as
        InternalDefinition ).BuiltInParameter )
    {
      \_parameterInElementParametersCollection = true;
      return true;
    }
  }
  return false;
}
```

Regarding the second question on how to get the length of any element:

I agree with you that looking for parameter whose name contains 'Length' is a bad approach.

You say 'Better is to determine exactly which built-in parameter corresponds to it and use that instead'.
But how I can do it if I don't know anything about the element I am analysing?
In my case it may be absolutely any element.

I've found some parameters among BuiltInParameters that contains word 'LENGTH' but named not 'length'.
So, my other approach is not good either.

Now I think apply next algorithm to find length:

- Get all visible parameters of element. It's easy – Element.Parameters.- Check if (Parameter.Definition as InternalDefinition).BuiltInParameter contains 'LENGTH' substring.- Check if Parameter.Definition.ParameterGroup == PG\_GEOMETRY.- Profit if parameter exists :-)

I looked at your useful BipChecker tool and made some little changes.

I added a ParameterGroup field to the ParameterData class, and added a different parameter form with grouping using a ListView instead of DataGridView. Now you can see which parameters are provided by the Element.Parameters collection and which are retrieved via BuiltInParameter.
It seems more user friendly than in DataGridView.

I hope you like these changes.

Here is
[BipCheckerVc.zip](zip/BipCheckerVc.zip) containing
Victor's version using the list view display.

**Answer:** Great, I love the addition of the new properties for the parameter group, group name, and whether or not a parameter is included in the Element.Parameters collection or not.

Since the ParameterData class is getting a bit heavy now, I moved it out into a new module of its own.

I do not like the list view all that much, because it segregates the parameters into separate groups and prevents me from sorting them all in one collection by any one of the properties.
I therefore re-implemented the data grid view as well.
You can switch back and forth between the two views by setting a compile time pragma and rebuilding:
```csharp
#if USE\_LIST\_VIEW
  using( BuiltInParamsCheckerFormListView form
    = new BuiltInParamsCheckerFormListView(
      description, data ) )
#else
  using (BuiltInParamsCheckerForm form
    = new BuiltInParamsCheckerForm(
      description, data))
#endif // USE\_LIST\_VIEW
  {
    form.ShowDialog();
  }
```

I have not defined USE\_LIST\_VIEW in my code, and therefore the data grid view is used.
If you want to use the list view instead, simply add the following as the first line of the file:
```csharp
#define USE\_LIST\_VIEW
```

#### Determining Whether a Parameter is Contained in Element.Parameters

Yes, I see that it is indeed unexpectedly difficult to determine whether a given parameter retrieved from an element using the Parameter property is contained in the official Element.Parameters collection.

The ParameterSet returned by Element.Parameters does provide a Contains method, but it does not return the expected result.

For example, for a wall, which has a parameter named "Length" with the built-in parameter id CURVE\_ELEM\_LENGTH, the following code snippet will still return false:
```csharp
  wall.Parameters.Contains(
    wall.get\_Parameter(
      BuiltInParameter.CURVE\_ELEM\_LENGTH ) )
```

I used your code as a basis for implementing the following workaround:
```csharp
/// <summary>
/// Return BuiltInParameter id for a given parameter,
/// assuming it is a built-in parameter.
/// </summary>
static BuiltInParameter BipOf( Parameter p )
{
  return ( p.Definition as InternalDefinition )
    .BuiltInParameter;
}

/// <summary>
/// Check whether two given parameters represent
/// the same parameter, i.e. shared parameters
/// have the same GUID, others the same built-in
/// parameter id.
/// </summary>
static bool IsSameParameter( Parameter p, Parameter q )
{
  return( p.IsShared == q.IsShared )
    && ( p.IsShared
      ? p.GUID.Equals( q.GUID )
      : BipOf( p ) == BipOf( q ) );
}

/// <summary>
/// Return true if the given element parameter
/// retrieved by  get\_parameter( BuiltInParameter )
/// is contained in the element Parameters collection.
/// Workaround to replace ParameterSet.Contains.
/// Why does this not work?
/// return \_parameter.Element.Parameters.Contains(\_parameter);
/// </summary>
bool ContainedInCollection( Parameter p, ParameterSet set )
{
  bool rc = false;

  foreach( Parameter q in set )
  {
    rc = IsSameParameter( p, q );

    if( rc )
    {
      break;
    }
  }
  return rc;
}
```

Of course, as Victor pointed out, in the current context, one of the parameters will always be a built-in one obtained from the iteration over the BuiltInParameter enumeration, so the situation that both parameters are shared will never arise.

Furthermore, the comparison above can be vastly simplified by using the parameters Id property instead of the GUID or the built-in parameter id.

Making use of the Id property and letting LINQ perform the list processing operations for us in a generic fashion, all three methods above can be condensed into
```csharp
  bool ContainedInCollection(
    Parameter p,
    ParameterSet set )
  {
    return set
      .OfType<Parameter>()
      .Any( x => x.Id == p.Id );
  }
```

Here is
[BipChecker03.zip](zip/BipChecker03.zip) containing
the source code, add-in manifest, and Visual Studio solution for the updated BipChecker version 2012.0.3.0.

Here is what it looks like now displaying the parameters of a standard wall:

![BipChecker](img/BipChecker.png)

I hope you find this as useful as I do!

P.S. One thing that is possibly still missing here is support for shared parameters.
If you need that, add it, and let us know, please.