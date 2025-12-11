---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.6
content_type: qa
optimization_date: '2025-12-11T11:44:17.413363'
original_url: https://thebuildingcoder.typepad.com/blog/2069_fab_part_service.html
post_number: '2069'
reading_time_minutes: 5
series: general
slug: fab_part_service
source_file: 2069_fab_part_service.md
tags:
- elements
- filtering
- levels
- parameters
- python
- revit-api
- selection
- sheets
- views
title: Fab Part Service
word_count: 1069
---

### DevCon, API Docs and Fabrication Part Service
My last Autodesk conference nears, new and updated API documentation online, and a tutorial-style exloration of implementing fabrication part – show service:
- [DevCon in Amsterdam](#2)
- [CivApiDocs Civil3D API documentation](#3)
- [ApiDocs Revit 2025 API docs](#4)
- [Fabrication part – show service](#5)
#### DevCon in Amsterdam
I will be attending [DevCon in Amsterdam](https://aps.autodesk.com/blog/register-today-autodesk-devcon-returns-2025-amsterdam)!
It is taking place on May 20-21 2025 in
the famous old [Beurs van Berlage](https://en.wikipedia.org/wiki/Beurs_van_Berlage) stock market
constructed 1896-1903.
I mentioned the [call for papers](https://thebuildingcoder.typepad.com/blog/2024/11/devcon-ai-for-revit-api-modeless-add-ins-leave.html#2) in November.
May will also be the last month of my life as an Autodesk employee.
![DevCon in Beurs van Berlage](img/devcon_in_boers_berlage_by_gemini.jpg "DevCon in Beurs van Berlage")
#### CivApiDocs Civil3D API Documentation
Jaime Alonso Candau of the Revit API consulting company [Nonica.io](https://nonica.io/) published
[Civ API Docs](https://civapidocs.com/)
The new website provides online Civil3D API documentation for the releases 2022, 2023, 2024 and 2025, in the same style as
the [`RevApiDocs` Revit API documentation](https://revapidocs.com/) supporting
Revit 2025 as well as preceding versions that
I [mentioned in February](https://thebuildingcoder.typepad.com/blog/2025/02/unit-testing-and-more-serious-matters.html#4).
Many thanks to Jaime for creating and sharing these resource.
#### ApiDocs Revit 2025 API Docs
Following up on that,
[Gui Talarico](https://twitter.com/gtalarico) now also
updated the [apidocs](https://apidocs.co) web site for both Revit 2025 and Revit 2025.3 API:
- [Revit 2025.3 API](https://apidocs.co/apps/revit/2025.3/1593f994-fb7b-4b7d-ae1d-1c0ba3337577.htm)
- [Revit 2025 API](https://apidocs.co/apps/revit/2025/1593f994-fb7b-4b7d-ae1d-1c0ba3337577.htm#)
[Apidocs](https://apidocs.co) also includes API documentation for Grasshopper, Navisworks, Rhino and previous Revit releases all the way back to Revit 2015.
Thank you, Gui, for maintaining this invaluable resource!
#### Fabrication Part – Show Service
[Steven Williams](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/11800801) shared
a very nice tutorial-style explanation of how to replicate
[Fabrication Part – Show Service](https://forums.autodesk.com/t5/revit-api-forum/fabrication-part-show-service/td-p/13354945):
This post started as a question, but I figured it out and thought someone might be interested. I couldn't find any examples of this in the API samples or through web searches, and ChatGPT failed miserably.
My goal is to replicate the Show Service button that appears in the Modify tab when you select a Fabrication Part.
![Fabrication Part - Show Service](img/sw_fab_part_service_1.png "Fabrication Part - Show Service")
Clicking this button opens the MEP Fabrication Parts panel to the matching service and palette that generated that part, and highlights the button that was used.
I snooped through the attributes and parameters of several fabrication parts but could only find ServiceId indicating what button was used to draw the part. The fabrication button name does not necessarily correspond to the name of the part drawn in Revit.
Example: I have a button named "Pipe":
![Pipe](img/sw_fab_part_service_2.png "Pipe")
These are the relevant properties of the button object, which I obtained like this:

```
# doc is the Autodesk.Revit.DB.Document in question
# service_id, palette_index, and button_index are given integers
fabrication_configuration = FabricationConfiguration.GetFabricationConfiguration(doc)
service = fabrication_configuration.GetService(service_id)
button = service.GetButton(palette_index, button_index)
print(button.ButtonIndex) # == 0
print(button.Code) # == P01
print(button.ConditionCount) # == 1
print(button.IsAHanger) # == False
print(button.IsStraight) # == True
print(button.Name) # == Pipe
print(button.PaletteIndex) # == 0
print(button.ServiceId) # == 3381
```

Not much information about what will be drawn in Revit.
There is a method FabricationServiceButton.ContainsFabricationPartType(partType) that I finally found to be the key.
The part drawn in Revit looks like this ("Cerro Type L Hard Copper Pipe"):
![Copper pipe](img/sw_fab_part_service_3.png "Copper pipe")
It has attributes and properties like this:

```
# select your part
ref = uidoc.Selection.PickObject(ObjectType.Element)
element = doc.GetElement(ref)
# properties
print(element.ProductName) # == Pipe
print(element.ProductLongDescription) # == Type L Hard Copper Pipe
print(element.ProductOriginalEquipmentManufacture) # == Cerro
print(element.ProductSpecificationDescription) # == Type L
print(element.Name) # == Default
# parameters
print(element.LookupParameter('Product Name').AsValueString()) # == Pipe, also BuiltInParameter.FABRICATION_PRODUCT_DATA_PRODUCT
print(element.LookupParameter('Product Long Description').AsValueString()) # == Type L Hard Copper Pipe, also BuiltInParameter.FABRICATION_PRODUCT_DATA_LONG_DESCRIPTION
print(element.LookupParameter('OEM').AsValueString()) # == Cerro, also BuiltInParameter.FABRICATION_PRODUCT_DATA_OEM
print(element.LookupParameter('Product Specification Description').AsValueString()) # == Type L, also BuiltInParameter.FABRICATION_PRODUCT_DATA_SPECIFICATION
```

Then I discovered the `FabricationPartType` class that can be retrieved using the fabrication part `GetTypeId` method:

```
doc.GetElement(element.GetTypeId())
```

The API documentation says "For the product-based MAP parts, every size is a new part type in Revit.
For others, one part type can have many sizes."
Now I can put FabricationServiceButton.ContainsFabricationPartType(partType) together with element.GetTypeId().
There is one last step, which took me a little longer.
Sometimes we have several ITM files in a single button:
![Adapters](img/sw_fab_part_service_4.png "Adapters")
Revit classifies each of these as a Condition. ContainsFabricationPartType() only tells you that the button contains the given partType; now you have to get the condition that matches the partType. For that you have to go back to the FabricationPartType class and use the Lookup(document, button, condition) method with each valid condition the button contains.
This is the summary.

```
# you can run this in RevitPythonShell
fabrication_configuration = FabricationConfiguration.GetFabricationConfiguration(doc)

# select a FabricationPart
# -- consider creating an ISelectionFilter that only passes elements of type FabricationPart
ref = uidoc.Selection.PickObject(ObjectType.Element)
element = doc.GetElement(ref)
part_type = doc.GetElement(element.GetTypeId())

service_id = element.ServiceId
service = fabrication_configuration.GetService(service_id)

matching_button = None
matching_condition = None
for palette_index in range(service.PaletteCount):
    for button_index in range(service.GetButtonCount(palette_index)):
        button = service.GetButton(palette_index, button_index)
        for condition in range(button.ConditionCount):
            part_type_element = FabricationPartType.Lookup(doc, button, condition)
            # the GUI always picks the first button and condition that matches
            # -- the efficiency of this would be improved by making this into
            # a a function that yields the first match
            if part_type_element != ElementId.InvalidElementId and matching_button is None:
                matching_button = button
                matching_condition = condition
```

I think you can now use this matching button, for example, to replicate the 'Create Similar' command for fabrication parts, using the second overload with matching_button and matching_condition from above.

```
public static FabricationPart Create(
  Document document,
  FabricationServiceButton button,
  int condition,
  ElementId levelId
)
```

Many thanks to Steven for his nice clear explanation and overview.