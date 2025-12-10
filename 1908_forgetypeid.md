---
post_number: "1908"
title: "Forgetypeid"
slug: "forgetypeid"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'parameters', 'python', 'revit-api', 'rooms', 'sheets', 'walls']
source_file: "1908_forgetypeid.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1908_forgetypeid.html"
---

### ForgeTypeId and Units Revisited
We already discussed quite a few aspects of the Revit 2022 unit handling API and `ForgeTypeId` usage:

* [Revit 2022 released](https://thebuildingcoder.typepad.com/blog/2021/04/revit-2022-released.html)
* [Replacing deprecated ParameterType with ForgeTypeId](https://thebuildingcoder.typepad.com/blog/2021/04/pdf-export-forgetypeid-and-multi-target-add-in.html#2)
* [Removal of DisplayUnitType in RevitLookup 2022](https://thebuildingcoder.typepad.com/blog/2021/04/revit-2022-migrates-bim360-team-to-docs.html)
* [ParameterType and ForgeTypeId in the Revit 2022 SDK and The Building Coder samples](https://thebuildingcoder.typepad.com/blog/2021/04/revit-2022-sdk-and-the-building-coder-samples.html)
* [What's New in the Revit 2022 API](https://thebuildingcoder.typepad.com/blog/2021/04/whats-new-in-the-revit-2022-api.html)

Here are some more related questions that came up since then:
- [`FixtureUnit` ParameterType](#2)
- [Revit 2022 unit handling API in Dynamo](#3)
- [String values for Forge units](#4)
- [Unit conversion without knowing](#5)
- [How will we live together?](#6)
![Yardstick](img/yardstick.jpg "Yardstick")
Before diving into this topic, let us congratulate the China team in their celebration of the [Dragon Boat festival](https://en.wikipedia.org/wiki/Dragon_Boat_Festival) today.
- Traditional sport : dragon boating race
- Customary food: Zongzi, a kind of rice dumpling, packaged in bamboo or reed leaves, sweet or salty, even in ice cream (by Starbucks :-)
- Memorials on the sage [Qu Yuan](https://en.wikipedia.org/wiki/Qu_Yuan), poet and politician, known for his patriotism
- Prayer for health and peace by hanging calamus or mugwort on the door
![Dragon Boat festival](img/dragon_boat_festival.png "Dragon Boat festival")
#### FixtureUnit ParameterType
David Becroft comes to the rescue again answering
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [convert `ParameterType.FixtureUnit` to `ForgeTypeId`](https://forums.autodesk.com/t5/revit-api-forum/convert-parametertype-fixtureunit-to-forgetypeid/m-p/10268488)
\*\*Question:\*\* I am trying to port some legacy code to Revit 2022 and I am running into the problem converting some `ParameterType` enum values to a `ForgeTypeId`.
`FixtureUnit` and `Invalid` (the result for `ElementId` parameters) are the most pressing ones.
There does not seem to be anything Fixture or Piping related in the `SpecTypeId` class that may apply.
`PipeDimension` comes closest but is not, I think, the same?
Plus, it also exists separately in the ParameterType enum which makes it unlikely that it got a double meaning in Revit 2022.
According to the release documentation there should be a deprecated method in the Parameter class that can help porting ParameterType enum values to ForgeTypeId objects, but Intellisense does not seem to be aware of the existence of such a method.
\*\*Answer:\*\* These methods are in `SpecUtils`, `GetSpecTypeId(ParameterType)` and `GetParameterType(ForgeTypeId)`.
In place of `ParameterType.FixtureUnit`, you can use `SpecTypeId.Number`.
`ParameterType.Invalid` is equivalent to an empty, default-constructed `ForgeTypeId`, i.e. `new ForgeTypeId()`.
#### Revit 2022 Unit Handling API in Dynamo
Konrad Sobon does a great job explaining how to deal with this in Dyname and Python in his discussion
of [handling the Revit 2022 unit changes](https://archi-lab.net/handling-the-revit-2022-unit-changes).
#### String Values for Forge Units
The unit handling changes also affected some Forge apps:
![RVT ForgeTypeId in Forge](img/forge_type_id_schema_id.jpg "RVT ForgeTypeId in Forge")
This was discussed and resolved in the StackOverflow question
on [Autodesk Forge returning odd measurement data](https://stackoverflow.com/questions/63992151/autodesk-forge-is-returning-odd-measurement-data)
I ended up implementing
the [method `ListForgeTypeIds` in The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs#L1306-L1367) to help resolve the issue.
I also shared this in the discussion
on [DisplayUnitType in Revit 2022](https://forums.autodesk.com/t5/revit-api-forum/displayunittype-in-revit-2022/m-p/10320697).
#### Unit Conversion Without Knowing
In a related vein, how can you implement a unit conversion without knowing the internal Revit unit?
This question has been around since the beginnings of The Building Coder and was first addressed in blog post #11
on [units](https://thebuildingcoder.typepad.com/blog/2008/09/units.html) in September 2008.
It came up again in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on a [unit conversion question](https://forums.autodesk.com/t5/revit-api-forum/unit-conversion-question/m-p/9840917):
Simply set the value that you are analysing to 1.0 manually through the user interface using your current display units.
Then, use RevitLookup to analyse what value `V` actually gets stored.
The ratio between those two values, i.e., `V`/1.0 == `V`, immediately provides the conversion factor from display unit to internal database unit (or v.v.).
#### How Will We Live Together?
Moving from Revit to the topic of architecture in general,
at [La Biennale di Venezia](https://www.labiennale.org),
the [17th International Architecture Exhibition](https://www.labiennale.org/en/architecture/2021)
is focussed on the theme of \*How will we live together?\*
> We need a new spatial contract.
In the context of widening political divides and growing economic inequalities, we call on architects to imagine spaces in which we can generously live together.
At the biennale,
the [German pavilion looks back from the future](https://www.floornature.com/blog/biennale-di-venezia-german-pavilion-looks-back-future-16269):
> [2038](https://2038.xyz) is the name of the German Pavilion in the 17th International Architecture Exhibition at Biennale di Venezia.
A look back from the future, which the curators envision as a world where many of today’s problems have been overcome.
Digital technology makes it possible to visit the pavilion from anywhere in the world: [2038.xyz](https://2038.xyz)