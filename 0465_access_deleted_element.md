---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.995076'
original_url: https://thebuildingcoder.typepad.com/blog/0465_access_deleted_element.html
post_number: '0465'
reading_time_minutes: 3
series: elements
slug: access_deleted_element
source_file: 0465_access_deleted_element.htm
tags:
- csharp
- elements
- filtering
- levels
- revit-api
- transactions
title: Access Deleted Element
word_count: 668
---

### Access Deleted Element

Here is a
[question](http://thebuildingcoder.typepad.com/blog/2010/10/power-to-the-user-and-application.html?cid=6a00e553e1689788330133f5511cc1970b#comment-6a00e553e1689788330133f5511cc1970b) posted
by Kristian (Krispy5) Parsons, Systems Analyst - Design at
[Westfield Design & Construction Pty. Ltd.](http://www.westfield.com) on
accessing deleted element data in the DocumentChanged event handler:

**Question:** How do I find the category of the deleted elements?
When I try to get the Element from the ElementId it fails since the element has already been deleted.

In my case I would like to warn users when they have deleted a floor.

**Answer:** I was planning to test your assertion by looking at the Revit SDK ChangesMonitor sample which demonstrates the use of the DocumentChanged method.

I assume that your question is based on using that event to be notified of the deleted elements. Looking at the ChangesMonitor DocumentChanged event handler, I note that it calls a helper method AddChangeInfoRow to list the added, deleted and modified element information in its modeless form. That method includes the following statements and comments which clearly tell you that the deleted element information is no longer accessible:
```csharp
private void AddChangeInfoRow(
  ElementId id,
  Document doc,
  string changeType )
{
  // retrieve the changed element
  Element elem = doc.get\_Element( id );

  DataRow newRow = m\_ChangesInfoTable.NewRow();

  // set the relative information of
  // this event into the table.

  if( elem == null )
  {
    // this branch is for deleted element due
    // to the deleted element cannot be retrieved
    // from the document.

    newRow["ChangeType"] = changeType;
    newRow["Id"] = id.IntegerValue.ToString();
    newRow["Name"] = "";
    newRow["Category"] = "";
    newRow["Document"] = "";
  }
  else
  {
    newRow["ChangeType"] = changeType;
    newRow["Id"] = id.IntegerValue.ToString();
    newRow["Name"] = elem.Name;

    newRow["Category"] = (null == elem.Category)
      ? "<null>"
      : elem.Category.Name; // added by jeremy

    newRow["Document"] = doc.Title;
  }
  m\_ChangesInfoTable.Rows.Add( newRow );
}
```

So the answer to your question is presumably 'no way', and that is as designed.

There is an obvious workaround, though:

The DocumentChanged event is a very simple notification after the fact. The transaction in which the document was modified has already been closed, you cannot do anything more about it yourself, and as we have seen, you cannot even query the deleted elements for their data.

If you wish to access the document before the transaction is closed, there is a very powerful alternative possibility, the dynamic model update framework or DMU. It enables you to register an updater to be triggered by certain events, and also to react on these events within the same transaction that triggered them.
It is demonstrated by the
[DynamicModelUpdate and DistanceToSurfaces SDK samples](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html#2) and
Saikat's
[Structural DMU sample](http://thebuildingcoder.typepad.com/blog/2010/08/structural-dynamic-model-update-sample.html).

In your case, you can easily implement an updater that simply brings up a message box as described above, and register a trigger for it which reacts to the deletion of floors and nothing else.

Your updater Execute method will be called while the transaction is still open.
Unfortunately, even though the transaction is open, you can no longer access or modify the elements that have already been deleted.

The only information available to you at this point is the element id of the deleted element.
If you are interested in floors only, you can maintain a list of all floor element ids and ensure that it is kept up to date using either DMU or the DocumentChanged event.

**Response:** Yes that all looks good.
By the way, your assumption is correct that I was using the DocumentChanged event.

I looked into the 'dynamic model update framework'.
This sounds like what I want.

Later: Thanks again, this solved my problem and was easy to implement.

I filter for floors and use the 'GetChangeTypeElementDeletion' in the 'AddTrigger' call, then in my 'IUpdater' class in the 'Execute' method I simply display a task dialog showing the number of floors being deleted.
There is no need for any more filtering, I can just use 'data.GetDeletedElementIds().Count'.