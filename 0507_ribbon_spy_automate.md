---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.7
content_type: documentation
optimization_date: '2025-12-11T11:44:14.070512'
original_url: https://thebuildingcoder.typepad.com/blog/0507_ribbon_spy_automate.html
post_number: '0507'
reading_time_minutes: 10
series: general
slug: ribbon_spy_automate
source_file: 0507_ribbon_spy_automate.htm
tags:
- csharp
- elements
- levels
- references
- revit-api
- views
- walls
- windows
title: Ribbon Spying and UI Automation
word_count: 1982
---

### Ribbon Spying and UI Automation

I hope you had a wonderful end of the last and beginning of the new year!

Happy New Year to you all!

I returned from the week of rest and was completely occupied with the backlog of ADN support cases in the past few days.

Here is a really heavy-duty post to get us started again, and started with a bang!

It revisits the never-ending and always popular issue of
[launching a Revit command programmatically](http://thebuildingcoder.typepad.com/blog/2010/11/launching-a-revit-command.html),
but this time based on some much more in-depth analysis of the Revit ribbon internals performed by Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de).
Here is what he discovered and has to say about it:

#### Driving Revit from Outside

An interesting (and recurring) point is to 'drive Revit from outside'.

I know, you have discussed this many times, but I think that this will be a different approach:

Since Revit uses Ribbons, there is no way to click buttons or menu items just by sending Windows messages, as we could do it in Revit 2009.

If you take a close look using Spy++, you'll see that the whole RibbonBar is just a container, a Black Box:

![Ribbon bar in Spy++](img/rh_spy.png)

With Spy++, you cannot examine this container because it's a WPF element.

But using UISpy, you can see this:

![Ribbon bar in UISpy](img/rh_uispy.png)

In this case, the selected element is a single button. So, how we can invoke ***any*** button?

Assuming there is a new Addin panel called 'TestPanel' with a single button called 'TestButton', here is a way to press it from any application:
```csharp
  AutomationElement mainWndFromHandle
    = AutomationElement.FromHandle(
      \_hWndRevit.Handle ); // the revit window handle

  PropertyCondition nameRibbonCondition
    = new PropertyCondition(
      AutomationElement.NameProperty,
      "RibbonHostWindow" );

  PropertyCondition typeRibbonCondition
    = new PropertyCondition(
      AutomationElement.ControlTypeProperty,
      ControlType.Pane );

  AndCondition andCondition
    = new AndCondition(
      typeRibbonCondition,
      nameRibbonCondition );

  ribbonWnd = mainWndFromHandle.FindFirst(
    TreeScope.Children, andCondition );

  PropertyCondition aIDCondition
    = new PropertyCondition(
      AutomationElement.AutomationIdProperty,
      "ADD\_INS\_TAB" );

  AutomationElement addinbutton
    = ribbonWnd.FindFirst(
    TreeScope.Children, aIDCondition );

  // show addin panel by pressing the tab header

  InvokePattern invPattern
    = addinbutton.GetCurrentPattern(
      InvokePattern.Pattern ) as InvokePattern;

  invPattern.Invoke();

  // pause, so ribbon panels can re-arrange

  System.Threading.Thread.Sleep( 1000 );

  PropertyCondition aIDPanelCondition
    = new PropertyCondition(
      AutomationElement.AutomationIdProperty,
      "ADD\_INS\_TAB\_PanelBarScrollViewer" );

  AutomationElement addinPanel
    = ribbonWnd.FindFirst( TreeScope.Children,
      aIDPanelCondition );

  PropertyCondition aIDTestPanelCondition
    = new PropertyCondition(
      AutomationElement.AutomationIdProperty,
      "CustomCtrl\_%ADD\_INS\_TAB%TestPanel" );

  AutomationElement testPanel
    = addinPanel.FindFirst( TreeScope.Children,
      aIDTestPanelCondition );

  PropertyCondition aIDContainerCondition
    = new PropertyCondition(
      AutomationElement.AutomationIdProperty,
      "CustomCtrl\_%CustomCtrl\_%ADD\_INS\_TAB%TestPanel%TestButton\_RibbonItemControl" );

  AutomationElement testContainer
    = testPanel.FindFirst( TreeScope.Children,
      aIDContainerCondition );

  PropertyCondition aIDTestButtonCondition
    = new PropertyCondition(
      AutomationElement.AutomationIdProperty,
      "CustomCtrl\_%CustomCtrl\_%ADD\_INS\_TAB%TestPanel%TestButton" );

  AutomationElement testButton
    = testContainer.FindFirst( TreeScope.Children,
      aIDTestButtonCondition );

  InvokePattern invPatternButton
    = testButton.GetCurrentPattern(
      InvokePattern.Pattern ) as InvokePattern;

  // now press our button via uiautomation

  invPatternButton.Invoke();
```

