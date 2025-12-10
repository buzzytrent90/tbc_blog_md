---
post_number: "0844"
title: "Deleting a GraphicStyle Element"
slug: "delete_graphicstyle"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'python', 'revit-api', 'transactions']
source_file: "0844_delete_graphicstyle.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0844_delete_graphicstyle.html"
---

### Deleting a GraphicStyle Element

Here is a short and sweet deletion question handled by Joe Ye:

**Question:** I am trying to delete a GraphicStyle object in a RFA family document.

I tried to achieve this using the following code, but it was not successful:
```csharp
Dim elem As Element = m\_rvtDoc.Element(
"5ad9f0cf-6eff-4c63-8a44-9f3a87881dd7-00000b22")
m\_rvtDoc.Delete(elem)
```

How can I delete this object?

**Answer:** Some internal line style objects cannot be deleted, because the model requires their presence.
As you can see in the following image, the Delete and Rename buttons are disabled when Hidden Lines are selected:

![Protected line style objects](img/object_styles.png)

Some of the other line types, i.e. GraphicStyle objects, can very well be deleted.

I manually created an own line type in the dialogue shown above.
Since it is my own, the model does not depend on it.
I can then delete the line type I created using the following external command:
```python
[TransactionAttribute( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    Transaction trans = new Transaction( doc );
    trans.Start( "Delete Line Style" );

    //ElementId id = new ElementId( 4889 );
    //Element elem = doc.get\_Element( id );

    string guid = "6387d73b-1e94-456a-8804"
      + "-aaaf48a905f0-0000131a";

    Element elem = doc.get\_Element( guid );

    doc.Delete( elem );

    trans.Commit();

    return Result.Succeeded;
  }
}
```

By the way, deleting elements obviously requires an open transaction, which needs to be committed after the deletion.