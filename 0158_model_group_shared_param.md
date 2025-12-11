---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.7
content_type: qa
optimization_date: '2025-12-11T11:44:13.459347'
original_url: https://thebuildingcoder.typepad.com/blog/0158_model_group_shared_param.html
post_number: 0158
reading_time_minutes: 10
series: general
slug: model_group_shared_param
source_file: 0158_model_group_shared_param.htm
tags:
- csharp
- doors
- elements
- filtering
- parameters
- references
- revit-api
- walls
title: Model Group Shared Parameter
word_count: 1938
---

### Model Group Shared Parameter

Several people have recently been struggling with adding shared parameters to model groups, including myself.
Since I had such a hard time finally setting up a reliable working solution, I thought I would share some of the steps and pitfalls with you.
These are some of the issues that made it hard for me:

- You cannot add **visible** shared parameters to a model group, only **invisible** ones.
- You cannot retrieve the model group category from the Document.Settings.Categories property.
- Various error messages are misleading.

When you finally succeed in circumnavigating these obstacles, all works well.
In the code sample below, I show how to set up shared parameters for several different categories:

- The standard category Doors.
- The system category Walls.
- The Model Groups category.
- The Model Lines category.

Note that this is one of the few cases where something can be done through the API that is not possible in the user interface.
In the user interface, shared parameters cannot be defined for model groups or model lines.
This is reflected in the Category.AllowsBoundParameters property, which indicates if a category can
have visible shared or project parameters.
If it is false, the category may not be bound to visible shared parameters using the BindingMap.
Please note that non-user-visible parameters can still be bound to these categories.

