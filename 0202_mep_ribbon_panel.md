---
post_number: "0202"
title: "MEP Sample Ribbon Panel"
slug: "mep_ribbon_panel"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'filtering', 'levels', 'revit-api', 'schedules', 'windows']
source_file: "0202_mep_ribbon_panel.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0202_mep_ribbon_panel.html"
---

### MEP Sample Ribbon Panel

I am preparing our upcoming Revit MEP API webcast to be held on Thursday August 27th.
To register, you can go to the Autodesk Developer Network (ADN) DevTech API
[training schedule](http://www.adskconsulting.com/adn/cs/api_course_sched.php)
and filter for Revit MEP API.

One thing I did today was to update our MEP sample external application from the Revit 2009 menu-based user interface to a 2010-style custom ribbon panel.

Here is the code for the AddMenu method that was being used in Revit 2009 to create a custom menu:
```csharp
private static void AddMenu( ControlledApplication app )
{
  const string m = "mep.Cmd"; // namespace and command prefix
  string path = System.Reflection.Assembly.GetExecutingAssembly().Location;
  Autodesk.Revit.MenuItem rootMenu = app.CreateTopMenu( "ME&P API Samples" );
  MenuItem.MenuType mt = MenuItem.MenuType.BasicMenu;
  rootMenu.Append( mt, "&Assign flow to terminals", path, m + "AssignFlowToTerminals" );
  rootMenu.Append( mt, "&Change size", path, m + "ChangeSize" );
  rootMenu.Append( mt, "&Populate CFM per SF on spaces", path, m + "PopulateCfmPerSf" );
  rootMenu.Append( mt, "&Reset demo", path, m + "ResetDemo" );
  rootMenu.Append( MenuItem.MenuType.SeparatorMenu );
  rootMenu.Append( mt, "Electrical &System Browser", path, m + "ElectricalSystemBrowser" );
  rootMenu.Append( mt, "Electrical &Hierarchy", path, m + "ElectricalHierarchy" );
  rootMenu.Append( mt, "Electrical Hierarchy &2", path, m + "ElectricalHierarchy2" );
  rootMenu.Append( mt, "&Unhosted elements", path, m + "UnhostedElements" );
  rootMenu.Append( MenuItem.MenuType.SeparatorMenu );
  rootMenu.Append( mt, "A&bout...", path, m + "About" );
}
```

To see the overly long lines, you can copy and paste to an editor.

AddMenu is called from the external application OnStartup method.

How can this be easily converted to a ribbon panel?

Well, note a couple of basic things here:

- All the command implementation classes reside in the namespace mep and have the prefix "Cmd". Therefore, the variable 'm' is defined as "mep.Cmd" and used to prefix each command class name stem.- All commands reside in the same assembly as the external application, so they all have the same assembly path as the currently executing assembly.- A new top level menu with the text 'MEP API Samples' is created.- The menu items for the individual commands are added one by one, grouped by separators into HVAC, electrical, and the about groups.

Converting this to a ribbon panel is pretty straightforward.
The basics and all possibilities provided by the Revit API for creating your own ribbon panels are demonstrated by the Revit SDK sample Ribbon.

Here is the code for the new AddRibbonPanel method, also called from the external application OnStartup method, which creates an equivalent custom ribbon panel:
```csharp
static void AddRibbonPanel(
  ControlledApplication a )
{
  const string m = "mep.Cmd"; // namespace and command prefix
  string path = Assembly.GetExecutingAssembly().Location;

  string[] text = new string[] {
    "Assign flow to terminals",
    "Change size",
    "Populate CFM per SF on spaces",
    "Reset demo",
    "Electrical System Browser",
    "Electrical Hierarchy",
    "Electrical Hierarchy 2",
    "Unhosted elements",
    "About..."
  };

  string[] classNameStem = new string[] {
    "AssignFlowToTerminals",
    "ChangeSize",
    "PopulateCfmPerSf",
    "ResetDemo",
    "ElectricalSystemBrowser",
    "ElectricalHierarchy",
    "ElectricalHierarchy2",
    "UnhostedElements",
    "About"
  };
  //
  // create three stacked buttons for the
  // HVAC, electrical and about commands respectively:
  //
  RibbonPanel panel = a.CreateRibbonPanel(
    "MEP Sample" );

  PulldownButtonData d1 = new PulldownButtonData(
    "Hvac", "HVAC" );

  d1.ToolTip = "HVAC Commands";

  PulldownButtonData d2 = new PulldownButtonData(
    "Electrical", "Electrical" );

  d2.ToolTip = "Electrical Commands";

  PushButtonData d3 = new PushButtonData(
    classNameStem[8], text[8], path, m + classNameStem[8] );

  d3.ToolTip = "About the HVAC and Electrical MEP Sample.";

  List<RibbonItem> ribbonItems = panel.AddStackedButtons(
    d1, d2, d3 );
  //
  // add subitems to the HVAC and electrical pulldown buttons:
  //
  PulldownButton pulldown;
  PushButton pb;
  int j;

  for( int i = 0; i < 8; ++i )
  {
    j = i < 4 ? 0 : 1;
    pulldown = ribbonItems[j] as PulldownButton;

    pb = pulldown.AddItem( text[i], path,
      m + classNameStem[i] );

    pb.ToolTip = text[i];
  }
}
```

The procedure to create and populate the panel is straightforward:

- We define the external command name prefix and assembly path like before.- The command name stems and menu item texts are stored in two string arrays.- A new ribbon panel is created.- Two pulldown buttons to group the HVAC and electrical commands are added to the panel.- One push button for the about box command is added.- The string arrays of menu item texts and class name stems are iterated.- A ribbon item is created for each command.

Here is the panel as it appears when dragged out of the ribbon to float freely:

![MEP sample custom ribbon panel](img/mep_panel.png)

Here is the complete source code and Visual Studio solution for the
[MEP sample application](zip/mep_20090813.zip)
in its current state.
We will be discussing more details of its implementation in the coming weeks.

Here is a related question that just came up:

#### Ribbon Panel Image Resources

**Question:**
I want to populate my Revit add-in ribbon panel with images which I obtain from the resources with a method like this:

```
internal static System.Drawing.Bitmap door_button {
  get {
    object obj = ResourceManager.GetObject(
      "door_button", resourceCulture);

    return ((System.Drawing.Bitmap)(obj));
  }
}
```

The object type of the image to assign to a button for the ribbon toolbar is System.Windows.Media.ImageSource.
I tried the same approach as above, to provide a method that returns an object of this type and cast to it to the required type before returning.
This does not work, even though the compiler does not complain.
How can I cast between these types, or how else can I retrieve an image of the type System.Windows.Media.ImageSource from the resources, so that I do not have to have the actual image file (png or bitmap) to be distributed with the application?

I see that the method "ResourceManager.GetObject()" returns an object of type "System.Drawing.Bitmap", but the data type to be assigned to a ribbon-button is "BitmapImage".
I do not see any common base types or a constructor for this class that allows an object of type "System.Drawing.Bitmap" or any derived class from it, so I do not see how to convert between the types.
However, investigating the classes in the class browser I have found a method "FromResource" for the Bitmap class, but I have not seen an example on how to use it.

**Answer:**
When I look at the Revit SDK sample Ribbon, I see it using the class System.Windows.Media.Imaging.BitmapImage to define the external application ribbon images.

I do agree that retrieving the bitmap image from the resources is a much cleaner solution than the one used in the Ribbon sample.

System.Windows.Media.ImageSource is an abstract class, so you will never be able to obtain an instance of that from anywhere.

You can open the PresentationCore assembly in the Visual Studio Object Browser, navigate to the class System.Windows.Media.Imaging.BitmapImage, and open its 'Base Types' sub node. That will show you that it is derived from BitmapSource, which is derived from ImageSource.

The Ribbon sample simply uses the BitmapImage constructor to create a BitmapImage which also constitutes an ImageSource.

Now back to your main question, how to convert from the Bitmap returned from the resource manager to the BitmapImage required by the Revit ribbon API.

I googled for "cast Bitmap BitmapImage" and found various useful hits.
Based on these, this is the solution that I ended up with, which I tested in a simple console application:

```csharp
static void Main( string[] args )
{
  Bitmap a = Resource1.Image1;

  MemoryStream ms = new MemoryStream();
  a.Save( ms, ImageFormat.Png );

  BitmapImage b = new BitmapImage();
  b.BeginInit();
  b.StreamSource = ms;
  b.EndInit();
}
```

By the way, another example of using resources and utilising the .NET framework ResourceManager class to manage them is provided by the PowerCircuit Revit SDK sample.