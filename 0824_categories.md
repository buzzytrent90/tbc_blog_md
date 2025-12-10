---
post_number: "0824"
title: "Categories Collection Item Accessor Fails"
slug: "categories"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'python', 'references', 'revit-api', 'schedules', 'selection', 'views', 'walls', 'windows']
source_file: "0824_categories.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0824_categories.html"
---

### Categories Collection Item Accessor Fails

The document settings categories collection Item property returns null for certain built-in categories, and even throws an exception for certain others.
You should be aware of this.

I mentioned a related issue back in the Revit 2011 timeframe, describing
[The Building Coder CmdCategories sample command](http://thebuildingcoder.typepad.com/blog/2010/05/categories.html).

Still, this is a bit unexpected and has caused issues for several developers.

Here are some examples:

**Question 1:** For some reason I cannot obtain the category for stacked walls, e.g. using
```csharp
Dim cat As Category = Doc.Settings.Categories
.Item(BuiltInCategory.OST\_StackedWalls)
```

This call returns nothing even though I have stacked walls in my document.
I can also find the BuiltinCategory.OST\_StackedWalls using RevitLookup.

Should I be looking somewhere else?

**Question 2:** I am adding several parameters through
```csharp
doc.ParameterBindings.Insert(
information, caseTying, bipgroup );
```

Sometimes I get an error, even though the parameters all seem OK.

The stack trace says NullReferenceException in BindingMap.Insert.

What is causing this?

Here is a code snippet demonstrating the problem:
```csharp
  DefinitionFile sharedParametersFile
    = app.OpenSharedParameterFile();

  DefinitionGroup group = sharedParametersFile
    .Groups.Create( "Reinforcement" );

  Definition def = group.Definitions.Create(
    "ReinforcementParameter", ParameterType.Text );

  List<BuiltInCategory> bics
    = new List<BuiltInCategory>();

  //bics.Add(BuiltInCategory.OST\_AreaRein);
  //bics.Add(BuiltInCategory.OST\_FabricAreas);
  //bics.Add(BuiltInCategory.OST\_FabricReinforcement);
  //bics.Add(BuiltInCategory.OST\_PathRein);
  //bics.Add(BuiltInCategory.OST\_Rebar);

  bics.Add( BuiltInCategory
    .OST\_IOSRebarSystemSpanSymbolCtrl );

  CategorySet catset = new CategorySet();

  foreach( BuiltInCategory bic in bics )
  {
    catset.Insert(
      doc.Settings.Categories.get\_Item( bic ) );
  }

  InstanceBinding binding
    = app.Create.NewInstanceBinding( catset );

  doc.ParameterBindings.Insert( def, binding,
    BuiltInParameterGroup.PG\_CONSTRUCTION );
```

**Answer:** The first issue is due to OST\_StackedWalls being marked 'Hidden to User', causing certain internal methods to return false.
As a final result, the Categories Item accessor returns null.

'Hidden to User' means that the user interface to Object Styles, Visibility/Graphics, and Export Layers will not include these categories.

It does not mean that the categories are hidden everywhere in the product.
Some features get a specific category from an element and display its name.
For example, the status prompt displays the category of the element that the mouse is hovering over, and the Selection Filter dialog displays the categories of selected elements.
This works because other methods return all categories regardless of the 'Hidden to User' flag.

Aside from OST\_StackedWalls, here are some other categories that may correspond to real elements that users can see, select, and whose category name may appear:

- OST\_Reveals- OST\_ArcWallRectOpening- OST\_SWallRectOpening- OST\_SitePropertyLineSegment- OST\_RvtLinks- OST\_LegendComponents- OST\_ConnectorElem

Some features also display lists of categories that don't come from the CategoryTableInterface methods.
Schedules, project parameters, the Family Category and Parameters dialog are examples.
Typically those can also include 'Hidden to User' categories.
For example, OST\_SitePropertyLineSegment is a schedulable category despite being 'Hidden to User'.

In some cases it may be incorrect for the API to prevent people from working with 'Hidden to User' categories.
Of course any APIs for category-related features should have proper validation so that people can only use appropriate categories. For example, APIs for category visibility, project parameters, and schedules should restrict which categories can be used for those features, just like the UI does. We have to be careful about exposing 'Hidden to User' categories without checking other APIs for proper validation. For example, the project parameters API may allow more categories that it should.

I expanded The Building Coder samples CmdCategories external command to determine and display a list of all the built-in categories causing the Categories.Item property to either return null or throw an exception.
Here is the code that demonstrates that:
```python
  Array bics = Enum.GetValues(
    typeof( BuiltInCategory ) );

  int nBics = bics.Length;

  Debug.Print( "{0} built-in categories and the "
    + "corresponding document ones:", nBics );

  Category cat;
  string s;

  List<BuiltInCategory> bics\_null
    = new List<BuiltInCategory>();

  List<BuiltInCategory> bics\_exception
    = new List<BuiltInCategory>();

  foreach( BuiltInCategory bic in bics )
  {
    try
    {
      cat = categories.get\_Item( bic );

      if( null == cat )
      {
        bics\_null.Add( bic );
        s = "<null>";
      }
      else
      {
        s = string.Format( "{0} ({1})",
          cat.Name, cat.Id.IntegerValue );
      }
    }
    catch( Exception ex )
    {
      bics\_exception.Add( bic );

      s = ex.GetType().Name + " " + ex.Message;
    }
    Debug.Print( "  {0} --> {1}",
      bic.ToString(), s );
  }

  int nBicsNull = bics\_null.Count;
  int nBicsException = bics\_exception.Count;

  TaskDialog dlg = new TaskDialog(
    "Hidden Built-in Categories" );

  s = string.Format(
    "{0} categories obtained from the Categories collection;\r\n"
    + "{1} built-in categories;\r\n"
    + "{2} built-in categories retrieve null result;\r\n"
    + "{3} built-in categories throw an exception:\r\n",
    nCategories, nBics, nBicsNull, nBicsException );

  Debug.Print( s );

  dlg.MainInstruction = s;

  s = bics\_exception
    .Aggregate<BuiltInCategory, string>(
      string.Empty,
      ( a, bic ) => a + "\n" + bic.ToString() );

  Debug.Print( s );

  dlg.MainContent = s;

  dlg.Show();
```

This is the resulting output in an empty architectural project reporting the count of document categories, built-in categories, and built-in categories for which a document category cannot be retrieved, either because the call the Categories.Item returns null or throws an exception:

![Document and built-in category counts](img/bic_hidden.png)

To spell it out in full text:

- 259 categories obtained from the Categories collection;- 897 built-in categories;- 358 built-in categories retrieve null result;- 12 built-in categories throw an exception:
        - OST\_FurnitureHiddenLines- OST\_SplitterProfile- OST\_HostTemplate- OST\_MassFaceSplitter- OST\_MassCutter- OST\_ZoningEnvelope- OST\_StairsSupports- OST\_InstanceDrivenLineStyle- OST\_CurtainGridsSystem- OST\_StairsStringerCarriage- OST\_IOSNotSilhouette- OST\_InvisibleLines

The complete lists of built-in categories for which no corresponding document category can be retrieved are logged to the Visual Studio debug output window.
They obviously depend on the document that you run this command in.

You should probably run it yourself in the documents you are working with to be aware of what is going on.
In any case, all code accessing the Categories.Item property with untested built-in categories needs to implement appropriate handling of these conditions.

Here is the updated
[version 2013.0.99.1](zip/bc_13_99_1.zip) of
The Building Coder samples including the expanded version of the CmdCategories external command displaying the list of hidden built-in categories.

**Addendum:** Saikat presented another example of an inaccessible category, the
[Profiles category OST\_ProfileFamilies](http://adndevblog.typepad.com/aec/2012/09/accessing-profiles-category-while-omniclass-code-and-description-access.html).

#### Brute Force Computing

I stumbled across this interesting article on
[brute force computing](http://bits.blogs.nytimes.com/2012/09/11/the-brute-force-computing-revolution) explaining
one interesting aspect of cloud based computing and access to huge amounts of computing power.

One fact I was totally unaware of, and that had me a bit surprised in the past year or two, is that "brute force computing is behind Google's successful language translation.
By comparing thousands of Web pages in different languages to find patterns, in one year Google was able to discern and refine translation better than linguistic theorists had been able to do with their fancy programs for years."

As you of course know, Autodesk offers the
[Autodesk 360](https://360.autodesk.com) cloud-based
platform providing access to storage, a collaboration workspace, and a fast-growing number of cloud services to help improve design, visualisation, simulation, and work sharing.

#### Blending Profiles with Different Vertex Counts

Arjen de Pater raised a
[question](http://thebuildingcoder.typepad.com/blog/2010/11/newblend-sample.html?cid=6a00e553e168978833017744a73173970d#comment-6a00e553e168978833017744a73173970d) on
using the
[NewBlend method](http://thebuildingcoder.typepad.com/blog/2010/11/newblend-sample.html) that
Mikako Harada just addressed:

**Question:** I want to create a blend with 8 curves in the BottomProfile and 4 curves in the TopProfile.

When I try this the Document.FamilyCreate.NewBlend method throws an ApplicationException saying "Unexpected internal error: code 1. at Autodesk.Revit.Creation.FamilyItemFactory.NewBlend(Boolean isSolid, CurveArray topProfile, CurveArray baseProfile, SketchPlane sketchPlane)".

How can I create such a blend?

**Answer:** There are some points you need to pay attention to.
These points are documented in the page explaining the NewBlend method in Revit API help file RevitAPI.chm file.
It also includes sample code to create a blend form:

1. The two profiles should be parallel.- Each of the profiles must be planar.- If the profile shape is circle or ellipse, it must be split into at least two segments, so that Revit can find vertices to use for mapping the blend.

Once that is clear, you should definitely ensure that you can create the same shape through the user interface, as recommended for
[debugging form creation](http://thebuildingcoder.typepad.com/blog/2009/07/debug-geometric-form-creation.html).
If that fails, then it will fail through the API as well.
It may provide you with a more informative error message, though.

**Response:** Yes, I can create the blend in the UI.

The problem is the different number of curves in the BottomProfile and TopProfile because the NewBlend method lacks a parameter for the VertexConnectionMap.

**Answer:** Ah.
Well, here are two
[NewSweptBlend](http://thebuildingcoder.typepad.com/blog/2010/03/newsweptblend.html)
[samples](http://adndevblog.typepad.com/aec/2012/08/newsweptblend-code-sample.html), which may or may not prove useful.

However, Mikako hopefully provided the ultimate solution by demonstrating
[blending two profiles with different top and bottom vertex counts](http://adndevblog.typepad.com/aec/2012/09/blending-two-profiles-with-different-topbottom-vertex-counts-.html).

#### Creating a ViewPlan in a Family Document

On the topic of Revit API blog posts by my colleagues on the
[AEC DevBlog](http://adndevblog.typepad.com/aec),
here is another interesting topic brought up by Saikat Bhattacharya:

You can
[create a new ViewPlan in a family document](http://adndevblog.typepad.com/aec/2012/09/creating-a-new-viewplan-in-a-family-document-using-the-revit-api.html)  using
the View.Duplicate method taking a ViewDuplicateOption, something that I was not previously aware of.

You cannot use the NewViewPlan method to achieve this in a family document, because it throws an InvalidOperationException,
as Saikat explains very clearly.