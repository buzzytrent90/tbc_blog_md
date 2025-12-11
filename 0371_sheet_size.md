---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: code_example
optimization_date: '2025-12-11T11:44:13.834351'
original_url: https://thebuildingcoder.typepad.com/blog/0371_sheet_size.html
post_number: '0371'
reading_time_minutes: 7
series: views
slug: sheet_size
source_file: 0371_sheet_size.htm
tags:
- elements
- family
- filtering
- parameters
- python
- references
- revit-api
- sheets
- transactions
- views
title: Determine Sheet Size
word_count: 1398
---

### Determine Sheet Size

We already discussed some issues related to sheets and title blocks, such as the
[relationship between viewports and sheets](http://thebuildingcoder.typepad.com/blog/2009/01/viewports-and-sheets.html) and how to
[retrieve the title block of a sheet](http://thebuildingcoder.typepad.com/blog/2009/11/title-block-of-sheet.html).

Here is a question on determining the size of a sheet which provides an opportunity to update these explorations based on the Revit 2011 platform.

**Question:** Using the pre-release 2011 RevitAPI.dll I was able to detect the sheet width/height by iterating the ParameterSet collection for each element in the ViewSheet.ElementSet object. I was able to locate the "Sheet Height" and "Sheet Width" definitions.

With the new Revit 2011 release there are some major changes in the API. Could you please briefly explain how I can get the sheet's width/height with the new API?

**Answer:** The approach you describe sounds rather inefficient.
Do you mean that you iterated over all the elements in the ViewSheet.ElementSet, and for each element iterated over all its parameters?

I would suggest that you enhance that, even if it did still work.
There is no need to iterate over all parameters, since you can access the specific parameters you want, either using the built-in parameter enumeration or their display name. Using the enumeration is much better, of course, since it is language independent.

Similarly, there is no need to iterate over multiple elements, since you can determine exactly which elements you need to access to obtain the data for a specific sheet.

As far as I can tell, the sheet itself does not have any information about its width and height. On the other hand, though, each sheet seems to have exactly one title block instance assigned to it, and that does include information on the sheet width and height.

I analysed a sample project using the Revit API Introduction labs built-in parameter checker and was unable to find any width and height data on the sheet itself. On the other hand, I did find the built-in parameters SHEET\_WIDTH and SHEET\_HEIGHT on the title block of the sheet.

The document also has a collection of all loaded title block types which is accessible through the TitleBlocks property. These types do not list the sheet size themselves. Instead, they are referenced by the title block instances which do.

The relationship between sheets and their associated title blocks can be determined through the sheet number, which is available on both of these elements.

I implemented a new Building Coder sample command CmdSheetSize which illustrates some relationships between the title block, sheet and title block types.
It includes source code to access the sheet size for all title blocks.
For a given sheet number, you can locate the associated title block and retrieve the sheet size from that.
Here are the steps it performs:

- Iterate over the document TitleBlocks collection and lists some data for each title block element type.- Use a filtered element collector to retrieve and list the same elements.- Retrieves all title block and view sheet instances in the model.- List the sheet size defined by the title block instance.- Display the one-to-on correspondence between title block and view sheet instances, linked through the sheet number.

```python
[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
class CmdSheetSize : IExternalCommand
{
  ///
  /// Return a string value for the specified
  /// built-in parameter if it is available on
  /// the given element, else an empty string.
  ///
  string GetParameterValueString(
    Element e,
    BuiltInParameter bip )
  {
    Parameter p = e.get\_Parameter( bip );

    string s = string.Empty;

    if( null != p )
    {
      switch( p.StorageType )
      {
        case StorageType.Integer:
          s = p.AsInteger().ToString();
          break;

        case StorageType.ElementId:
          s = p.AsElementId().IntegerValue.ToString();
          break;

        case StorageType.Double:
          s = Util.RealString( p.AsDouble() );
          break;

        case StorageType.String:
          s = string.Format( "{0} ({1})",
            p.AsValueString(),
            Util.RealString( p.AsDouble() ) );
          break;

        default: s = "";
          break;
      }
      s = ", " + bip.ToString() + "=" + s;
    }
    return s;
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref String message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    FilteredElementCollector a;
    Parameter p;
    string s;

    int n = doc.TitleBlocks.Size;

    Debug.Print(
      "{0} title block element type{1} listed "
      + "in doc.TitleBlocks collection{2}",
      n,
      ( 1 == n ? "" : "s" ),
      ( 0 == n ? "." : ":" ) );

    foreach( FamilySymbol tb in doc.TitleBlocks )
    {
      // these are the family symbols,
      // i.e. the title block element types,
      // i.e. not instances, i.e. not sheets,
      // and they obviously do not have any sheet
      // number, width or height, so 's' ends up empty:

      s = GetParameterValueString( tb, BuiltInParameter.SHEET\_NUMBER )
        + GetParameterValueString( tb, BuiltInParameter.SHEET\_WIDTH )
        + GetParameterValueString( tb, BuiltInParameter.SHEET\_HEIGHT );

      Debug.Print(
        "Title block element type {0} {1}" + s,
        tb.Name, tb.Id.IntegerValue );
    }

    // using this filter returns the same elements
    // as the doc.TitleBlocks collection:

    a = new FilteredElementCollector( doc );
    a.OfCategory( BuiltInCategory.OST\_TitleBlocks );
    a.OfClass( typeof( FamilySymbol ) );

    Debug.Print( "{0} title block element type{1} "
      + "retrieved by filtered element collector{2}",
      n,
      ( 1 == n ? "" : "s" ),
      ( 0 == n ? "." : ":" ) );

    foreach( FamilySymbol symbol in a )
    {
      Debug.Print(
        "Title block element type {0} {1}",
        symbol.Name, symbol.Id.IntegerValue );
    }

    // retrieve the title block instances:

    a = new FilteredElementCollector( doc );
    a.OfCategory( BuiltInCategory.OST\_TitleBlocks );
    a.OfClass( typeof( FamilyInstance ) );

    Debug.Print( "Title block instances:" );

    foreach( FamilyInstance e in a )
    {
      p = e.get\_Parameter(
        BuiltInParameter.SHEET\_NUMBER );

      Debug.Assert( null != p,
        "expected valid sheet number" );

      string sheet\_number = p.AsString();

      p = e.get\_Parameter(
        BuiltInParameter.SHEET\_WIDTH );

      Debug.Assert( null != p,
        "expected valid sheet width" );

      string swidth = p.AsValueString();
      double width = p.AsDouble();

      p = e.get\_Parameter(
        BuiltInParameter.SHEET\_HEIGHT );

      Debug.Assert( null != p,
        "expected valid sheet height" );

      string sheight = p.AsValueString();
      double height = p.AsDouble();

      ElementId typeId = e.GetTypeId();
      Element type = doc.get\_Element( typeId );

      Debug.Print(
        "Sheet number {0} size is {1} x {2} "
        + "({3} x {4}), id {5}, type {6} {7}",
        sheet\_number, swidth, sheight,
        Util.RealString( width ),
        Util.RealString( height ),
        e.Id.IntegerValue,
        type.Name, typeId.IntegerValue );
    }

    // retrieve the view sheet instances:

    a = new FilteredElementCollector( doc );
    a.OfClass( typeof( ViewSheet ) );

    Debug.Print( "View sheet instances:" );

    foreach( ViewSheet vs in a )
    {
      string number = vs.SheetNumber;
      Debug.Print(
        "View sheet name {0} number {1} id {2}",
        vs.Name, vs.SheetNumber,
        vs.Id.IntegerValue );
    }
    return Result.Succeeded;
  }
}
```

As said, it iterates over the document TitleBlocks collection and lists some data for each title block element type, then shows that the same elements are also available through the filtered element collector.

It then retrieves the title block and the view sheet instances and shows that there is a one-to-on correspondence between them, linked through the sheet number.

Here is the output from this command in my sample project:

```
2 title block element types listed in doc.TitleBlocks collection:
Title block element type A0 metric 28295
Title block element type A4 metric 129886
2 title block element types retrieved by filtered element collector:
Title block element type A0 metric 28295
Title block element type A4 metric 129886
Title block instances:
Sheet number A101 size is 1188.0 x 840.0 (3.9 x 2.76), id 127154, type A0 metric 28295
Sheet number A102 size is 1188.0 x 840.0 (3.9 x 2.76), id 127168, type A0 metric 28295
Sheet number A103 size is 1188.0 x 840.0 (3.9 x 2.76), id 127182, type A0 metric 28295
Sheet number A104 size is 210.0 x 297.0 (0.69 x 0.97), id 129892, type A4 metric 129886
View sheet instances:
View sheet name Unnamed number A101 id 127148
View sheet name Unnamed number A102 id 127162
View sheet name Unnamed number A103 id 127176
View sheet name Unnamed number A104 id 129888
```

If you have a need for a direct mapping between the view sheets and the title block sizes, you could obviously use this information to create a bidirectional mapping as described for the
[relationship inverter](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html).
Then, for a given sheet 's', you might be able to use something like 'title\_block[s.Number].Width/Height'.

Another little snippet of source code demonstrating these relationships is given by the sample code in the Revit API help file RevitAPI.chm in the description of the Document.NewViewSheet method.

Here is
[version 2011.0.68.0](zip/bc_11_68.zip)
of The Building Coder sample source code and Visual Studio solution including the new command.