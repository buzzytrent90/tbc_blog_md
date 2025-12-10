---
post_number: "0146"
title: "Host Reference"
slug: "host_reference"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'references', 'revit-api', 'transactions', 'walls', 'windows']
source_file: "0146_host_reference.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0146_host_reference.html"
---

### Host Reference

Still enjoying my climbing and sunshine holiday with my sons in Avignon.
Yesterday we went climbing and exploring the cliffs or 'calanque' of Cassis, east of Marseille
([more images](http://images.google.fr/images?hl=fr&q=calanque+de+cassis&um=1&ie=UTF-8&ei=-kkmSqO8KYq5jAe-q_3oBw&sa=X&oi=image_result_group&resnum=5&ct=title)):

![Calanque de Cassis](img/calanque-cassis.jpg)

Besides climbing, swimming in the Gard river and Mediterranean Sea, and blogging now and then, my sons are also artistically active:

![Dog by Cornelius](img/hund1.jpg)

Returning to the Revit API, however:
We already touched on the relationship between a host and its hosted elements in some previous posts.
The first such topic was the
[relationship inverter](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html),
making use of the explicit property providing a reference to the host of hosted elements such as doors, windows and other family instances.
Since this property only specifies the relationship in one direction, we implemented the relationship inverter to obtain the inverse references.

No such explicit property is provided in either direction in the
[association between a tag and the tagged element](http://thebuildingcoder.typepad.com/blog/2009/04/tag-association.html),
and that discussion mentions four ideas for possible workarounds instead.

Here is another question on a similar topic, the relationship between a wall and its wall footing:

**Question:**
How can I obtain a reference to a wall footing from the wall attached to it?
And vice versa, is it possible to obtain a reference to the wall supported by a wall footing?

![Wall footing](img/wall_footing.png)

**Answer:**
As in the case of the tag and the tagged element, the Revit API does not provide any direct access to the link in either direction,
neither to get the reference of a wall footing from the wall attached to it or vice versa.
The workaround via the Document.Delete method mentioned as one of the four workarounds to determine the association between
[a tag and the tagged element](http://thebuildingcoder.typepad.com/blog/2009/04/tag-association.html)
can be used in this case as well:
You call the method to delete the wall.
It automatically includes the wall footing in the deletion, and returns the element ids of all deleted objects in an ElementIdSet, from which you can pick out the wall footing.
If you encapsulate this operation in a transaction which is aborted after the call, the deletion will not be committed, and you will have obtained the desired information with no changes made to the model.
Here is a sample code snippet to implement this:

I added a new command class CmdWallFooting to The Building Coder sample implementing this functionality.
Here is the source code of its Execute method:

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

ContFooting footing = null;

Wall wall = Util.SelectSingleElementOfType(
  doc, typeof( Wall ), "a wall" ) as Wall;

if ( null == wall )
{
  message = "Please select a single wall element.";
}
else
{
  doc.BeginTransaction();
  ElementIdSet delIds = null;
  try
  {
    delIds = doc.Delete( wall );
  }
  catch ( System.Exception )
  {
    message = "Deletion failed.";
    doc.AbortTransaction();
    return CmdResult.Failed;
  }
  doc.AbortTransaction();

  foreach ( ElementId id in delIds )
  {
    ElementId refId = id;
    Element elem = doc.get\_Element( ref refId );
    if ( null == elem )
      continue;
    footing = elem as ContFooting;
    if ( null != footing )
      break;
  }
}
string s = Util.ElementDescription( wall );

Util.InfoMsg( ( null == footing )
  ? string.Format( "No footing found for {0}.", s )
  : string.Format( "{0} has {1}.", s,
    Util.ElementDescription( footing ) ) );

return CmdResult.Failed;
```

This is the result of running the new command on the wall depicted above:

![Wall footing message](img/wall_footing_message.png)

Another efficient workaround has become possible in Revit 2010 by making use of the new API method **FindReferencesByDirection**.
It finds both elements and geometric references that intersect a ray extending in a certain direction from an origin point.
The Revit 2010 SDK Samples AvoidObstruction and RaytraceBounce demonstrate the use of this new method.
By using a couple of points on the wall footing, we can easily determine which wall is located directly above it, and vice versa as well.
This provides a much stronger possibility to the workarounds using general geometric proximity suggested in the post on the
[tag association](http://thebuildingcoder.typepad.com/blog/2009/04/tag-association.html).

Here is
[version 1.1.0.34](zip/bc11034.zip)
of the complete Visual Studio solution with the new command.

**Update:** I updated this article for Revit 2011 on August 3 2010 based on the comment below by Jo Lee to fix the
[transaction migration errors](http://thebuildingcoder.typepad.com/blog/2010/08/transaction-migration-errors.html) introduced
by the simplistic initial flat port of the code from Revit 2010 to the Revit 2011 API.