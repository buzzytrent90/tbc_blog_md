---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.092653'
original_url: https://thebuildingcoder.typepad.com/blog/0519_set_duct_width_height.html
post_number: 0519
reading_time_minutes: 2
series: mep
slug: set_duct_width_height
source_file: 0519_set_duct_width_height.htm
tags:
- csharp
- elements
- family
- parameters
- revit-api
- transactions
- mep
title: Setting Duct Width and Height Requires Regeneration
word_count: 495
---

### Setting Duct Width and Height Requires Regeneration

Whenever you have a problem modifying or querying the model with your Revit plug-in, one of the first things to check is always your regeneration mode.
If you are using the manual regeneration option, you need to check whether you possibly omitted a required intermediate regeneration call.
It seems that there can never be enough reminders of this.
Here is another case at hand:

**Question:** I am sizing rectangular duct fittings through the API, and I noticed that the different dimensions for width and height must be set in different transactions.
Otherwise, you lose the first size change when the second one is made.

Is this a feature or a bug?

The requirement to start and commit a separate transaction for each dimension change makes sizing of the network quite clumsy and even slow with larger drawings.

Here are two simple example code snippets in managed C++ setting the width and height of an elbow to 500x500 mm.
They have both the TransactionMode and the RegenerationOption set to Manual.
The first one uses a single transaction and does not work.
The second uses two transactions and does.

Here is the first test with the size changes in same transaction:
```csharp
Result AddinTest1::Execute(
  ExternalCommandData^ commandData,
  System::String^% message,
  ElementSet^ elements)
{
  Document ^doc = commandData->Application
    ->ActiveUIDocument->Document;

  // Duct/bend IDs in example project

  int west\_east   = 505594;
  int north\_south = 505598;
  int bend\_id     = 505610;

  ElementId ^elemId = gcnew ElementId(west\_east);
  Element ^ductElem1 = doc->Element::get(elemId);

  elemId = gcnew ElementId(north\_south);
  Element ^ductElem2 = doc->Element::get(elemId);

  elemId = gcnew ElementId(bend\_id);
  Element ^bendElem1 = doc->Element::get(elemId);

  FamilyInstance ^bend
    = safe\_cast<FamilyInstance^>(bendElem1);

  ConnectorSet ^cSet
    = bend->MEPModel->ConnectorManager->Connectors;

  Transaction tr(doc, L"sizing");
  tr.Start();

  for each (Connector ^connector in cSet)
  {
    if (connector->ConnectorType
      != ConnectorType::EndConn)
    {
      continue;
    }

    connector->Width::set((500 \* 0.0032808399));
    connector->Height::set((500 \* 0.0032808399));   // This is not set into the drawing
    break;
  }

  tr.Commit();

  return Result::Succeeded;
}
```
Here is the second test doing exactly the same thing but using two separate transactions:
```csharp
  Transaction tr(doc, L"sizing");
  tr.Start();

  // We do separate transactions for both dimensions

  for each (Connector ^connector in cSet)
  {
    if (connector->ConnectorType
      != ConnectorType::EndConn)
    {
      continue;
    }

    connector->Width::set((500 \* 0.0032808399));
    break;
  }

  tr.Commit();

  tr.Start();

  for each (Connector ^connector in cSet)
  {
    if (connector->ConnectorType
      != ConnectorType::EndConn)
    {
      continue;
    }

    connector->Height::set((500 \* 0.0032808399));
    break;
  }

  tr.Commit();
```

**Answer:** I have two suggestions for you, but only one is meant seriously:

1. You could use automatic regeneration mode.
   I would not recommend this, however, for many reasons, not least because it is obsolete and will soon be removed.- You could try to regenerate the document between the setting of the two parameters. Does this help?

**Response:** Thanks for your quick and complete answer.

Yes, regeneration between those settings **does** help; no whole additional transaction procedure is needed.

Thanks again and best wishes for this New Year 2011!