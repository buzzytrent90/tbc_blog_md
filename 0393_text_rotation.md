---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: code_example
optimization_date: '2025-12-11T11:44:13.872915'
original_url: https://thebuildingcoder.typepad.com/blog/0393_text_rotation.html
post_number: 0393
reading_time_minutes: 4
series: general
slug: text_rotation
source_file: 0393_text_rotation.htm
tags:
- elements
- family
- python
- revit-api
- transactions
- views
title: TextNote Rotation
word_count: 770
---

### TextNote Rotation

Although the Document and FamilyItemFactory classes provide a NewTextNote method which allows us to freely specify the rotation angle when creating new text note instances, there is currently no access to read the rotation of an existing TextNote element in the Revit API.
Exploring this issue in a little bit more depth discovered a possibility for a workaround, though, which I think worth while presenting.
It also prompted me to update The Building Coder sample command
[CmdBoundingBox](http://thebuildingcoder.typepad.com/blog/2008/10/element-bounding-box.html),
which displays the values of a selected building element bounding box.
Here is the question that prompted this update:

**Question:** I would like to transfer all text elements from a backup version of a project to a newer updated version.
It seems the using the clipboard to copy and paste aligned works nicely view by view, but there is no way to access this UI functions from the API. Since there are over a thousand views to process, doing it manually is not an option for us.
I am able to extract all the views and text notes in it programmatically, but I do not see how to obtain the rotational value of an existing text note. Is there any accessible property providing this data?

**Answer:** Unfortunately not, although we have registered a wish list item for this functionality. Here is a suggestion for a workaround, though:

From the TextNote element, you can access the following graphical properties:

- BoundingBox- Coord- Width

I created a sample model with the following text note:

![Rotated text note](img/textnote_rotated.png)

Extracting those three properties and creating a couple of simple geometric drawing primitives to represent them, I obtain the following:

![Text note graphical properties](img/textnote_rotated_properties.png)

In this drawing,

- BoundingBox is displayed as a box.- Coord is displayed as a circle.- Width is represented by the length of the line.

From this you can see that you can calculate an approximate rotation angle plus minus π from these three values, by determining the angle required to rotate the line around the Coord position to fit it inside the bounding box.

#### CmdBoundingBox Update

In preparing the images above, I noticed that my original implementation of The Building Coder sample command
[CmdBoundingBox](http://thebuildingcoder.typepad.com/blog/2008/10/element-bounding-box.html) examining
the bounding box of a Revit element was rather limited in functionality.

Querying the bounding box of a text note without specifying a view returns null.
In that case, the command would throw an exception.
Although it included a debugging assertion to check that a non-null value was returned, there was no runtime check to react to the null value and avoid dereferencing it.

Also, in this case, the null value returned when no view is specified should cause the command to try again with a valid view, which will return a valid bounding box in the case of a text note.

I therefore updated command in the following ways:

- If a null value is returned for the bounding box without a view specified, try again using the current view.- If a null value is still returned, print an error message in both release and debug mode.- List the results in a message box, not just in the Visual Studio debug output console.- Since the command makes no modifications to the database, change the transaction mode to read-only.

Here is the updated source code for the bounding box command:
```python
[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
class CmdBoundingBox : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    UIDocument uidoc = app.ActiveUIDocument;
    Document doc = uidoc.Document;

    Element e = Util.SelectSingleElement(
      uidoc, "an element" );

    if( null == e )
    {
      message = "No element selected";
      return Result.Failed;
    }

    View v = null;

    BoundingBoxXYZ b = e.get\_BoundingBox( v );

    if( null == b )
    {
      v = commandData.View;
      b = e.get\_BoundingBox( v );
    }

    if( null == b )
    {
      Util.InfoMsg(
        Util.ElementDescription( e )
        + " has no bounding box." );
    }
    else
    {
      Debug.Assert( b.Transform.IsIdentity,
        "expected identity bounding box transform" );

      string in\_view = ( null == v )
        ? "model space"
        : "view " + v.Name;

      Util.InfoMsg( string.Format(
        "Element bounding box of {0} in "
        + "{1} extends from {2} to {3}.",
        Util.ElementDescription( e ),
        in\_view,
        Util.PointString( b.Min ),
        Util.PointString( b.Max ) ) );
    }
    return Result.Succeeded;
  }
}
```

It now displays the following message when running it and selecting the text note shown above:

![Text note bounding box coordinates](img/textnote_bounding_box.png)

Here is
[version 2011.0.72.2](zip/bc_11_72_2.zip)
of The Building Coder samples including the complete source code and Visual Studio solution and the updated version of the command.