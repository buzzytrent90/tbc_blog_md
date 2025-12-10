---
post_number: "0479"
title: "Launching a Revit Command"
slug: "launch_command"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'revit-api', 'selection', 'transactions', 'walls', 'windows']
source_file: "0479_launch_command.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0479_launch_command.html"
---

### Launching a Revit Command

One question that I frequently hear is how to launch a built-in Revit command programmatically.

We recently discussed a specific case related to this issue in some depth,
[how to close the active document](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html)
and especially pointed out
[why this is risky, undesirable and not supported](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html#2).

To close the active document, we used the SendKeys.SendWait method provided in the .NET framework System.Windows.Forms namespace.
Many of the other attempts that I have seen in the past to launch a Revit command were also based on this method, but unsuccessful.

Here is now a solution that actually does work, suggested by Robert Pleysier of
[ALFA Development](http://www.alfadevelopment.nl) in the Netherlands.
Instead of SendKeys, he uses the native Windows API PostMessage call to send the Windows messages WM\_KEYDOWN and WM\_KEYUP for each required keystroke to the Revit process main window handle.

The reason for Robert to launch a Revit command at all is the requirement to predefine which wall type should be active when the user enters the new wall creation command.
Since there is no API method available to programmatically predefine the current wall type to be used by this command, Robert selects an existing wall with the desired type and then uses the 'Create Similar' command to set up its type as the new default user interface wall type.
If no wall with the desired type exists yet in the model, a temporary wall is created and then deleted again.

Here is the question initially raised by Robert, and his own answer and solution:

**Question:** In the Revit API you can use the function PromptForFamilyInstancePlacement to place family symbols.

Is it possible to place a Wall with this function or another function from code?

I want to change the wall or other system family type by code in the property dialog.

After that I want to activate the Revit wall or stair function by code, so a user can draw his or her own wall or stair with the right wall or stair type.

**Answer:** I solved the issue by using a key press function "cs" for "create similar".

Based on Robert's example code showing how he did this, I implemented a new Building Code sample command CmdPressKey which illustrates his solution.

First of all, here is the helper class Press which accesses the Windows API methods and implements the public method Keys to actually send keystrokes using PostMessage. It requires the namespace System.Runtime.InteropServices:
```csharp
public class Press
{
  [DllImport( "USER32.DLL" )]
  public static extern bool PostMessage(
    IntPtr hWnd, uint msg, uint wParam, uint lParam );

  [DllImport( "user32.dll" )]
  static extern uint MapVirtualKey(
    uint uCode, uint uMapType );

  enum WH\_KEYBOARD\_LPARAM : uint
  {
    KEYDOWN = 0x00000001,
    KEYUP = 0xC0000001
  }

  enum KEYBOARD\_MSG : uint
  {
    WM\_KEYDOWN = 0x100,
    WM\_KEYUP = 0x101
  }

  enum MVK\_MAP\_TYPE : uint
  {
    VKEY\_TO\_SCANCODE = 0,
    SCANCODE\_TO\_VKEY = 1,
    VKEY\_TO\_CHAR = 2,
    SCANCODE\_TO\_LR\_VKEY = 3
  }

  /// <summary>
  /// Post one single keystroke.
  /// </summary>
  static void OneKey( IntPtr handle, char letter )
  {
    uint scanCode = MapVirtualKey( letter,
      ( uint ) MVK\_MAP\_TYPE.VKEY\_TO\_SCANCODE );

    uint keyDownCode = ( uint )
      WH\_KEYBOARD\_LPARAM.KEYDOWN
      | ( scanCode << 16 );

    uint keyUpCode = ( uint )
      WH\_KEYBOARD\_LPARAM.KEYUP
      | ( scanCode << 16 );

    PostMessage( handle,
      ( uint ) KEYBOARD\_MSG.WM\_KEYDOWN,
      letter, keyDownCode );

    PostMessage( handle,
      ( uint ) KEYBOARD\_MSG.WM\_KEYUP,
      letter, keyUpCode );
  }

  /// <summary>
  /// Post a sequence of keystrokes.
  /// </summary>
  public static void Keys( string command )
  {
    IntPtr revitHandle = System.Diagnostics.Process
      .GetCurrentProcess().MainWindowHandle;

    foreach( char letter in command )
    {
      OneKey( revitHandle, letter );
    }
  }
}
```

With this in place, we can go ahead and implement the external command.
It requires two helper methods: GetFirstWallTypeNamed retrieves the appropriate wall type for a given wall type name, and GetFirstWallUsingType retrieves the first wall element encountered in the model making use of a given wall type.
Both of these obviously use filtered element collectors, and both of them even use parameter filters.
GetFirstWallTypeNamed uses the built-in parameter SYMBOL\_NAME\_PARAM and a string equality filter to do its job, i.e. to return the first wall type with the given name:
```csharp
static WallType GetFirstWallTypeNamed(
  Document doc,
  string name )
{
  // built-in parameter storing this
  // wall type's name:

  BuiltInParameter bip
    = BuiltInParameter.SYMBOL\_NAME\_PARAM;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  FilterStringRuleEvaluator evaluator
    = new FilterStringEquals();

  FilterRule rule = new FilterStringRule(
    provider, evaluator, name, false );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( WallType ) )
      .WherePasses( filter );

  return collector.FirstElement() as WallType;
}
```

GetFirstWallUsingType uses the built-in parameter ELEM\_TYPE\_PARAM and a numerical equality filter to retrieve all walls using the required wall type, because their corresponding parameter value will equal the wall type element id.
We don't care which wall is used to launch the "Create Similar" command, so we simply return the first one encountered:
```csharp
static Wall GetFirstWallUsingType(
  Document doc,
  WallType wallType )
{
  // built-in parameter storing this
  // wall's wall type element id:

  BuiltInParameter bip
    = BuiltInParameter.ELEM\_TYPE\_PARAM;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  FilterNumericRuleEvaluator evaluator
    = new FilterNumericEquals();

  FilterRule rule = new FilterElementIdRule(
    provider, evaluator, wallType.Id );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( Wall ) )
      .WherePasses( filter );

  return collector.FirstElement() as Wall;
}
```

With the support class and helper methods in place, the external command mainline Execute method implementation is short and sweet:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  // name of target wall type that we want to use:

  string wallTypeName = "Generic - 203";

  WallType wallType = GetFirstWallTypeNamed(
    doc, wallTypeName );

  Wall wall = GetFirstWallUsingType(
    doc, wallType );

  // select the wall in the UI

  uidoc.Selection.Elements.Add( wall );

  if( 0 == uidoc.Selection.Elements.Size )
  {
    // no wall with the correct wall type found

    FilteredElementCollector collector
      = new FilteredElementCollector( doc );

    Level ll = collector
      .OfClass( typeof( Level ) )
      .FirstElement() as Level;

    // place a new wall with the
    // correct wall type in the project

    Line geomLine = app.Create.NewLineBound(
      XYZ.Zero, new XYZ( 2, 0, 0 ) );

    Transaction t = new Transaction(
      doc, "Create dummy wall" );

    t.Start();

    Wall nw = doc.Create.NewWall( geomLine,
      wallType, ll, 1, 0, false, false );

    t.Commit();

    // Select the new wall in the project

    uidoc.Selection.Elements.Add( nw );

    // Start command create similar. In the
    // property menu, our wall type is set current

    Press.Keys( "CS" );

    // select the new wall in the project,
    // so we can delete it

    uidoc.Selection.Elements.Add( nw );

    // erase the selected wall (remark:
    // doc.delete(nw) may not be used,
    // this command will undo)

    Press.Keys( "DE" );

    // start up wall command

    Press.Keys( "WA" );
  }
  else
  {
    // the correct wall is already selected:

    Press.Keys( "CS" ); // start "create similar"
  }
  return Result.Succeeded;
}
```

As you see, an arbitrary dummy wall of the required type is created if none previously exists.
For this, we start up a transaction of our own, so we are obviously using manual transaction mode.
And so we have to, since we have to close our transaction again before the Revit commands are invoked.

Robert says the following about this code:
Here is a part or our code to start a Revit command.
The aim of the code is to set a wall type current in the Revit property window.
We only start up the wall command with the API and let the user do the drawing of the wall.
This solution can also be used to launch other Revit commands.

When I start up the standard sample project rac\_basic\_sample\_project.rvt, switch to Level 1, and launch this command from the Revit ribbon, I immediately enter the standard Revit wall command.
The desired wall type is active, in this case the type named "Generic - 203", and I can immediately start placing new walls of that type.

There have been many other queries on how to programmatically set up the type before launching a Revit command, and this looks as if it could solve them.

I have to repeat the
[warning about the risks involved with using this](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html#2), though,
and also point back to the
[disclaimer](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html#3) accompanying that warning.

Still, if this is just for your personal use, you might find it pretty handy.

Here is
[version 2011.0.80.0](zip/bc_11_80.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.