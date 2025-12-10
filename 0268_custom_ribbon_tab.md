---
post_number: "0268"
title: "Custom Ribbon Tab"
slug: "custom_ribbon_tab"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'references', 'revit-api', 'windows']
source_file: "0268_custom_ribbon_tab.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0268_custom_ribbon_tab.html"
---

### Custom Ribbon Tab

I am still on tour presenting at the Western European DevDays conferences, and with no time for blogging or responding to comments.
In fact, I have almost no time for anything at all except presenting, meeting and discussing with participants during the day, and getting from one city to the next in the evenings.
Right now I am sitting in the airport waiting for a plain to Milano.
Back to Bella Italia, albeit for less than twenty-four hours.

I was hoping to find time to prepare a few blog posts in advance for the coming weeks, when I will be gone on holidays and vacation.
Friday is supposed to be my last working day this year, and I am starting to wonder whether I will be able to just walk away from all the unresolved issues and leave them to lie until next year.

Anyway, in a sleepless hour in between I noticed that Augusto Gonçalves responded once again to a question that has already come up a few times in the past, so his interesting result is well worth while presenting, even if it is not directly useful in the context of the Revit API.
It deals with the frequent question on whether it is possible to add your own ribbon tab to the Revit user interface.

**Question:** I expect the answer to this is no, but I thought I would at least ask anyway.
Is it possible to create a new ribbon tab in Revit, similar to AutoCAD 2010, or are panels within the Add-Ins tab and items within those panels the only ribbon objects that can be accessed and created from a Revit add-in?

**Answer:** There is no documented support for this in the Revit API that I am aware of.
There are however a couple of undocumented and unsupported .NET assemblies that can be used to access the Revit ribbon in an unsupported way.
The functionality they provide can even be used to add your own custom panel to the Revit ribbon.
What you cannot do, however, is create the context and data required to invoke a standard Revit external command.
We have implemented a sample that creates an own custom tab and adds it to the Revit ribbon.
It displays a command button which can be used to invoke Revit independent functionality.

The functionality to create a new custom panel and add a command button to it is provided by classes in the Autodesk.Windows namespace.
These classes have no knowledge of Revit and its API, and we have not found any way to access the command data required to invoke an external command and make use of the Revit API from such a button.
As long as you are happy just doing .NET stuff completely independently of Revit, you can make use of this.
This functionality is unsupported, and to be used at your own risk, of course.

In addition to the RevitAPI.dll assembly providing access to the Revit API functionality, we reference two other undocumented .NET assemblies provided by Revit, which also live in the Revit Program folder: AdWindows.dll and UIFramework.dll.
These in turn require us to reference some other .NET framework functionality, so we end up with the following list of references.
As always, we need to remember to set the 'Copy Local' flag to 'False' on the three assemblies referenced from the Revit folder:

- AdWindows- PresentationCore- PresentationFramework- RevitAPI- System- System.Windows.Forms- UIFramework- WindowsBase

We implement an external application named App which makes use of the ribbon functionality provided by the AdWindows and UIFramework assemblies to add its own custom tab and panel to the existing Revit ribbon.
The custom panel displays one ribbon button which invokes a command.

The command needs to implement the System.Windows.Input.ICommand interface.
Note that this definition is completely independent of the Revit external command interface, and actually that is the main problem with our custom ribbon tab: while we can add a tab with its panel and button invoking a command, this is not a standard Revit command, and we have no way to connect it with Revit or make proper use of the Revit API within the command implementation.
It can be used to invoke Revit-independent functionality.

Here is the definition of the command implementation derived from ICommand and its Execute method, which in turn invokes the Execute method of an external Revit command implementation AddRibbonTab.Command, but supplies it with a null ExternalCommandData instance:
```csharp
public class AdskCommandHandler
  : System.Windows.Input.ICommand
{
  string AssemblyName
  {
    get;
    set;
  }

  string ClassName
  {
    get;
    set;
  }

  public AdskCommandHandler(
    string assemblyName,
    string className )
  {
    AssemblyName = assemblyName;
    ClassName = className;
  }

  public event EventHandler CanExecuteChanged;

  public bool CanExecute( object a )
  {
    return true;
  }

  public void Execute( object a )
  {
    System.Reflection.Assembly assembly
      = System.Reflection.Assembly.LoadFrom(
        AssemblyName );

    IExternalCommand command
      = assembly.CreateInstance(
        ClassName ) as IExternalCommand;

    Debug.Print(
      "AdskCommandHandler.Execute command invoked: "
      + "assembly {0}, class {1}",
      AssemblyName, ClassName );

    ExternalCommandData commandData = null;
    string message = string.Empty;
    ElementSet elements = null;

    IExternalCommand.Result r
      = command.Execute( commandData,
        ref message, elements );
  }
}
```

Here is the OnStartup method of the external application creating the custom ribbon tab.
Note that the RibbonPanel created here is an Autodesk.Windows.RibbonPanel instance, not an Autodesk.Revit one:
```csharp
public IExternalApplication.Result OnStartup(
  ControlledApplication a )
{
  // create new ribbon button:

  RibbonButton button = new RibbonButton();
  button.Text = "My Button";
  button.ShowText = true;
  button.CommandHandler = new AdskCommandHandler(
    "AddRibbonTab.dll", "AddRibbonTab.Command" );

  // create new ribbon panel:

  RibbonPanelSource source = new RibbonPanelSource();
  source.Title = "My Panel";
  source.Items.Add( button );

  RibbonPanel panel = new RibbonPanel();
  panel.Source = source;

  // create custom ribbon tab:

  RibbonTab tab = new RibbonTab();
  tab.Id = "MY\_TAB\_ID";
  tab.Title = "My Custom Tab";
  tab.IsVisible = true;

  // access Revit ribbon control and add custom tab:

  RibbonControl control
    = UIFramework.RevitRibbonControl.RibbonControl;

  control.Tabs.Add( tab );
  tab.Panels.Add( panel );

  return IExternalApplication.Result.Succeeded;
}
```

Here is the resulting custom ribbon tab with its panel and command button displayed in Revit MEP 2010:

![Custom ribbon tab](img/custom_ribbon_tab.png)

Here is the complete
[AddRibbonTab](zip/AddRibbonTab.zip) source code and Visual Studio solution.

Many thanks to Augusto for exploring and discovering this undocumented functionality and providing the surprising sample code!