Remarks:

- Invisible RibbonItems cannot be pressed, so make sure the panel that contains them is visible.- Thus, if RibbonBar is collapsed, expand it before pressing buttons.- Give it some time; RibbonBar needs to re-arrange, this takes some milliseconds every time.- The hierarchy of the RibbonBar and its items (descendants) is different in 2010 and 2011; sometimes there is an AutomationID, but this ID may be not unique, so you have to combine ControlTypeProperty, NameProperty and/or AutomationIDProperty PropertyConditions to get an element.- Don't use TreeScope.Descendants; it's faster to search just in the TreeScope.Children list.- Store mainWindowHandle, ribbonWnd and other AutomationElements in global variables because it saves some time, but be aware because some items may become invalid while RibbonBar is switching.

This way, you can invoke any command from outside. In opposite to the OnIdling event, which cannot be fired if a modal dialogue is opened in Revit, this technique allows you to close even this (blocking) dialog from outside.

Also Revit standard commands can be invoked (e.g., creating a new Wall via GUI).

It even allows you to open a Revit file via GUI, avoiding the use of the Process.Start method.

Remark: in this example, I use some German control texts; in a localized application, this would be replaced by resource strings, or the AutomationElements would be got in a different manner.
```csharp
private void OpenRevitFile( string filePath )
{
  // get the Revit 'R' button in the upper left corner
  // FindWindowEx has been imported via P/Invoke
  IntPtr startButtonHandle = FindWindowEx(
    IntPtr.Zero, IntPtr.Zero, "AdApplicationButton",
    "AdApplicationButton" );

  if( startButtonHandle != IntPtr.Zero )
  {
    // SendMessage has been imported via P/Invoke

    SendMessage( startButtonHandle, WM\_LBUTTONDOWN,
      IntPtr.Zero, IntPtr.Zero ); // click

    SendMessage( startButtonHandle, WM\_LBUTTONUP,
      IntPtr.Zero, IntPtr.Zero ); // release

    // these lines should be known

    Process[] processes = Process.GetProcessesByName(
      "Revit" );

    WindowHandle \_hWndRevit = null;

    if( 0 < processes.Length )
    {
      IntPtr h = processes[0].MainWindowHandle;

      \_hWndRevit = new WindowHandle( h );
    }
    if( \_hWndRevit != null )
    {
      // revit window
      AutomationElement mainWndFromHandle
        = AutomationElement.FromHandle(
          \_hWndRevit.Handle );

      // start menu

      PropertyCondition idMenuCondition
        = new PropertyCondition(
          AutomationElement.AutomationIdProperty,
          "Id\_ApplicationMenuWindow" );

      AutomationElement menuWnd
        = mainWndFromHandle.FindFirst(
          TreeScope.Children, idMenuCondition );

      // start submenu

      PropertyCondition idSubMenuCondition
        = new PropertyCondition(
          AutomationElement.AutomationIdProperty,
          "mFirstLevelMenuList" );

      // list

      AutomationElement subMenuWnd
        = menuWnd.FindFirst( TreeScope.Children,
          idSubMenuCondition );

      // list item

      PropertyCondition typeItemCondition
        = new PropertyCondition(
          AutomationElement.ControlTypeProperty,
          ControlType.ListItem );

      PropertyCondition nameItemCondition
        = new PropertyCondition(
          AutomationElement.NameProperty,
          "Autodesk.Windows.ApplicationMenuItem" );

      PropertyCondition idItemCondition
        = new PropertyCondition(
          AutomationElement.AutomationIdProperty,
          "ID\_REVIT\_FILE\_OPEN" );

      AndCondition andItemCondition
        = new AndCondition( idItemCondition,
          typeItemCondition );

      AutomationElement openItemWnd
        = subMenuWnd.FindFirst( TreeScope.Children,
          andItemCondition );

      PropertyCondition typeButtonCondition
        = new PropertyCondition(
          AutomationElement.ControlTypeProperty,
          ControlType.Button );

      AutomationElementCollection openButtons
        = openItemWnd.FindAll( TreeScope.Children,
          typeButtonCondition );

      foreach( AutomationElement openButton
        in openButtons )
      {
        PropertyCondition typeImageCondition
          = new PropertyCondition(
            AutomationElement.ControlTypeProperty,
            ControlType.Image );

        AutomationElementCollection images
          = openButton.FindAll( TreeScope.Children,
            typeImageCondition );

        // search a button with an image

        if( images.Count > 0 )
        {
          InvokePattern invPattern
            = openButton.GetCurrentPattern(
              InvokePattern.Pattern ) as InvokePattern;

          invPattern.Invoke();
        }
      }

      // open dialog window

      // pause while dialog is being opened

      Thread.Sleep( 700 );

      // re-read revit window components to find new dialog

      PropertyCondition nameOpenDlgCondition
        = new PropertyCondition(
          AutomationElement.NameProperty,
          "Öffnen" ); // us-EN "Open"

      AutomationElementCollection allOpenDlgs
        = mainWndFromHandle.FindAll(
          TreeScope.Children, nameOpenDlgCondition );

      foreach( AutomationElement openOpenDlgWnd
        in allOpenDlgs )
      {
        if( openOpenDlgWnd.Current.LocalizedControlType
          == "Dialogfeld" )
        {
          // comboBox has also an&

          PropertyCondition typeComboBoxCondition
            = new PropertyCondition(
              AutomationElement.ControlTypeProperty,
              ControlType.ComboBox );

          PropertyCondition idComboBoxCondition
            = new PropertyCondition(
              AutomationElement.AutomationIdProperty,
              "13006" );

          AndCondition andComboBoxCondition
            = new AndCondition( idComboBoxCondition,
              typeComboBoxCondition );

          AutomationElement comboBoxWnd
            = openOpenDlgWnd.FindFirst( TreeScope.Children,
              andComboBoxCondition );

          // &edit field

          PropertyCondition typeEditCondition
            = new PropertyCondition(
              AutomationElement.ControlTypeProperty,
              ControlType.Edit );

          PropertyCondition idEditCondition
            = new PropertyCondition(
              AutomationElement.AutomationIdProperty,
              "1001" );

          AndCondition andEditCondition
            = new AndCondition( idEditCondition,
              typeEditCondition );

          AutomationElement editWnd
            = comboBoxWnd.FindFirst( TreeScope.Children,
              andEditCondition );

          Thread.Sleep( 900 );

          ValuePattern valPattern
            = editWnd.GetCurrentPattern(
              ValuePattern.Pattern ) as ValuePattern;

          // paste file path into the edit field

          valPattern.SetValue( filePath );

          // press ok button

          PropertyCondition idOpenButtonCondition
            = new PropertyCondition(
              AutomationElement.AutomationIdProperty,
              "1" );

          AndCondition andOpenButtonCondition
            = new AndCondition( idOpenButtonCondition,
              typeButtonCondition );

          AutomationElement openButton
            = openOpenDlgWnd.FindFirst( TreeScope.Children,
              andOpenButtonCondition );

          InvokePattern invPattern
            = openButton.GetCurrentPattern(
              InvokePattern.Pattern ) as InvokePattern;

          if( invPattern != null )
          {
            invPattern.Invoke();
          }
        }
      }
    }
  }
}
```

