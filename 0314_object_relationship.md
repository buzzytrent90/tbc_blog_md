---
post_number: "0314"
title: "Object Relationships"
slug: "object_relationship"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'references', 'revit-api', 'sheets', 'transactions', 'views', 'walls']
source_file: "0314_object_relationship.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0314_object_relationship.html"
---

### Object Relationships

We already discussed several uses of the Document.Delete method to discover object relationships, e.g. the relationship between a
[tag and the tagged element](http://thebuildingcoder.typepad.com/blog/2009/04/tag-association.html), a
[wall and its footing](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html),
[host and its hosted elements](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html), the
[title block of a sheet](http://thebuildingcoder.typepad.com/blog/2009/11/title-block-of-sheet.html), or
host elements and their openings for the purposes of
[gross material quantity determination](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html).
Another use of it is to obtain the
[unmodified element geometry](http://thebuildingcoder.typepad.com/blog/2010/02/unmodified-element-geometry.html).

Here is a very neat little sample named ObjRel by Saikat Bhattacharya that shows how to use the method to determine generic objects relationships in a building model and display the results in a tree view.
The tree view is hosted by a .NET form, implemented by a class named Result.

The really neat trick of Saikat's implementation lies in the helper method GetDependentsElementIds and its three lines of code that determine all object relationships in a generic fashion by creating a transaction, calling the Delete method to obtain the dependent element ids, and aborting the transaction to restore the original state:
```csharp
ElementIdSet GetDependentsElementIds(
  Element e )
{
  Document doc = \_app.ActiveDocument;

  doc.BeginTransaction();

  ElementIdSet ids = doc.Delete( e );

  doc.AbortTransaction();

  return ids;
}
```

This method is used by the two tree view helper methods DisplayNode and CreateRelationships to populate the tree view with all BIM elements having relationships with other objects in the model:

- DisplayNode:
  This recursive method creates the tree nodes
  based on dependent element ids. Each node
  displays its element name, category and id.- CreateRelationships :
    Creates the tree node with the initial node,
    populates it, and maintains a list of elements
    already included in the nodes to avoid duplication.

Here is the implementation of these two and the tree view dialogue constructor calling them:
```csharp
void DisplayNode( Element e, TreeNode node )
{
  string cat = (null == e.Category)
    ? "<category unknown>"
    : e.Category.Name;

  string label = string.Format( "{0}: {1} {2}",
    e.Name, cat, e.Id.Value );

  TreeNode father = node.Nodes.Add( label );

  // save element id to handle select event,
  // cf. treeView1\_AfterSelect:

  father.ImageKey = e.Id.Value.ToString();

  ElementIdSet ids = GetDependentsElementIds( e );

  try
  {
    if( null != ids && 1 < ids.Size )
    {
      Document doc = \_app.ActiveDocument;

      foreach( ElementId id1 in ids )
      {
        ElementId id = id1;
        Element e2 = doc.get\_Element( ref id );
        if( e2 != null )
        {
          if( !e2.Id.Equals( e.Id )
            && !\_displayedElems.Contains( e2 ) )
          {
            \_displayedElems.Insert( e2 );
            DisplayNode( e2, father );
          }
        }
      }
    }
  }
  catch( Exception ex )
  {
    MessageBox.Show( ex.Message.ToString() );
  }
}

void CreateRelationships()
{
  rootNode = new TreeNode( \_app.ActiveDocument.Title );
  this.treeView1.Nodes.Add( rootNode );

  \_displayedElems = new ElementSet();

  foreach( Element e in \_elems )
  {
    if( !\_displayedElems.Contains( e ) )
    {
      DisplayNode( e, rootNode );
    }
  }
}

public Result( ElementSet elems, Autodesk.Revit.Application app )
{
  \_elems = elems;
  \_app = app;

  InitializeComponent();
  CreateRelationships();

  treeView1.AfterSelect
    += new TreeViewEventHandler(
      treeView1\_AfterSelect );
}
```

Here is the external command Execute mainline implementation.
It creates a set of all model elements with visible graphics and representing a valid physical part of the building model and sends it the Result dialogue constructor, which calls the CreateRelationships method which in turn uses DisplayNode and GetDependentsElementIds to determine and display all dependencies:
```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

Autodesk.Revit.Geometry.Options opt
  = app.Create.NewGeometryOptions();

BuiltInCategory bicPreviewLegendComponent
  = BuiltInCategory.OST\_PreviewLegendComponents;

int iBic = ( int ) bicPreviewLegendComponent;

try
{
  // select all model elements:

  ElementSet a = app.Create.NewElementSet();

  ElementIterator it = app.ActiveDocument.Elements;

  while( it.MoveNext() )
  {
    Element e = it.Current as Element;

    if( !( e is Symbol )
      && !( e is FamilyBase )
      && ( null != e.Category )
      && ( iBic != e.Category.Id.Value )
      && ( null != e.get\_Geometry( opt ) ) )
    {
      a.Insert( e );
    }
  }

  // show the object relationship dialog

  Result res = new Result( a, app );
  res.ShowDialog();
}
catch( Exception ex )
{
  message = ex.Message;
}
return IExternalCommand.Result.Failed;
```

We can use the following simple model to see the relationships displayed by this tool.
Here is a 3D view showing the physical building model:

![Simple house 3D view](img/SimpleHouse3d.png)

The model also includes a few annotation elements which are only visible in plan view:

![Simple house plan view](img/SimpleHousePlan.png)

Here is the result of running the ObjRel command on this model and then fully expanding the tree view nodes to display all of the nested relationships detected:

![Simple house object relationships](img/SimpleHouseObjRel.png)

As you can see, a number of relationships between various objects have been determined and are displayed, and many of the types of relationships are ones which we had not previously explicitly noted.

Here is the complete
[ObjRel](zip/ObjRel.zip)
source code and Visual Studio solution including the sample model we used.

Many thanks to Saikat for this brilliant idea of making such generic use of the Delete method to determine all object relationships and the neat little sample to demonstrate its use so succinctly and powerfully!