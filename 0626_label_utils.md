---
post_number: "0626"
title: "Built-in Parameter Name and LabelUtils"
slug: "label_utils"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api']
source_file: "0626_label_utils.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0626_label_utils.html"
---

### Built-in Parameter Name and LabelUtils

Over a year ago, I mentioned the
[LabelUtils class](http://thebuildingcoder.typepad.com/blog/2010/04/user-visible-enumeration-value-labels.html) introduced
in the Revit 2011 API.
Here is another question that came up and shows that it might be useful to point it out again:

**Question:** I need access to the definition of an internal (built-in) parameter, because I want to retrieve its name via Autodesk.Revit.DB.Definition.Name.
I know there's an overload of the Element.Parameter property which takes a BuiltInParameter argument.
The problem is that there is no element available at the time when I need to obtain the parameter name.
So I am looking for something similar to Document.Settings.Categories which allows global access to built-in categories.
I've tried to find something similar via Document.ParameterBindings, but this map seems to contain only external definitions, i.e. definitions of shared parameters, not of built-in ones.

**Answer:** Is this what you are after?
```csharp
  string s = string.Empty;

  foreach( BuiltInParameter bip in
    Enum.GetValues( typeof( BuiltInParameter ) ) )
  {
    s += "\r\n" + bip.ToString();
  }
  TaskDialog.Show( "Parameter Names", s );
```

It retrieves the string values of all the built-in parameter enumeration values.

**Reponse:** No, I do not want to use the BuiltInParameter.ToString method, since the string is presented to the user.
Therefore, I would like to use the Parameter.Definition.Name instead, which is user friendlier and also localized.

To access the definition name, I would need to have an element with all the parameters I am interested in attached to it.
If I had such an element 'e', I could use the following code to create a mapping from built-in parameters to the corresponding user visible names:
```csharp
  Element e;

  Dictionary<BuiltInParameter, string> mapBipToName
    = new Dictionary<BuiltInParameter, string>();

  foreach( BuiltInParameter bip in
    Enum.GetValues( typeof( BuiltInParameter ) ) )
  {
    // translate built-in enum to parameter name

    Parameter p = e.get\_Parameter( bip );

    if( null != p )
    {
      mapBipToName.Add( bip, p.Definition.Name );
    }
  }
```

However, I do not have such an element available.

**Answer:** Please have a look at the LabelUtils class, especially its GetLabelFor method taking a BuiltInParameter argument.

It returns the user-visible name for a given built-in parameter.
The name is obtained in the current Revit language.

**Reponse:** Exactly what I was looking for! Thank you.