P.S.: these code fragments may contain some typos; it could also be that one does it for 2010 and another for 2011 because this is old stuff that I played with months ago.

But the code shows the way, so it's useful as well.

Before using this code, you need to import some UIAutomation dlls:

![UI automation references](img/rh_ui_automation_references.png)

There may be some better sources for UIAutomation in the Internet...

#### Stand-Alone Sample Application

Rudolf created a stand-alone Windows Forms project
DrivingRevitViaUIAutomation that
demonstrates the use of this approach.
Since UIAutomation can drive any application, the mechanism is not restricted to Revit a add-in, but can be used there as well as in any other context.
For instance, you also could drive 3dsmax or something else from outside.

By the way, while developing my Revit MDI window arranging tool, I discovered that AutoCAD has a window structure similar to Revit.
This is of course not surprising because it's Autodesk's policy to make the products look similar.

DrivingRevitViaUIAutomation is adjusted to drive a German version of Revit 2011.

There are about three German literals in the code, describing some dialog titles.

It includes added comments like this: "ffnen" // en-US:"Open"

These are the parts the user must replace by his own dialog titles if his Revits language is not German.

It may be necessary to adjust some pause intervals in the code.

This example demonstrates three steps:

- Open a file.- Close a file (in fact, the active document); you can choose whether you want to save it before closing.- Switch the current RibbonTab. Before selecting a Tab by pressing a TreeNode, you must read the Ribbon once, so the switching process will be faster because the Tabs are in a buffer.

