---
post_number: "0873"
title: "Installing a Macro and Closing the Active Document"
slug: "close_active_doc"
author: "Jeremy Tammik"
tags: ['revit-api', 'windows']
source_file: "0873_close_active_doc.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0873_close_active_doc.html"
---

### Installing a Macro and Closing the Active Document

You can close an open Revit document programmatically.
Unfortunately, though, this only works for all except the last one.

People keep running into this issue, of course.
It even motivated René Gerlach to implement a workaround using SendKeys.SendWait to send an Alt-F4 keystroke to Revit to
[close the active window](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html),
and Arnošt Löbel to
[warn gravely](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html#2) against its use and even publish a
[Windows message workaround disclaimer](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html#3).

Well, happily, such an adventurous workaround is not at all required.
Another workaround making use only of fully supported API calls is obvious and simple, once you think of it: if you really want to close the current document, and it happens to be the last one, all you have to do is open some other document first.
You can use a dummy document for this purpose.
Once it has been opened, the other 'real' document can be closed.

Steven Mycynek of the Revit API development team provided a sample add-in named UiClose implementing this.
Here is the description in his own words:

> Enclosed is some sample code I use to close the active document through the API.
> I am sure it has some limitations and bugs, but it's a nice start.
>
> To use, simply install the UiClose macro to your macros folder and place the \_placeHolder\_.rvt dummy model into C:/uiclose or some other location where the main macro can find it.
>
> The demo application closes the active document, opening \_placeholder\_.rvt when necessary, and will not accidentally close the placeholder document.
>
> It might also be possible to do something with events in an application to make this more automatic, but this wraps it up for now.

Steve implemented this as a SharpDevelop macro.

The macro files need to be placed in the appropriate
[macro directory](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00001-Revit_He0/3032-Customiz3032/3207-Automati3207/3208-Getting_3208/3209-Upgradin3209):

- Windows XP: C:\Documents and Settings\All Users\Application Data\Autodesk\Revit\Macros\<release>\<product>\VstaMacros
- Windows 7: C:\ProgramData\Autodesk\Revit\Macros\<release>\<product>\VstaMacros

In my case, this resolves to

- C:\Documents and Settings\All Users\Application Data\Autodesk\Revit\Macros\2013\Revit\VstaMacros

Actually, to see the UiClose macro in the SharpDevelop IDE, I had to place the UiClose folder into the AppHookup subdirectory, so its full path ends up as:

- C:\Documents and Settings\All Users\Application Data\Autodesk\Revit\Macros\2013\Revit\VstaMacros\AppHookup\UiClose

Now I see the UiClose module and its CloseActiveDocDemo macro in the Manage > Macros > Macro Manager:

![UiClose CloseActiveDocDemo macro](img/UiClose.png)

When I run the macro, the current active document is closed.
If it is the last one, the placeholder document is opened in its stead.

If it has unsaved changes, a dialogue box prompting me whether to save or not is displayed.
The macro could obviously be improved to check for that before trying to close the document to avoid such an interruption, or handle it more gracefully.

The macro itself is a trivial one-liner, since it just calls the CloseAndSave method provided by the CloseHelper helper class:

```
  public void CloseActiveDocDemo()
  {
    Autodesk.Revit.UI.UIDocument doc
      = this.ActiveUIDocument;

    CloseHelper.CloseAndSave( doc );
  }
```

The rest of the ThisApplication class methods deal with setting up and managing the helper class:

```
public partial class ThisApplication
{
  private void Module_Startup(
    object sender,
    EventArgs e )
  {
    m_CloseHelper = new UiCloseHelper(
      this,
      @"C:\\uiClose\\_placeholder_.rvt" );
  }

  private void Module_Shutdown(
    object sender,
    EventArgs e )
  {
  }

  #region Revit Macros generated code
  private void InternalStartup()
  {
    this.Startup += new System.EventHandler(
      Module_Startup );

    this.Shutdown += new System.EventHandler(
      Module_Shutdown );
  }

  public UiCloseHelper CloseHelper
  {
    get
    {
      return m_CloseHelper;
    }
  }

  private UiCloseHelper m_CloseHelper;
  #endregion
}
```

So finally, let's take a look at the UiCloseHelper implementation in all its glory:

```
/// <summary>
/// A helper class to keep a placeholder document
/// open in Revit to allow closing the active
/// document in Revit.
/// </summary>
public class UiCloseHelper
{
  /// <summary>
  /// Constructor -- initializes object with
  /// a UIApplication and location of a
  /// placeholder document.
  /// </summary>
  /// <param name="uiAppInterface">The Revit UI Application, pass either a UIApplication or an ApplicationEntryPoint</param>
  /// <param name="placeholderPath">Path to a dummy .rvt file.</param>
  public UiCloseHelper(
    dynamic uiAppInterface,
    string placeholderPath )
  {
    m_uiAppInterface = uiAppInterface;

    if( m_uiAppInterface == null )
    {
      throw new ArgumentNullException(
        "m_uiAppInterface" );
    }

    if( ( !( uiAppInterface is Autodesk.Revit.UI.UIApplication ) )
        && ( !( uiAppInterface is Autodesk.Revit.UI.Macros.ApplicationEntryPoint ) ) )
    {
      throw new ArgumentException(
        "Must pass a UIApplication or ApplicationEntryPoint." );
    }

    SetPlaceHolderPath( placeholderPath );
  }

  /// <summary>
  /// Closes a UIDocument with the same
  /// interface as UIDocument.Close
  /// </summary>
  /// <param name="uiDoc">The document to close</param>
  /// <returns>True if successful, false otherwise.</returns>
  public bool CloseAndSave(
    Autodesk.Revit.UI.UIDocument uiDoc )
  {
    if( uiDoc == null )
    {
      throw new ArgumentNullException( "uiDoc" );
    }

    RejectClosingPlaceHolder( uiDoc );

    if( !EnsureActivePlaceHolder( m_uiAppInterface ) )
      return false;

    return uiDoc.SaveAndClose();
  }

  /// <summary>
  /// Closes a DB.Document with the same
  /// interface as DB.Document.Close
  /// </summary>
  /// <param name="doc">The DB.Document to close</param>
  /// <returns>True if successful, false otherwise.</returns>
  public bool Close( Autodesk.Revit.DB.Document doc )
  {
    return Close( doc, true );
  }

  /// <summary>
  /// Closes a DB.Document with the same
  /// interface as DB.Document.Close
  /// </summary>
  /// <param name="doc">The DB.Document to close</param>
  /// <param name="saveModified">true to save the document if modified, false to not save.</param>
  /// <returns>True if successful, false otherwise.</returns>
  public bool Close(
    Autodesk.Revit.DB.Document doc,
    bool saveModified )
  {
    if( doc == null )
    {
      throw new ArgumentNullException( "doc" );
    }

    RejectClosingPlaceHolder( doc );

    if( !EnsureActivePlaceHolder( m_uiAppInterface ) )
      return false;

    return doc.Close( saveModified );
  }

  #region Helper methods

  /// <summary>
  /// Throws an exception if the user
  /// passes the placeholder document.
  /// </summary>
  /// <param name="doc">The document to test to ensure it is not the placeholder document.</param>
  private void RejectClosingPlaceHolder(
    Autodesk.Revit.DB.Document doc )
  {
    if( doc.Title == System.IO.Path.GetFileName(
      this.m_placeHolderPath ) )
    {
      throw new ArgumentException(
        "Cannot close placeholder doc: " + doc.Title );
    }
  }

  /// <summary>
  /// Throws an exception if the user
  /// passes the placeholder document.
  /// </summary>
  /// <param name="doc">The document to check</param>
  private void RejectClosingPlaceHolder(
    Autodesk.Revit.UI.UIDocument uiDoc )
  {
    RejectClosingPlaceHolder( uiDoc.Document );
  }

  /// <summary>
  /// Ensures that the placeholder document is open
  /// and active to that another open document can be closed.
  /// </summary>
  private bool EnsureActivePlaceHolder(
    dynamic application )
  {
    try
    {
      if( m_placeHolderDoc == null )
      {
        m_placeHolderDoc = application
          .OpenAndActivateDocument( m_placeHolderPath );
      }
      else
      {
        m_placeHolderDoc.SaveAndClose();
        m_placeHolderDoc = application
          .OpenAndActivateDocument( m_placeHolderPath );
      }
      return true;
    }
    catch( Exception )
    {
      return false;
    }
  }

  /// <summary>
  /// Sets the path of the placeholder document.
  /// </summary>
  /// <param name="path">Path to placeholder/dummy document.</param>
  private void SetPlaceHolderPath( string path )
  {
    m_placeHolderPath = path;

    if( !System.IO.File.Exists( m_placeHolderPath ) )
    {
      throw new ArgumentException(
        "File not found: " + path );
    }
    if( !( System.IO.Path.GetExtension( m_placeHolderPath ) != "rvt" ) )
    {
      throw new ArgumentException( "Placeholder: "
        + path + " is not a revit file." );
    }
  }
  #endregion

  #region Data
  private Autodesk.Revit.UI.UIDocument m_placeHolderDoc;
  private string m_placeHolderPath;
  private dynamic m_uiAppInterface;
  #endregion
}
```

It reads a lot harder than it actully is :-)

Here is
[UiClose.zip](zip/UiClose.zip) containing the macro source code and its associated
[placeholder](zip/_placeholder_.rvt) project document.

Many thanks to Steve for providing this useful and interesting workaround!