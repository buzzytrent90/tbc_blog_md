---
post_number: "1292"
title: "List and Switch Design Options Using UI Automation"
slug: "design_option_modifier"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api', 'selection', 'views', 'windows']
source_file: "1292_design_option_modifier.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1292_design_option_modifier.html"
---

### List and Switch Design Options Using UI Automation

Yesterday, we discussed a Revit add-in using .NET UI Automation to retrieve the current state of the Revit thin lines setting.

Today, let's look at another application demonstrating use of that functionality to determine, list and set the Revit design options from a stand-alone executable application.

Once again, this sample is presented by
[Revitalizer](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/1103138), aka
Rudolf Honke of [Mensch und Maschine acadGraph](http://www.acadgraph.de),
who already contributed numerous other examples making use of the
[.NET UI Automation library](http://thebuildingcoder.typepad.com/blog/automation) to
hack the Revit user interface.

Please note once again
[The Building Coder Disclaimer](http://thebuildingcoder.typepad.com/blog/about-the-author.html#4):
in the following, we present a going beyond the officially supported Revit API, leading to an experimental implementation suitable only for a personal controlled usage that should not be relied upon for production use.

#### Revitalizer Implementation Notes

**Rudolf says:**
Here is a tool for switching between DesignOptions.

This is a stand-alone application that gets and sets DesignOptions.

Since it works very, very slowly, it is not too useful for everyday professional work.

![DesignOptionModifier main form](img/DesignOptionModifier_main_form.png)

It just reads and sets the ComboBox, but reading the items requires expanding them all first, which costs several seconds each time.

![Revit Design Option combo box](img/DesignOptionModifier_revit.png)

To test it, just start a Revit project containing a few DesignOptions.

Then start the DesignOptionModifier program.

You can drive Revit from outside this way.

Please also note that DesignOption groups are displayed in the DesignOptionModifier program listbox as well.

As far as I know, there is no way to check if entries are greyed out in the Revit combobox.

#### Sample Run

This is a stand-alone executable application presenting one single Windows form.

It implements two buttons:

- Retrieve the current design options defined in Revit.
- Set the current Revit design option to the selected list entry.

Here is an example of running it after starting up Revit, defining a couple of new design options and clicking the 'Get' button:

![DesignOptionModifier retrieving design options](img/DesignOptionModifier_form_2.png)

After selecting the second entry and clicking 'Set', the selected entry is activated in Revit:

![DesignOptionModifier modified the  design option](img/DesignOptionModifier_revit_2.png)

#### Implementation Notes

The program mainline is trivial, because all action is implemented by the form implementation:

```csharp
using System;
using System.Windows.Forms;

namespace DesignOptionModifier
{
  static class Program
  {
    /// <summary>
    /// Der Haupteinstiegspunkt für die Anwendung.
    /// </summary>
    [STAThread]
    static void Main()
    {
      Application.EnableVisualStyles();
      Application.SetCompatibleTextRenderingDefault(false);
      Application.Run(new Form1());
    }
  }
}
```

The form implementation demonstrates how to:

- Use P/Invoke to access and make use of the Windows API functionality defined in User32.dll to find and enumerate specific windows.
- Find the design option combo box in the Revit main window status bar.
- Read the current design option from the Revit combo box.
- Update the value of the Revit combo box to modify the selected design option.
- Handle the form button clicks.
- Control the foreground window, etc.

Here is the form source code:

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Diagnostics;
using System.Drawing;
using System.Runtime.InteropServices;
using System.Text;
using System.Windows.Automation;
using System.Windows.Forms;
using System.Windows.Input;

namespace DesignOptionModifier
{
  public partial class Form1 : Form
  {
    #region Windows API

    [DllImport( "user32.dll" )]
    [return: MarshalAs( UnmanagedType.Bool )]
    public static extern bool SetForegroundWindow(
      IntPtr hWnd );

    [DllImport( "user32.dll", SetLastError = true, CharSet = CharSet.Auto )]
    static extern int GetWindowText(
      IntPtr hWnd, [Out] StringBuilder lpString,
      int nMaxCount );

    [DllImport( "user32.dll", SetLastError = true, CharSet = CharSet.Auto )]
    static extern int GetWindowTextLength(
      IntPtr hWnd );

    [DllImport( "user32.dll" )]
    [return: MarshalAs( UnmanagedType.Bool )]
    public static extern bool EnumChildWindows(
      IntPtr window, EnumWindowProc callback, IntPtr i );

    [DllImport( "user32.dll", EntryPoint = "GetClassName" )]
    public static extern int GetClass(
      IntPtr hWnd, StringBuilder className, int nMaxCount );

    public delegate bool EnumWindowProc(
      IntPtr hWnd, IntPtr parameter );

    [DllImport( "user32.dll", SetLastError = true )]
    [return: MarshalAs( UnmanagedType.Bool )]
    private static extern bool GetWindowRect(
      IntPtr hWnd, out RECT lpRect );

    public static string GetText( IntPtr hWnd )
    {
      int length = GetWindowTextLength( hWnd );
      StringBuilder sb = new StringBuilder( length + 1 );
      GetWindowText( hWnd, sb, sb.Capacity );
      return sb.ToString();
    }

    private static bool EnumWindow(
      IntPtr handle,
      IntPtr pointer )
    {
      GCHandle gch = GCHandle.FromIntPtr( pointer );
      List<IntPtr> list = gch.Target as List<IntPtr>;
      if( list != null )
      {
        list.Add( handle );
      }
      return true;
    }

    public static List<IntPtr> GetChildWindows(
      IntPtr parent )
    {
      List<IntPtr> result = new List<IntPtr>();
      GCHandle listHandle = GCHandle.Alloc( result );
      try
      {
        EnumWindowProc childProc = new EnumWindowProc( EnumWindow );
        EnumChildWindows( parent, childProc, GCHandle.ToIntPtr( listHandle ) );
      }
      finally
      {
        if( listHandle.IsAllocated )
          listHandle.Free();
      }
      return result;
    }

    internal struct RECT
    {
      public int Left;
      public int Top;
      public int Right;
      public int Bottom;
    }
    #endregion

    /// <summary>
    /// Cache combobox
    /// </summary>
    private AutomationElement comboBoxElement = null;

    public Form1()
    {
      InitializeComponent();

      GetComboBox();
    }

    private void button\_ok\_Click(
      object sender,
      EventArgs e )
    {
      Close();
    }

    private void GetComboBox()
    {
      int maxX = -1;

      Process[] revits = Process.GetProcessesByName(
        "Revit" );

      IntPtr cb = IntPtr.Zero;

      if( revits.Length > 0 )
      {
        List<IntPtr> children = GetChildWindows(
          revits[0].MainWindowHandle );

        foreach( IntPtr child in children )
        {
          StringBuilder classNameBuffer
            = new StringBuilder( 100 );

          int className = GetClass( child,
            classNameBuffer, 100 );

          if( classNameBuffer.ToString().Contains(
            "msctls\_statusbar32" ) )
          {
            List<IntPtr> grandChildren
              = GetChildWindows( child );

            foreach( IntPtr grandChild in grandChildren )
            {
              StringBuilder classNameBuffer2
                = new StringBuilder( 100 );

              int className2 = GetClass( grandChild,
                classNameBuffer2, 100 );

              if( classNameBuffer2.ToString().Contains(
                "ComboBox" ) )
              {
                RECT r;

                GetWindowRect( grandChild, out r );

                // There are at least two comboboxes,
                // and we want the rightmost one.

                if( r.Left > maxX )
                {
                  maxX = r.Left;
                  cb = grandChild;
                }
              }
            }
          }
        }
      }

      if( cb != IntPtr.Zero )
      {
        comboBoxElement = AutomationElement.FromHandle(
          cb );
      }
    }

    private void buttonGet\_Click(
      object sender,
      EventArgs e )
    {
      if( ( comboBoxElement != null )
        && ( comboBoxElement.Current.IsEnabled ) )
      {
        ExpandCollapsePattern expandPattern
          = (ExpandCollapsePattern) comboBoxElement
            .GetCurrentPattern(
              ExpandCollapsePattern.Pattern );

        expandPattern.Expand();

        listBox1.Items.Clear();

        CacheRequest cacheRequest = new CacheRequest();
        cacheRequest.Add( AutomationElement.NameProperty );

        cacheRequest.TreeScope = TreeScope.Element
          | TreeScope.Children;

        AutomationElement comboboxItems = comboBoxElement
          .GetUpdatedCache( cacheRequest );

        foreach( AutomationElement item
          in comboboxItems.CachedChildren )
        {
          if( item.Current.Name == "" )
          {
            CacheRequest cacheRequest2 = new CacheRequest();
            cacheRequest2.Add( AutomationElement.NameProperty );
            cacheRequest2.TreeScope = TreeScope.Element
              | TreeScope.Children;

            AutomationElement comboboxItems2
              = item.GetUpdatedCache( cacheRequest );

            foreach( AutomationElement item2
              in comboboxItems2.CachedChildren )
            {
              listBox1.Items.Add( item2.Current.Name );
            }
          }
        }
        expandPattern.Collapse();
      }
      GetSelection();
    }

    private void GetSelection()
    {
      if( ( comboBoxElement != null )
        && ( comboBoxElement.Current.IsEnabled ) )
      {
        SelectionPattern selPattern = comboBoxElement
          .GetCurrentPattern( SelectionPattern.Pattern )
            as SelectionPattern;

        AutomationElement[] items = selPattern.Current
          .GetSelection();

        foreach( AutomationElement item in items )
        {
          int index = 0;

          foreach( var listItem in listBox1.Items )
          {
            if( (string) listItem == item.Current.Name )
            {
              listBox1.SelectedIndex = index;
              break;
            }
            index++;
          }
        }
      }
      SetForegroundWindow( this.Handle );
    }

    private void SetSelection( string option )
    {
      if( ( comboBoxElement != null )
        && ( comboBoxElement.Current.IsEnabled ) )
      {
        AutomationElement sel = null;

        ExpandCollapsePattern expandPattern
          = (ExpandCollapsePattern) comboBoxElement
            .GetCurrentPattern(
              ExpandCollapsePattern.Pattern );

        expandPattern.Expand();

        CacheRequest cacheRequest = new CacheRequest();
        cacheRequest.Add( AutomationElement.NameProperty );
        cacheRequest.TreeScope = TreeScope.Element
          | TreeScope.Children;

        AutomationElement comboboxItems
          = comboBoxElement.GetUpdatedCache(
            cacheRequest );

        foreach( AutomationElement item
          in comboboxItems.CachedChildren )
        {
          if( item.Current.Name == "" )
          {
            CacheRequest cacheRequest2 = new CacheRequest();
            cacheRequest2.Add( AutomationElement.NameProperty );
            cacheRequest2.TreeScope = TreeScope.Element
              | TreeScope.Children;

            AutomationElement comboboxItems2
              = item.GetUpdatedCache( cacheRequest );

            foreach( AutomationElement item2
              in comboboxItems2.CachedChildren )
            {
              if( item2.Current.Name == option )
              {
                sel = item2;
              }
            }
          }
        }

        if( sel != null )
        {
          SelectionItemPattern select =
            (SelectionItemPattern) sel.GetCurrentPattern(
              SelectionItemPattern.Pattern );

          select.Select();
        }
        expandPattern.Collapse();
      }
      SetForegroundWindow( this.Handle );
    }

    private void buttonSet\_Click(
      object sender,
      EventArgs e )
    {
      if( listBox1.Items.Count > 0
        && listBox1.SelectedIndex > -1 )
      {
        string option = (string)
          listBox1.Items[listBox1.SelectedIndex];

        SetSelection( option );
      }
      SetForegroundWindow( this.Handle );
    }
  }
}
```

#### Download

The complete source code and Visual Studio solution are provided in the
[DesignOptionModifier GitHub repository](https://github.com/jeremytammik/DesignOptionModifier),
and the version described here is
[release 2015.0.0.0](https://github.com/jeremytammik/DesignOptionModifier/releases/tag/2015.0.0.0).

As said, please be aware of
[The Building Coder Disclaimer](http://thebuildingcoder.typepad.com/blog/about-the-author.html#4) before
you even dream of making use of this.

Many thanks to Rudi for his research and nice, clean implementation!

#### Dr. Nutt on the Harm Caused by Alcohol

Before closing, let me add a note on something completely different.

I enjoy reading
[Das Magazin](http://blog.dasmagazin.ch),
a Saturday supplement to several major Swiss daily newspapers.

The current issue,
[nr. 10, March 6, 2015](http://blog.dasmagazin.ch/aktuelles_heft/10)
([PDF](http://blog.dasmagazin.ch/wp-content/uploads/2015/03/ma1510.pdf)),
includes an interview with
[Dr. David Nutt](https://en.wikipedia.org/wiki/David_Nutt) on
the harm caused by alcohol and other drugs and the legislation and politics we put in place to deal with it.

The succinct summary is surprising, funny and scary:

- Alcohol causes more harm worldwide than any other known drug.
- The sport of horse riding is more dangerous than taking ecstasy.
- Some politicians deny rational analysis and science and refuse to work towards achieving the best for society and us citizens, putting higher priorities on other agenda.

As said, pretty scary and shocking stuff, especially the way we as a society and our politicians deal with the issue.