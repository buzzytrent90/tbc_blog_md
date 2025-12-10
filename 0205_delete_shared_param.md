---
post_number: "0205"
title: "Deleting a Shared Parameter"
slug: "delete_shared_param"
author: "Jeremy Tammik"
tags: ['elements', 'parameters', 'python', 'references', 'revit-api', 'selection', 'walls']
source_file: "0205_delete_shared_param.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0205_delete_shared_param.html"
---

### Deleting a Shared Parameter

Here is another question handled by Joe Ye, on removal of shared parameters.

**Question:**
I'm creating a shared parameter and adding it to a parameter group.
I then bind it as an instance parameter.
I would like to be able to test and remove the binding if a parameter of the same name exists.
For instance, I am creating a parameter called XYZ\_TEST\_PARAMETER.
If I run the code twice, I end up with two parameters with the same name.
I fully understand that they have different GUIDs, but need a mechanism to remove the existing.

**Answer:**
Shared parameter can be removed using the method Document.ParameterBindings.Remove( Definition ).
The Definition object can be retrieved from the definition file, which can be obtained through the sequence
Application.OpenSharedParameterFile > DefinitionFile > DefinitionGroup > Definition.
You can also get a parameter definition from the parameter itself using the Parameter.Defintion property.

**Question:**
I would prefer not to require the shared parameter file to be present.
I need to be able to delete the bound parameter making no reference to the shared parameter file.

**Answer:**
I just tested removal of a shared parameter bound to walls.
The definition is retrieved from the Parameter.Definition, and Revit does not have a definition text file defined.
This works well in Revit Architecture 2010, so a shared parameter can indeed be removed without the definition text file.
Here is the code I used, which assumes that walls have a shared parameter named APIParameter bound to them:
```python
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string messages,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  doc.Selection.StatusbarTip
    = "Please select a wall to remove "
      + "the shared parameter bound to it.";

  doc.Selection.PickOne();

  Element wall = null;

  foreach( Element e in doc.Selection.Elements )
  {
    wall = e;
    break;
  }

  if( wall != null )
  {
    Parameter par = wall.get\_Parameter(
      "APIParameter" );

    Definition def = par.Definition;

    doc.ParameterBindings.Remove( def );
  }
  return IExternalCommand.Result.Succeeded;
}
```

**Question:**
I think we're getting there.
I want to do this with no user interaction.
The only thing I know is the parameter name.
I do not know what element type it's been bound to.

**Answer:**
Knowing only the shared parameter name is obviously sufficient to remove the parameter binding if the name is unique.
All the binding information is stored in Document.ParameterBindings collection, and we can access all the binding by iterating over that.

We can get the key and the binding for each item.
The key of the binding is the parameter definition.
We can compare the definition name with your target parameter name.
If they are equal, you have found the parameter you are searching for.
Please note that several shared parameters with the same name can live together in a document.
If this is the case in your situation, you will have to check for more information such as the category of each binding.

Here is the code for the Execute method of an external command which iterates over all parameter bindings and finds and removes the target shared parameter:
```python
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string messages,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  BindingMap bm = doc.ParameterBindings;

  DefinitionBindingMapIterator it
    = bm.ForwardIterator();

  while( it.MoveNext() )
  {
    Definition def = it.Key;

    if( def.Name.Equals( "APIParameter" ) )
    {
      bm.Remove( def );
    }
  }
  return IExternalCommand.Result.Succeeded;
}
```

There is one important thing to note here:
For the sake of simplicity, the target parameter is being removed here whilst the iteration is still going on.
This will not necessarily cause problems in every case, but in general, it should be avoided.
A better approach might be to perform the iteration first, collect all the entries you want to delete, and then
perform the actual deletion after the iteration has terminated.

Thank you very much Joe for these answers!