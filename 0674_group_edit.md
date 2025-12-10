---
post_number: "0674"
title: "Editing a Group Take Two"
slug: "group_edit"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'python', 'revit-api', 'selection', 'transactions']
source_file: "0674_group_edit.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0674_group_edit.html"
---

### Editing a Group Take Two

I am publishing this now, on November 11, 2011, at 11:11 my time, to celebrate this unique and yet completely arbitrary time and date in our calendar history.
I hope you appreciate it and celebrated that moment in your own way as well :-)

I discussed various aspects of programmatically handling Revit element groups in the past, such as how to
[create](http://thebuildingcoder.typepad.com/blog/2009/02/creating-a-group-and-how-to-fish.html),
[rename](http://thebuildingcoder.typepad.com/blog/2009/06/rename-a-group.html),
[edit](http://thebuildingcoder.typepad.com/blog/2010/08/editing-elements-inside-groups.html) and
[delete](http://thebuildingcoder.typepad.com/blog/2011/09/deleting-a-group.html) them.

In a
[comment](http://thebuildingcoder.typepad.com/blog/2010/08/editing-elements-inside-groups.html?cid=6a00e553e1689788330162fbe9f033970d#comment-6a00e553e1689788330162fbe9f033970d) on
the discussion on editing groups, YarUnderoaker asked for further clarification on how to implement the ungrouping and recreation of a group to make changes to it:

**Question:** I cannot figure out how to do as stated in the recommendations:
"You can programmatically ungroup, make the change, regroup and then swap the other instances of the old group to the new group to get the same effect."

Please, can you give a simple example?

**Answer:** My colleague Saikat Bhattacharya put together some sample code to answer this for you.
It performs the following steps:

- Prompt user to select a group.- Ungroup it.- Create an element set of all its members except the first.- Create a new group using the element set.- Change the new group type to the old group type.

Here is the complete code to implement the mainline Execute method of the external command:
```python
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    // Prompt user to select existing group

    Group grpExisting = doc.GetElement(
      uidoc.Selection.PickObject( ObjectType.Element,
        "Select an existing group" ) ) as Group;

    string name = grpExisting.Name;

    // Ungroup the group

    Transaction tx = new Transaction( doc );
    tx.Start( "Ungroup and delete" );

    ElementSet grpElements = grpExisting.Ungroup();

    // Create element set for new group

    ElementSet newgrpElements = new ElementSet();

    int counter = 0;

    foreach( Element e in grpElements )
    {
      if( 0 == counter )
      {
        // Delete the first group element

        doc.Delete( e );
      }
      else
      {
        newgrpElements.Insert( e );
      }
      ++counter;
    }

    tx.Commit();

    // Create new group

    tx.Start( "Group" );

    Group grpNew = doc.Create.NewGroup(
      newgrpElements );

    // Access the name of the previous group type
    // and change the new group type to previous
    // group type to retain the previous group
    // configuration

    FilteredElementCollector coll
      = new FilteredElementCollector( doc )
        .OfClass( typeof( GroupType ) );

    IEnumerable<GroupType> grpTypes
      = from GroupType g in coll
        where g.Name == name select g;

    grpNew.GroupType = grpTypes.First<GroupType>();

    tx.Commit();

    return Result.Succeeded;
  }
}
```

I hope this clarifies the process.

Many thanks to Saikat for sharing this!