The user interface initially looks like this, with a sample file that can be opened and buttons for closing it with and without saving and for populating the list of ribbon tabs:

![UI automation sample application](img/rh_ui_automation_sample.png)

Once the the list of ribbon tabs has been populated, each one of them can be clicked to activate it in the Revit user interface:

![UI automation sample application with populated ribbon tabs](img/rh_ui_automation_sample_populated.png)

The code uses a mix of the fast Window functions (P/Invoke) with the slow UIAutomation functions; this improves performance and may make a difference in more complex examples.

Regarding the language dependency: For example, the "open file" dialog is found by its title text, and in my example, this is "ffnen".

Open this dialog manually, read the (English) title and replace the German expression in the code.

Additionally, this part is important:
```csharp
  PropertyCondition nameOpenDlgCondition
    = new PropertyCondition(
      AutomationElement.NameProperty,
      "Öffnen" ); // en-US: "Open"

  AndCondition andOpenDlgCondition
    = new AndCondition( nameOpenDlgCondition,
      typeOpenDlgCondition );

  Thread.Sleep( 600 );

  AutomationElementCollection allOpenDlgs
    = mainWndFromHandle.FindAll(
      TreeScope.Children,
      nameOpenDlgCondition );

  //typeOpenDlgCondition andOpenDlgCondition

  foreach( AutomationElement openOpenDlgWnd
    in allOpenDlgs )
  {
    if( openOpenDlgWnd.Current.LocalizedControlType
      == "Dialogfeld" ) // en-US: ?
    {
```

If you are running an English version of Revit, you need to examine the LocalizedControlType for it; in fact, not for your Revit version but for your OS Language version.

Look at this screenshot taken from an English Revit residing in a German VM:
![UISpy looking at an English version](img/rh_uispy_en.png)

The LocalizedControlType depends on OS Language; nonetheless, the dialog titles change according to Revit language.

You need to examine the LocalizedControlType by your own, I think, but its just one use of UISpy.

I think there might be a way to do it without any localization issues, but as I explained before, finding the correct AutomationElement is always a compromise between performance and elegance.

Sometimes there is no property that makes a AutomationElement individual, no AutomationID, just Controltype.Pane, for example.

If you just search by ControlType.Button, for example, it might be that you find some unwanted results, or, as I explained, the performance will decrease.

**Combining the search conditions in the right way is the thing that matters.**

In Revit 2010, there were less AutomationIDs than in 2011, and some items shared their ID; e.g. there were some buttons with icons and others without icons, both for opening or closing files.

Getting the button with icon meant to see whether the button had children because all other Properties were identical – besides position, of course, but this may change...

Its "workarounding" at its best.

Another last comment or two from Rudolf on this topic:

I am hoping that Revit's ribbon response time will decrease in future versions; if so, then the calls to System.Threading.Sleep( aLotOfTime ) can be changed to System.Threading.Sleep( justAMillisecond ).
So I hope.

And, by the way, UISpy allows you to export the Window structure to an XML document, so you can browse this snapshot instead of the 'real' windows.

Very many thanks to Rudolf for this in-depth exploration which opens up many new possibilities!

Exploring this should keep you occupied for a while...

Once again I have to repeat the
[warning about the risks involved](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html#2) with
using this, though, and also point back to the
[disclaimer](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html#3) accompanying
that warning.