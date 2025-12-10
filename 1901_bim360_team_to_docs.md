---
post_number: "1901"
title: "Bim360 Team To Docs"
slug: "bim360_team_to_docs"
author: "Jeremy Tammik"
tags: ['parameters', 'revit-api', 'sheets', 'views']
source_file: "1901_bim360_team_to_docs.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1901_bim360_team_to_docs.html"
---

### Revit 2022 Migrates BIM360 Team to Docs
We continue the foray into Revit 2022 enhancements with a real-world migration tool using the new `SaveAsCloudModel` functionality and the flat migration of RevitLookup:
- [Save as cloud model from BIM360 Team to Docs](#2)
- [RevitLookup 2022](#3)
- [A librarian's take on corona](#4)
#### Save as Cloud Model from BIM360 Team to Docs
My colleague Zhong Wu published an enhancement to the Revit 2022 SDK sample \*CloudAPISample\*
to [migrate Revit Worksharing models from BIM 360 Team to BIM 360 Docs – powered by Revit 2022 Cloud Worksharing API](https://forge.autodesk.com/blog/migrate-revit-worksharing-models-bim-360-team-bim-360-docs-powered-revit-2022-cloud).
In his own words:
Revit 2022 was officially released on April 8th, 2021
with [a host of new features](https://thebuildingcoder.typepad.com/blog/2021/04/revit-2022-released.html).
Support for saving a Revit worksharing central model to the cloud is one important enhancement in the Revit 2022 API, using the method
```csharp
Document.SaveAsCloudModel(Guid, Guid, String, String)
```
I enhanced it to support uploading a local workshared file into BIM 360 Design as a Revit Cloud Worksharing central model.
The Revit 2022 SDK also includes a sample add-in \*CloudAPISample\* demonstrating how to use this API.
I made some improvements to make it easy to use and demonstrate how to migrate your Revit cloud worksharing model from BIM 360 Team to BIM 360 Docs.
It includes the following features:
- Access all the contents within BIM 360 Team and Docs by logging in with your Autodesk Account
- Download the Revit models from BIM 360 Team to a specified local folder
- Select a target folder by navigating from BIM 360 Docs
- Upload the Revit models from the local folder to the target folder on BIM 360 Docs
- Reload the links to the correct model in the cloud
![Migrating from BIM360 Team to BIM360 Docs](img/zw_bim360docs_migration.png "Migrating from BIM360 Team to BIM360 Docs")

Revit Cloud Worksharing Model Migration Sample from BIM 360 Team to BIM 360 Docs

The sample tool source code, full documentation and demo is hosted in
the [forge-rcw.file.migration-revit.addon GitHub repository](https://github.com/JohnOnSoftware/forge-rcw.file.migration-revit.addon).
Enjoy coding with Revit, Forge and BIM360, and please feel free to enhance the sample based on your needs.
Pull requests are always welcome.
Ever so many thanks to Zhong for implementing and sharing this useful and important utility!
#### RevitLookup 2022
I performed a quick flat migration of RevitLookup to the Revit 2022 API.
I just encountered one error and two warnings.
The error is caused by code checking the `DisplayUnitType`, which was deprecated in Revit 2021:
```csharp
#pragma warning disable CS0618
// warning CS0618: `DisplayUnitType` is obsolete:
// This enumeration is deprecated in Revit 2021 and may be removed in a future version of Revit.
// Please use the `ForgeTypeId` class instead.
// Use constant members of the `UnitTypeId` class to replace uses of specific values of this enumeration.
if( 2 == parameters.Length )
{
ParameterInfo p1 = parameters.First();
ParameterInfo p2 = parameters.Last();
return p1.ParameterType == typeof( Field )
&& (p2.ParameterType == typeof( DisplayUnitType )
|| p2.ParameterType == typeof( ForgeTypeId ));
}
#pragma warning restore CS0618
```
Since `DisplayUnitType` is obsolete in Revit 2022, we have no choice but to remove it.
The two warnings are related to the deprecated `ParameterType` and can be left for the moment.
Here is the complete [error and warning log so far](zip/revit_2022_revitlookup_errors_warnings_0.txt).
I wish you easy sailing and much success in your own migration work.
#### A Librarian's Take on Corona
A quite poetic librarian's recommendation to stay healthy:
![A librarian's take on Corona](img/corona_librarian.jpg "A librarian's take on Corona")