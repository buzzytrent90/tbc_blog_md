---
post_number: "0834"
title: "Create Dimension between Two Lines"
slug: "create_dimension"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'references', 'revit-api', 'transactions', 'views', 'walls', 'windows']
source_file: "0834_create_dimension.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0834_create_dimension.html"
---

﻿

### Create Dimension between Two Lines

Another one of those recurring questions: how to create dimensioning.

I took the opportunity to highlight how to search the samples, find a solution, and apply a correction to RevitLookup yourself.

You have the code.

May the Force be with you.

**Question:** I tried to create detail drawings programmatically using the ItemFactoryBase NewDimension method.

I am creating detail lines in the drafting view based the on the geometry wall elements and would like to use Revit dimensioning to insert measurements into the detail drawings.

How do I obtain the reference for the detail lines of a drafting view to be able to call the NewDimension method?

I am sure that this is possible, since I can create dimension lines in a drafting view using the dimension line commands on the Annotate ribbon.

**Answer:** Whenever you are faced with a question like this, please always begin by studying the three main official sources of information:

- The Revit API help file RevitAPI.chm.- The online wikihelp Revit
    [API Developer's Guide](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00006-API_Developer%27s_Guide).- The Revit SDK sample source code.

Normally, when researching an issue like this myself, I perform a global search across the entire project in the SDKSamples2013.sln solution.

This time, happily, I extended it to the entire samples directory instead, including RevitLookup.
To tell the truth, I even searched my whole hard disk for all C# files containing NewDimension, because there were not that many in total.

This turned up the following hits in the Revit SDK folder (copy and paste to an editor to see the truncated lines):

```
C:\...\SDK\RevitLookup\CS\Test\TestElements.cs(1255):               //        m_revitApp.ActiveUIDocument.Document.Create.NewDimension(dimension.View, line, dimension.References, dimension.DimensionType);
C:\...\SDK\RevitLookup\CS\Test\TestElements.cs(1470):            m_revitApp.ActiveUIDocument.Document.Create.NewDimension(m_revitApp.ActiveUIDocument.Document.ActiveView,
C:\...\SDK\RevitLookup\CS\Test\TestElements.cs(1475):            m_revitApp.ActiveUIDocument.Document.FamilyCreate.NewDimension(m_revitApp.ActiveUIDocument.Document.ActiveView,
C:\...\SDK\RevitLookup\CS\Utils\Elements.cs(272):            Dimension dimensionClone = app.ActiveUIDocument.Document.Create.NewDimension(dimension.View, line, dimension.References);
C:\...\SDK\Samples\CreateDimensions\CS\Command.cs(213):                    Dimension newDimension = doc.Create.NewDimension(
C:\...\SDK\Samples\CreateDimensions\CS\Command1.cs(219):                    Dimension newDimension = doc.Create.NewDimension(
C:\...\SDK\Samples\FamilyCreation\WindowWizard\CS\CreateDimension.cs(31):    /// The class allows users to create dimension using Document.FamilyCreate.NewDimension() function
C:\...\SDK\Samples\FamilyCreation\WindowWizard\CS\CreateDimension.cs(91):            dim = m_document.FamilyCreate.NewDimension(view, line, refArray);
C:\...\SDK\Samples\FamilyCreation\WindowWizard\CS\CreateDimension.cs(125):            dim = m_document.FamilyCreate.NewDimension(view, line, refArray);
C:\...\SDK\Samples\FamilyCreation\WindowWizard\CS\CreateDimension.cs(160):            dim = m_document.FamilyCreate.NewDimension(view, line, refArray);
```

The very first of those, in the RevitLookup test framework, looks like it should answer your question right away:
```csharp
  XYZ location1 = GeomUtils.kOrigin;
  XYZ location2 = new XYZ( 20.0, 0.0, 0.0 );
  XYZ location3 = new XYZ( 0.0, 20.0, 0.0 );
  XYZ location4 = new XYZ( 20.0, 20.0, 0.0 );

  Curve curve1 = m\_revitApp.Application.Create
    .NewLine( location1, location2, true );

  Curve curve2 = m\_revitApp.Application.Create
    .NewLine( location3, location4, true );

  DetailCurve dCurve1 = null;
  DetailCurve dCurve2 = null;

  if( !doc.IsFamilyDocument )
  {
    dCurve1 = doc.Create.NewDetailCurve(
      doc.ActiveView, curve1 );

    dCurve2 = doc.Create.NewDetailCurve(
      doc.ActiveView, curve2 );
  }
  else
  {
    if( null != doc.OwnerFamily
      && null != doc.OwnerFamily.FamilyCategory )
    {
      if( !doc.OwnerFamily.FamilyCategory.Name
        .Contains( "Detail" ) )
      {
        MessageBox.Show(
          "Please make sure you open a detail based family template.",
          "RevitLookup", MessageBoxButtons.OK,
          MessageBoxIcon.Information );

        return;
      }
    }
    dCurve1 = doc.FamilyCreate.NewDetailCurve(
      doc.ActiveView, curve1 );

    dCurve2 = doc.FamilyCreate.NewDetailCurve(
      doc.ActiveView, curve2 );
  }

  Line line = m\_revitApp.Application.Create.NewLine(
    location2, location4, true );

  ReferenceArray refArray = new ReferenceArray();

  refArray.Append( dCurve1.GeometryCurve.Reference );
  refArray.Append( dCurve2.GeometryCurve.Reference );

  if( !doc.IsFamilyDocument )
  {
    doc.Create.NewDimension(
      doc.ActiveView, line, refArray );
  }
  else
  {
    doc.FamilyCreate.NewDimension(
      doc.ActiveView, line, refArray );
  }
```

I tried running this sample command by selecting the ribbon tab Add-Ins > Revit Lookup > Test Framework... > Object Hierarchy > APIObject > Element > Dimension > OK.

That calls the DimensionHardWired method to do the job.

Unfortunately, it throws an exception saying "Attempt to modify the model outside of transaction".

I fixed it by adding the following lines right at the beginning of the method:
```csharp
  using( Transaction tx = new Transaction( doc ) )
  {
    tx.Start( "DimensionHardWired" );
```

Obviously, I need to commit the transaction and close the using statement at the end again:
```csharp
    tx.Commit();
  }
```

Now is successfully completes with the following result:
![New dimension](img/new_dimension.png)

You can also look at two The Building Coder samples creating wall dimensioning, either by
[iterating over the wall faces](http://thebuildingcoder.typepad.com/blog/2011/02/dimension-walls-by-iterating-faces.html) to
pick the two suitable ones, or using
[FindReferencesByDirection to shoot a ray](http://thebuildingcoder.typepad.com/blog/2011/02/dimension-walls-using-findreferencesbydirection.html) and
find them directly.

They both retrieve the geometry references from the wall faces, and not from a simple detail line, though.

All together, this should provide more than enough to solve your issue.

#### Accessing the Path of a Family Document from the Family Instance

Before closing, here is pointer to a cool discovery by Saikat Bhattacharya: if you try to find the family document of a family instance and go through the path family instance > family symbol > family > document, you end up at the document containing the family instance.
If you are actually after the family document defining the instance, you need to go through the EditFamily method instead.
More details on
[accessing the path of a family document from an instance](http://adndevblog.typepad.com/aec/2012/09/accessing-the-path-a-revit-family-document-from-the-family-instance.html).