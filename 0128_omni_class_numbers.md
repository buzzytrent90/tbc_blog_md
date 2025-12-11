---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.406961'
original_url: https://thebuildingcoder.typepad.com/blog/0128_omni_class_numbers.html
post_number: 0128
reading_time_minutes: 4
series: general
slug: omni_class_numbers
source_file: 0128_omni_class_numbers.htm
tags:
- csharp
- elements
- family
- filtering
- parameters
- revit-api
- views
- windows
title: Omni Class Numbers
word_count: 874
---

### Omni Class Numbers

Revit 2010 supports
[OmniClass codes](http://www.omniclass.org).
The Revit Architecture help provides the following background information on these codes in the section on Preparing Content for Sharing with Autodesk Seek, which is currently only available in the English edition:

OmniClass is a new classification system for the construction industry. The Autodesk Seek website uses codes from OmniClass Table 23 to filter and identify shared content. A code consists of an OmniClass number and title.

If an OmniClass code is not already assigned to a family, you are prompted to assign one during the sharing process. However, you can continue to share with Autodesk Seek without defining one. All Revit families have parameters for assigning an OmniClass code, except for the System and Annotation families.

You can access the OmniClass Number and OmniClass Title parameters in the Family Category and Parameters dialog under Family Parameters. See Family Category and Parameters.

**Question:**
How can I retrieve all available OmniClass numbers and their descriptions from a model?
I also need to retrieve the OmniClass numbers from the elements in the model.

I am using the export to ODBC functionality in Revit Architecture to find relevant data.
The export process produces an OmniClassNumbers table.
You can see the OmniClass numbers in the Revit user interface by opening a family drawing and clicking on the Category and Parameters tool.
The OmniClass Number field is displayed in Family Parameters in the bottom section of that window.
The resulting dialog displays the same data as that exported to the OmniClassNumbers table by the built in tool.

I need to

1. Acquire the data already exported to the OmniClassNumbers table by the built in tool.- Determine what OmniClass an element or type belongs to

**Answer:**
The Revit API defines two built-in parameters for accessing the OmniClass data:

- OMNICLASS\_CODE- OMNICLASS\_DESCRIPTION

We can extract these parameter values for the elements which are contained in the model. Some may already be present as instances in the model, some may be part of the template that was used to create it, and some may have been loaded into it as families.

We can extract these parameter values for the elements which are contained in the model.
Some of these elements may already be present as instances in the model, some may be part of the template that was used to create it, and some may have been loaded into it as families.

Below is a piece of code to filter out all the elements in the model which have OmniClass data available. First, we define two class variables for the built-in parameters of interest:

```csharp
BuiltInParameter \_bipCode
  = BuiltInParameter.OMNICLASS\_CODE;

BuiltInParameter \_bipDesc
  = BuiltInParameter.OMNICLASS\_DESCRIPTION;
```

Then we can implement an external command class with the following code in its Execute method, which produces the same data as the Revit exported ODBC data from the user interface:

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

List<Element> set = new List<Element>();

ParameterFilter f
  = app.Create.Filter.NewParameterFilter(
    \_bipCode,
    CriteriaFilterType.NotEqual,
    string.Empty );

ElementIterator it = doc.get\_Elements( f );

using( StreamWriter sw
  = File.CreateText( "C:/omni.txt" ) )
{
  while( it.MoveNext() )
  {
    Element e = it.Current as Element;
    sw.WriteLine( string.Format(
      "{0} code {1} desc {2}",
      Util.ElementDescription( e ),
      e.get\_Parameter( bipCode ).AsString(),
      e.get\_Parameter( bipDesc ).AsString() ) );
  }
  sw.Close();
}
return IExternalCommand.Result.Failed;
```

We use a highly efficient Revit API filter to select only those elements which have a non-empty OmniClass code. For those elements, the code and description is extracted from the element parameters and logged to a text file.

This code also demonstrates how to extract the built-in OmniClass parameters from any given individual element.

Many thanks to Saikat Bhattacharya for handling this case!

#### Revit 2010 Product Feature Recordings

There are a number of YouTube recordings available discussing and demonstrating new Revit 2010 product features. Here is a list of some of them:

- Revit Architecture 2010
  - [Revit Arch 2010 Whats New - Part 1 of 3](http://www.youtube.com/watch?v=dLy-DrUgTsA)
  - [Revit Arch 2010 Whats New - Part 2 of 3](http://www.youtube.com/watch?v=HTNPZ1lyuV0&feature=related)
  - [Revit Arch 2010 Whats New - Part 3 of 3](http://www.youtube.com/watch?v=Vt7VS6Bk4mU&feature=related)
  - [Autodesk Revit Architecture 2010 User Interface Tour](http://www.youtube.com/watch?v=afQOYjQ_peY&feature=related)
  - [Revit 2010 UI Preview](http://www.youtube.com/watch?v=saocarAdtlI&feature=related)
  - [The Revit 2010 Ribbon - Designing the User Experience](http://www.youtube.com/watch?v=qZBIAtdCEss&feature=related)
  - [Autodesk Revit Architecture 2010 Demo](http://www.youtube.com/watch?v=vhz4uyXKsmQ&feature=related)
  - [Autodesk Revit Architecture 2010 and 3ds Max Design Demo](http://www.youtube.com/watch?v=kb3T5d8ReUU&feature=related)
- Revit MEP 2010
  - [What's New in Autodesk Revit MEP 2010 - Part 1 of 3](http://www.youtube.com/watch?v=xPvuH90zPHE&feature=related)
  - [What's New in Autodesk Revit MEP 2010 - Part 2 of 3](http://www.youtube.com/watch?v=fbd2Nmwm5jc&feature=related)
  - [What's New in Autodesk Revit MEP 2010 - Part 3 of 3](http://www.youtube.com/watch?v=5HHicDjIsco&feature=related)

- Revit Structure 2010
  - [Autodesk Revit Structure 2010 Demo by Scott Hammond](http://www.youtube.com/watch?v=h6BbeqhdTp0&feature=related)