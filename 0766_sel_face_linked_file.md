---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: code_example
optimization_date: '2025-12-11T11:44:14.550321'
original_url: https://thebuildingcoder.typepad.com/blog/0766_sel_face_linked_file.html
post_number: '0766'
reading_time_minutes: 5
series: general
slug: sel_face_linked_file
source_file: 0766_sel_face_linked_file.htm
tags:
- csharp
- elements
- filtering
- geometry
- references
- revit-api
- selection
- transactions
- walls
title: Selecting a Face in a Linked File
word_count: 946
---

### Selecting a Face in a Linked File

Here is a chance to look at an interesting method that we never discussed yet, ConvertToStableRepresentation, and the hidden information that it provides access to.
It does what it says, converting a Reference to a stable string representation.

#### Reference and Stable Representation

A reference provides a possibility to identify a piece of geometry, even though geometry is transient, not persistent, memory only, generated on the fly.
Still, we sometimes need to identify a piece of it and remember which piece it was.
For instance, to
[dimension the distance](http://thebuildingcoder.typepad.com/blog/2011/02/dimension-walls-by-iterating-faces.html)
[between two parallel walls](http://thebuildingcoder.typepad.com/blog/2011/02/dimension-walls-using-findreferencesbydirection.html),
we need to identify which wall face we are measuring from.
This can be achieved using references.

A stable representation can be used to preserve and restore a reference later in the same Revit session or even in a different session in the same document.
The ParseFromStableRepresentation method is used to restore the reference.
The representation is based on the internal Revit structure and is not intended to be parsed by anyone else except ParseFromStableRepresentation.

Here is a rather unexpected use of this method and the undocumented internals of the stable string representation:

#### Face Selection in Linked File

Some Revit API functionality is limited to the current project and will not work for linked files.
Currently, this most heavily affects the geometric analysis, and methods like FindReferencesWithContextByDirection and its new optimised and simplified Revit 2013 wrapper class ReferenceIntersector.

Another affected area is the interactive element selection, which led to the following
[question](http://thebuildingcoder.typepad.com/blog/2012/04/revit-2013-product-guids-and-guid-algorithm.html?cid=6a00e553e1689788330168ea336953970c#comment-6a00e553e1689788330168ea336953970c) by
[Valentin Louzeau](mailto:vlouzeau@gmail.com):

**Question:** I'm working on a plugin and asking the user to select a face, but it does not work in linked files.
I found something using ObjectType.PointOnElement for selection but it doesn't seem to work for my needs.
Is there an easy way to do that or do you already have a solution?

**Answer:** Nope, sorry, there is no easy way to work on a face in a linked file, and I am not aware of any solution for this.

**Response:** I found a solution using the method ConvertToStableRepresentation on the Reference class.

This string contains the ref document unique ID, its name, and the ID of the picked element in this document.
You can get its position in the ref doc and if you insert origin to origin, the element has the same position in the two documents.

Here is an example of a stable representation for a reference to a picked face on a wall in a linked file:

- "7c25d827-3c9c-4cca-b3a0-dd28ee09a289-0002e3a1:0:RVTLINK/7c25d827-3c9c-4cca-b3a0-dd28ee09a289-0002e3a0:161224:5:SURFACE"

From this, the wall element id "161224" can be extracted.

Here is some code in which I tested making use of this:
```csharp
[TransactionAttribute( TransactionMode.Manual )]
public class SelectFaceInLinkedFile : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;

    PlanarFace Plan = SelectFace( uiapp );

    return Result.Succeeded;
  }

  public static PlanarFace SelectFace(
    UIApplication uiapp )
  {
    Document doc = uiapp.ActiveUIDocument.Document;

    // get all ref doc.

    IEnumerable<Document> doc2
      = GetLinkedDocuments( doc );

    // get the ref of a selected plane

    Selection sel = uiapp.ActiveUIDocument.Selection;

    Reference pickedRef = sel.PickObject(
      ObjectType.PointOnElement,
      "Please select a Face" );

    Element elem = doc.GetElement(
      pickedRef.ElementId );

    // get the true position picked
    // in the active document

    XYZ pos = pickedRef.GlobalPoint;

    // get the ID of the element containing the
    // face you picked in the active document
    // and in its host document

    string s = pickedRef
      .ConvertToStableRepresentation( doc );

    string[] tab\_str = s.Split( ':' );

    string id = tab\_str[tab\_str.Length - 3];

    int ID;
    Int32.TryParse( id, out ID );

    Type et = elem.GetType();

    if( typeof( RevitLinkType ) == et
      || typeof( RevitLinkInstance ) == et
      || typeof( Instance ) == et )
    {
      foreach( Document d in doc2 )
      {
        if( elem.Name.Contains( d.Title ) )
        {
          Element element = d.GetElement(
            new ElementId( ID ) );

          Options ops = new Options();
          ops.ComputeReferences = true;

          // write the name of the element and the
          // number of solids in this only for
          // control to show the possibilities

          MessageBox.Show( element.Name,
            element.get\_Geometry( ops )
              .Objects.Size.ToString() );

          GeometryObject obj
            = element.get\_Geometry( ops )
              .Objects.get\_Item( 0 );

          // test all surfaces of solids in the
          // element and return the one containing
          // the picked point as a planarface to
          // build my sketchplan

          foreach( GeometryObject obj2 in
            element.get\_Geometry( ops ).Objects )
          {
            if( obj2.GetType() == typeof( Solid ) )
            {
              Solid solid2 = obj2 as Solid;
              foreach( Face face2 in solid2.Faces )
              {
                try
                {
                  if( face2.Project( pos )
                    .XYZPoint.DistanceTo( pos ) == 0 )
                  {
                    return face2 as PlanarFace;
                  }
                }
                catch( NullReferenceException )
                {
                }
              }
            }
          }
        }
      }
    }
    return null;
  }

  // this part is not mine, i found it on the internet,
  // i don't anderstand all the code

  public static IEnumerable<ExternalFileReference>
    GetLinkedFileReferences( Document \_document )
  {
    var collector = new FilteredElementCollector(
      \_document );

    var linkedElements = collector
      .OfClass( typeof( RevitLinkType ) )
      .Select( x => x.GetExternalFileReference() )
      .ToList();

    return linkedElements;
  }

  public static IEnumerable<Document>
    GetLinkedDocuments( Document \_document )
  {
    var linkedfiles = GetLinkedFileReferences(
      \_document );

    var linkedFileNames = linkedfiles
      .Select( x => ModelPathUtils
        .ConvertModelPathToUserVisiblePath(
          x.GetAbsolutePath() ) ).ToList();

    return \_document.Application.Documents
      .Cast<Document>()
      .Where( doc => linkedFileNames.Any(
        fileName => doc.PathName.Equals( fileName ) ) );
  }
}
```

Here is
[SelectFaceInLinkedFile.zip](zip/SelectFaceInLinkedFile.zip) containing
Valentin's sample code and Visual Studio solution.

Many thanks to Valentin for the novel use of this method and its undocumented embedded information.

This is obviously all undocumented internal stuff that is not guaranteed to work at all in any way, and actually is guaranteed to change at some point in the future (as everything must change) with no prior warning whatsoever, so use at your own risk or just ponder and enjoy.