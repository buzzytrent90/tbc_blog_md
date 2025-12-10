---
post_number: "0113"
title: "More Questions"
slug: "more_questions"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'revit-api', 'rooms', 'schedules', 'views', 'walls']
source_file: "0113_more_questions.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0113_more_questions.html"
---

### More Questions

Here are some more of the questions that were raised lately, including some cases handled by my colleagues Joe Ye and Saikat Bhattacharya:

- Can I define my own user or API family type parameters that provide a list of options to the user and are displayed in a drop-down list, like 'Level' or 'Phase Created', or through a dialogue, like 'Assembly Code'?
- I am trying to use a built-in parameter by calling the Parameter method on a family type object, but it produces an error saying 'Parameter' is not a member of 'Autodesk.Revit.Symbols.FloorType'.
  What is wrong?
- Is it possible to change a design option to primary in the Revit API, for instance using the Element.DesignOption property?
- Is it possible to find all elements associated with a specific design option?
- How can I change the colour of one individual element instance through the API?
- How can I change the colour of a face using the API? Can I change colour of one and only one face?
- Is it possible to use event handling or some other method to add steps when a user clicks on Save As? We would like to make sure a few things are done before the Save As command executes.

#### Drop-down list for parameter values

**Question:**
Can I define my own user or API family type parameters that provide a list of options to the user and are displayed in a drop-down list, like 'Level' or 'Phase Created', or through a dialogue, like 'Assembly Code'?

**Answer:**
Nope, unfortunately not. As a workaround, you can always define your own user interface to set your parameters, and not make use of the default Revit one.

#### Parameter get method

**Question:**
I am trying to use built-in parameters like this:

```
famName = ft.Parameter(
  BuiltInParameter.SYMBOL_FAMILY_NAME_PARAM ).AsString
```

This produces an error saying

```
'Parameter' is not a member of 'Autodesk.Revit.Symbols.FloorType'.
```

What is wrong?

**Answer:**
The official name of the API method to obtain the parameter is indeed Parameter.
However, some internal compilation mechanism has converted it to get\_Parameter.
If you look at the true methods available, for instance by using the Visual Studio Intellisense or the object browser, you will see the true names listed.

#### Change design option to primary

**Question:**
Is it possible to change a design option to primary in the Revit API, for instance using the Element.DesignOption property?

**Answer:**
No, unfortunately it is not possible to change the design option via the API.
Design options enable the user to add alternative designs within the same project.
Each element can either be in a design option or not at all, in which case it is considered to be part of the main model and have no design alternatives.
The Element.DesignOption property is read-only.
There is a parameter 'Design Option' for this property to store the design option, and this parameter is read-only as well.
For elements that already were assigned a design option property, the design option cannot even be changed through the user interface.
If the design option value is Main Model, then it can still be changed through the user interface.

#### Finding elements associated with a design option

**Question:**
Is it possible to find all elements associated with a specific design option?

**Answer:**
Yes, one can use the parameter filter to quickly find all elements associated with a specified design option.
Here is an example to find all walls with the Main Model design option:

```csharp
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string messages,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;
  CreationFilter cf = app.Create.Filter;

  Filter f1 = cf.NewParameterFilter(
    BuiltInParameter.DESIGN\_OPTION\_PARAM,
    CriteriaFilterType.Equal, "Main Model" );

  Filter f2 = cf.NewTypeFilter( typeof( Wall ) );
  Filter f = cf.NewLogicAndFilter( f1, f2 );

  List<Element> a = new List<Element>();

  doc.get\_Elements( f, a );

  Util.InfoMsg( "There are "
    + a.Count.ToString()
    + " main model wall elements" );

  return IExternalCommand.Result.Succeeded;
}
```

#### Change colour of one instance

**Question:**
How can I change the colour of one individual element instance through the API?

**Answer:**
There is no direct API to change the colour of individual elements.
One workaround is to assign a different material which has a different material colour to the family instance symbol.
To get the colour of the material assigned to a family symbol, you can use the following traversal path:

```
FamilyInstance famIns
  = elem as FamilyInstance;

ElementId matId
  = famIns.Symbol.get_Parameter("Material").AsElementId();

Material mat = doc.get_Element( ref matId ) as Material;

System.Drawing.Color color
  = System.Drawing.Color.FromArgb(
    mat.Color.Red, mat.Color.Green,
    mat.Color.Blue );
```

Assigning a different material with different material colour to family instance symbol will change the colour of all instances of that family.
To change just one single instance you will have to create a new type for it.
This can be done as follows:

- Create a duplicate of the existing type.- Assign a different material colour to the newly created type.- Assign this type to the object that you want to colour differently.

Here is the result of a quick test with three columns. For each column, the type was changed by duplicating the first one and assigning a new material colour to the copy:

![Three columns with different colours](img/three_columns_with_different_colours.gif)
[Continued...](http://thebuildingcoder.typepad.com/blog/2009/04/revit-api-cases-1.html#3)

#### Changing the colour of a face

**Question:**
How can I change the colour of a face using the API? Can I change colour of one and only one face?

**Answer:**
The Face object MaterialElement property is read-only, and this is the only property of a face that refers to colour or material in any way, so you cannot change either colour or material of an individual face directly.

Actually, you should be aware of the fact that Face class is a member of the Autodesk.Revit.Geometry namespace. All classes in this namespace are used to provide views into the Revit database, but not to define the database elements themselves. Therefore, even if there was a way to modify the colour of an individual face, it would have no effect on the Revit model.

To persistently modify the colour of individual faces within the Revit model, you can use the face split operation provided by the Face Splitter tool in the user interface.

Please be aware that there is currently no API access to determine the colour or material of split faces.

#### Use event handling on Save As

**Question:**
Is it possible to use event handling or some other method to add steps when a user clicks on Save As? We would like to make sure a few things are done before the Save As command executes.

**Answer:**
The Revit API help file RevitAPI.chm mentions the DocumentSaveAsEventHandler delegate which is a handler for the event that is fired before the current document is saved as a file with different file name.
This should be what you are looking for.
The SDK samples include the RoomSchedule sample which shows how to register handlers for some events, including the document save as event. The main line is:

```
doc.OnSaveAs += new
  DocumentSaveAsEventHandler( OnDocumentSave );
```

#### More details on my break

Some colleagues asked for more details of my experiences during my break, and whether I took any pictures.
I did not, I was not carrying a camera and thought that there are probably loads of photos of all the places I visited available online.
Here are some of the nicest and main places I went to, with links to online pictures of some of them:

- Napoli or Naples.
- Ischia, a beautiful island that I totally fell in love with.
- Palermo.
- The
  [cathedral of Monreale](http://www.paradoxplace.com/Perspectives/Sicily%20&%20S%20Italy/Montages/Sicily/Palermo/Monreale%20Cathedral.htm).
- Cefalu.
- [Stromboli](http://www.swisseduc.ch/stromboli/volcano/photos/index-de.html), an island volcano ... I spent the night on top of it and watched the regular eruptions.
- Cathania.
- Etna, a volcano, where I walked in snow over the lava fields for several kilometres ... I did not see the erupting part due to clouds, though.
- The Amalfian coast, with the
  [Sentiero degli Dei](http://www.fotoeweb.it/sorrentina/Sentiero%20degli%20Dei.htm),
  or Path of the Gods, the most beautiful path I ever walked
  ([photo video](http://www.youtube.com/watch?v=OoYb7P4ioZs)).

It was a really wonderful trip, I met wonderful people, and I will definitely be returning to southern Italy again.