---
post_number: "1027"
title: "Max' Revision Wrapper Class"
slug: "revision"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api']
source_file: "1027_revision.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1027_revision.html"
---

### Max' Revision Wrapper Class

Max posted a
[comment](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html?cid=6a00e553e168978833019aff7c6098970c#comment-6a00e553e168978833019aff7c6098970c) on
[what's new in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) presenting
a neat little revision parameter wrapper class that I have taken the liberty of adding to The Building Coder sample collection.

In Max' own words:
For everyone also looking for the Revision parameters:
Here a little wrapper class in C#:

```csharp
using System;
using Autodesk.Revit.DB;

namespace BuildingCoder
{
  /// <summary>
  /// A Revision parameter wrapper class by Max.
  /// </summary>
  class JtRevision
  {
    /// <summary>
    /// The BIM element.
    /// </summary>
    Element \_e;

    /// <summary>
    /// Internal access to the named parameter.
    /// </summary>
    Parameter \_p( string parameter\_name )
    {
      return \_e.get\_Parameter( parameter\_name );
    }

    /// <summary>
    /// Create a Revision parameter accessor
    /// for the given BIM element.
    /// </summary>
    public JtRevision( Element e )
    {
      \_e = e;
    }

    public string Date
    {
      get { return \_p( "Revision Date" ).AsString(); }
      set { \_p( "Revision Date" ).Set( value ); }
    }

    public string IssuedTo
    {
      get { return \_p( "Issued to" ).AsString(); }
      set { \_p( "Issued to" ).Set( value ); }
    }

    public string Number
    {
      get { return \_p( "Revision Number" ).AsString(); }
      set { \_p( "Revision Number" ).Set( value ); }
    }

    public int Issued
    {
      get { return \_p( "Issued" ).AsInteger(); }
      set { \_p( "Issued" ).Set( value ); }
    }

    public int Numbering
    {
      get { return \_p( "Numbering" ).AsInteger(); }
      set { \_p( "Numbering" ).Set( value ); }
    }

    public int Sequence
    {
      get { return \_p( "Revision Sequence" ).AsInteger(); }
      set { \_p( "Revision Sequence" ).Set( value ); }
    }

    public string Description
    {
      get { return \_p( "Revision Description" ).AsString(); }
      set { \_p( "Revision Description" ).Set( value ); }
    }

    public string IssuedBy
    {
      get { return \_p( "Issued by" ).AsString(); }
      set { \_p( "Issued by" ).Set( value ); }
    }
  }
}
```

As you can see for yourself, there is no magic to this, and the principle could be used for many other purposes.

In fact, all this does is hard code a mapping of parameter names to wrapper class methods.

Mille grazie, Max, for providing this nice implementation!

I wish everybody a nice weekend!

### Addendum

Thorsten Meinecke pointed out the importance of avoiding the use of display names to identify the element parameters in his comment below.

Three years later, Jose Ignacio Montes was kind enough to actually do the work and re-implement this class using that approach:

```csharp
class JtRevision
{
  /// <summary>
  /// The BIM element.
  /// </summary>
  Element \_e;
  /// <summary>
  /// Internal access to the named parameter.
  /// </summary>
  Parameter \_p( BuiltInParameter bip )
  {
    return \_e.get\_Parameter( bip );
  }
  /// <summary>
  /// Create a Revision parameter accessor
  /// for the given BIM element.
  /// </summary>
  public JtRevision( Element e )
  {
    \_e = e;
  }
  public string Date
  {
    get { return \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_DATE ).AsString(); }
    set { \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_DATE ).Set( value ); }
  }
  public string IssuedTo
  {
    get { return \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_ISSUED\_TO ).AsString(); }
    set { \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_ISSUED\_TO ).Set( value ); }
  }
  public string Number
  {
    get { return \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_NUM ).AsString(); }
    set { \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_NUM ).Set( value ); }
  }
  public int Issued
  {
    get { return \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_ISSUED ).AsInteger(); }
    set { \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_ISSUED ).Set( value ); }
  }
  public int Numbering
  {
    get { return \_p( BuiltInParameter.PROJECT\_REVISION\_ENUMERATION ).AsInteger(); }
    set { \_p( BuiltInParameter.PROJECT\_REVISION\_ENUMERATION ).Set( value ); }
  }
  public int Sequence
  {
    get { return \_p( BuiltInParameter.PROJECT\_REVISION\_SEQUENCE\_NUM ).AsInteger(); }
    set { \_p( BuiltInParameter.PROJECT\_REVISION\_SEQUENCE\_NUM ).Set( value ); }
  }
  public string Description
  {
    get { return \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_DESCRIPTION ).AsString(); }
    set { \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_DESCRIPTION ).Set( value ); }
  }
  public string IssuedBy
  {
    get { return \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_ISSUED\_BY ).AsString(); }
    set { \_p( BuiltInParameter.PROJECT\_REVISION\_REVISION\_ISSUED\_BY ).Set( value ); }
  }
}
```

Many thanks to Jose Ignacio for this very cool and overdue improvement!

I added the updated version to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).

The new class lives beside the obsolete old one in the module [JtRevision.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/JtRevision.cs).