We already made several forays into the realm of shared parameters in previous posts, e.g. the
[creation of a new shared parameter](http://thebuildingcoder.typepad.com/blog/2008/11/defining-a-new-parameter.html)
and how to determine whether shared parameters can be added to certain elements, such as an
[inserted DWG file](http://thebuildingcoder.typepad.com/blog/2008/11/adding-a-shared-parameter-to-a-dwg-file.html)
and an
[RFA file](http://thebuildingcoder.typepad.com/blog/2009/06/adding-a-shared-parameter-to-an-rfa-file.html).
Performance issues were also addressed, both for the
[modification of a parameter value](http://thebuildingcoder.typepad.com/blog/2008/12/parameter-modification-performance.html)
and
[parameter binding](http://thebuildingcoder.typepad.com/blog/2009/03/parameter-binding-performance.html).

### Accessing the Model Groups Category

To test attaching a shared parameter to model groups, I created a sample model group through the Revit user interface by selecting Annotate > Detail Group > Create Group, ensuring that the 'Model' radio button is toggled on, and selecting some model elements to add to the group.

When I look at the resulting group with RvtMgdDbg using Add-Ins > RvtMgdDbg > Snoop Db... > Group, selecting the model group I just created, I can verify that its category name is 'Model Groups' and the built-in category is OST\_IOSModelGroups.

I tried to use this value in the
[Revit API introduction labs](zip/rac_labs_2009-06-24.zip)
Lab4\_3\_1\_CreateAndBindSharedParam by setting
```csharp
static public BuiltInCategory Target
= BuiltInCategory.OST\_IOSModelGroups;
```

Unfortunately, when I do so, the step which actually creates the parameter binding between the shared parameter definition and the category set containing the designated target category throws an exception:
```csharp
doc.ParameterBindings.Insert(
fireRatingParamDef, binding );
```

The exception message states that "Object reference not set to an instance of an object."

In later attempts, I discovered that we cannot obtain the model groups category from the Document.Settings.Categories collection at all.
The typical way to access it would be using
```csharp
doc.Settings.Categories.get\_Item( BuiltInCategory.OST\_IOSModelGroups );
```

This returns null.
An alternative method which is not recommended is to use the language dependent category name "Model Groups" as a target string instead of the language independent built-in category enumeration value.
Doing so throws an exception, SystemInvalidOperationException "Operation is not valid due to the current state of the object."

Yet another way to obtain the model group category is to query it from an existing model group in the project.
This is implemented in the following helper method GetCategory, which returns a valid category for a given built-in enumeration value.
Note that in the case of model groups, it requires the existence of at least one such group in the project to work.
This limitation could be removed by inserting a dummy group on the fly, querying it for its category, and then deleting it again.
Also note that for all other built-in categories except the model groups, we simply access the document categories collection in the normal way.
There may actually be other built-in categories which require some kind of special handling as well, but I am currently not aware of any:
```csharp
Category GetCategory( Application app, BuiltInCategory target )
{
  Document doc = app.ActiveDocument;
  Category cat = null;

  if( target.Equals( BuiltInCategory.OST\_IOSModelGroups ) )
  {
    //
    // determine model group category:
    //
    Autodesk.Revit.Creation.Filter cf
      = app.Create.Filter;

    List<Element> modelGroups
      = new List<Element>();

    Filter fType = cf.NewTypeFilter(
      typeof( Group ) );

    //Filter fType = cf.NewTypeFilter( // this works as well
    //  typeof( GroupType ) );

    Filter fCategory = cf.NewCategoryFilter(
      BuiltInCategory.OST\_IOSModelGroups );

    Filter f = cf.NewLogicAndFilter(
      fType, fCategory );

    if( 0 == doc.get\_Elements( f, modelGroups ) )
    {
      Util.ErrorMsg( "Please insert a model group." );
      return cat;
    }
    else
    {
      cat = modelGroups[0].Category;
    }
  }
  else
  {
    try
    {
      cat = doc.Settings.Categories.get\_Item( target );
    }
    catch( Exception ex )
    {
      Util.ErrorMsg( string.Format(
        "Error obtaining document {0} category: {1}",
        target.ToString(), ex.Message ) );
      return cat;
    }
  }
  if( null == cat )
  {
    Util.ErrorMsg( string.Format(
      "Unable to obtain the document {0} category.",
      target.ToString() ) );
  }
  return cat;
}
```

### Creating the Shared Parameter

Once we have determined the categories we wish to bind the shared parameter to, the rest of the steps are pretty straightforward and remain unchanged from the existing labs sample code.

We need to ensure that the newly created shared parameter is set to invisible for model groups, otherwise the binding will fail.
This applies to all categories whose property AllowsBoundParameters return false.

The error message produced by attempting to bind a visible shared parameter to a category whose AllowsBoundParameters is set false is rather misleading, because it simply states that "Binding the parameter to the category Model Groups is not allowed".

This message may also be generated when a parameter by the same name already exists.
Be careful to remove all potentially conflicting shared parameters before running any tests.
I now check through the user interface using Manage > Shared Parameters and select Parameters: > [group name] > Delete > Yes > OK to remove any previously created definitions from the shared parameter definition file before relaunching my test command to add a new one again.

Here is the CreateSharedParameter method that I implemented to create a new shared parameter for a given category:
```csharp
bool CreateSharedParameter(
  Application app,
  Category cat,
  int nameSuffix )
{
  Document doc = app.ActiveDocument;
  //
  // get or set the current shared params filename:
  //
  string filename
    = app.Options.SharedParametersFilename;

  if( 0 == filename.Length )
  {
    string path = \_filename;
    StreamWriter stream;
    stream = new StreamWriter( path );
    stream.Close();
    app.Options.SharedParametersFilename = path;
    filename = app.Options.SharedParametersFilename;
  }
  //
  // get the current shared params file object:
  //
  DefinitionFile file
    = app.OpenSharedParameterFile();

  if( null == file )
  {
    Util.ErrorMsg(
      "Error getting the shared params file." );

    return false;
  }
  //
  // get or create the shared params group:
  //
  DefinitionGroup group
    = file.Groups.get\_Item( \_groupname );

  if( null == group )
  {
    group = file.Groups.Create( \_groupname );
  }

  if( null == group )
  {
    Util.ErrorMsg(
      "Error getting the shared params group." );

    return false;
  }
  //
  // set visibility of the new parameter:
  //
  bool visible = cat.AllowsBoundParameters;
  //
  // get or create the shared params definition:
  //
  string defname = \_defname + nameSuffix.ToString();

  Definition definition = group.Definitions.get\_Item(
    defname );

  if( null == definition )
  {
    definition = group.Definitions.Create(
      defname, \_deftype, visible );
  }
  if( null == definition )
  {
    Util.ErrorMsg(
      "Error in creating shared parameter." );

    return false;
  }
  //
  // create the category set containing our category for binding:
  //
  CategorySet catSet = app.Create.NewCategorySet();
  catSet.Insert( cat );
  //
  // bind the param:
  //
  try
  {
    Binding binding = app.Create.NewInstanceBinding(
      catSet );
    //
    // we could check if it is already bound,
    // but it looks like insert will just ignore
    // it in that case:
    //
    doc.ParameterBindings.Insert( definition, binding );
  }
  catch( Exception ex )
  {
    Util.ErrorMsg( string.Format(
      "Error binding shared parameter to category {0}: {1}",
      cat.Name, ex.Message ) );
    return false;
  }
  return true;
}
```

### Putting it Together

With the two methods listed above in place, the rest of the code for the new external command CmdCreateSharedParams is pretty minimal.
First, we define a couple of pretty arbitrary constants for the shared parameters filename, group name, parameter name prefix and type:
```csharp
const string \_filename = "C:/tmp/SharedParams.txt";
const string \_groupname = "The Building Coder Parameters";
const string \_defname = "SP";
ParameterType \_deftype = ParameterType.Number;
```

What element types are we interested in?

- We start with the standard SDK FireRating example, whcih uses BuiltInCategory.OST\_Doors.- We use BuiltInCategory.OST\_Walls to demonstrate that the same technique works with system families just as well as with standard ones.- To attach shared parameters to
      [inserted DWG files](http://thebuildingcoder.typepad.com/blog/2008/11/adding-a-shared-parameter-to-a-dwg-file.html),
      which generate their own category on the fly, we can also identify the category by category name instead of built-in category enumeration.
      This is commented out here, so we do not have to deal with the additional complexity of mixing strings and built-in enumeration values.- We can attach shared parameters to model groups.
        Unfortunately, this does not work in the same way as the others, because we cannot retrieve the category from the document Settings.Categories collection, neither using the built-in category enumeration value nor the category name.
        We implemented the special switch for that case in the GetCategories method presented above, so that we can obtain the category from an existing instance of a group instead.- Model lines.

Here is the list of these built-in categories that we use to drive the shared parameter creation loop, followed by the implementation of the command mainline and creation loop in the external command Execute method:
```csharp
BuiltInCategory[] targets = new BuiltInCategory[] {
  BuiltInCategory.OST\_Doors,
  BuiltInCategory.OST\_Walls,
  //"Drawing1.dwg", // inserted DWG file
  BuiltInCategory.OST\_IOSModelGroups,
  BuiltInCategory.OST\_Lines // model lines
};
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  int i = 0;

  foreach( BuiltInCategory target in targets )
  {
    Category cat = GetCategory( app, target );
    CreateSharedParameter( app, cat, ++i );
  }
  return CmdResult.Succeeded;
}
```

So you can actually add a shared parameter to a model group after all. Hooray!

Here is
[version 1.1.0.36](zip/bc11036.zip)
of the complete Visual Studio solution with the new command.

I am also providing the current version of the
[Revit API introduction labs](zip/rac_labs_2009-06-24.zip)
here.
It includes an updated implementation of Lab 4-3-1, which now also includes this workaround to support adding a shared parameter to the model group category.

### First Response

Here is some enthusiastic response from Henrik Bengtsson of
[Lindab](http://www.lindab.se)
after trying this out:

Now it is much clearer. And it works like a rocket !!!!!!!!

I simply implemented your function for retrieving the Category via the doc.Elements(Filter, List), and included a small IF statement for ordinary cases and the Model Groups case. One good idea was to include the Visibility settings in my shared parameters class, so that it takes care of the visibility setting depending on the AllowsBoundParameters property no matter what the user wants to create when he uses the class... Thanks again for the smashing answer...

You have really done a great job here!!!!

**Question:**
Are there any other families that you know can be handled via the GetCategory solution, like Model Groups?

**Answer:**
No, I am currently not aware of any other categories that require the special handling I implemented for Model Groups, but possibly more would crop up with extensive testing. You could actually simply loop through all the built-in category enumeration values and try to retrieve a category from the document categories collection for each.