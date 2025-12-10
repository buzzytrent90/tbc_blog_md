---
post_number: "0807"
title: "Cancel Family Creation and UK NBS and BIM News"
slug: "cancel_family_create"
author: "Jeremy Tammik"
tags: ['family', 'python', 'revit-api']
source_file: "0807_cancel_family_create.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0807_cancel_family_create.html"
---

### Cancel Family Creation and UK NBS and BIM News

Let me begin directly with some Revit API sample code today.

Here is a little external application showing how you can optionally prohibit family creation by subscribing to the cancellable DocumentCreating event:
```python
class App : IExternalApplication
{
  void OnDocumentCreating(
    object sender,
    DocumentCreatingEventArgs e )
  {
    DocumentType typ = e.DocumentType;

    if( DocumentType.Family == typ )
    {
      TaskDialog d = new TaskDialog(
        "Open Family Editor?" );

      d.MainInstruction = "Creating a new family "
        + "document... would you like to proceed?";

      d.MainContent = string.Format(
        "Document type: {0}\r\nCancellable: {1}",
        typ, (e.Cancellable ? "Yes" : "No") );

      d.CommonButtons = TaskDialogCommonButtons.Yes
        | TaskDialogCommonButtons.No;

      d.DefaultButton = TaskDialogResult.No;

      if( TaskDialogResult.No == d.Show() )
      {
        e.Cancel();
      }
    }
  }

  public Result OnStartup(
    UIControlledApplication a )
  {
    a.ControlledApplication.DocumentCreating
      += new EventHandler<DocumentCreatingEventArgs>(
        OnDocumentCreating );

    return Result.Succeeded;
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    return Result.Succeeded;
  }
}
```

Surprisingly short and simple, isn't it, and the code speaks for itself, does it not?

Here is
[CancelFamilyCreation.zip](zip/CancelFamilyCreation.zip) containing
the full source code, Visual Studio solution and add-in manifest of this little add-in.

#### The UK National BIM Library and NBS

For some little less hard-core UK-related BIM news, Stephen Hamil of
[RIBA Enterprises](http://www.ribaenterprises.com) pointed
out a couple of interesting titbits related to NBS and the National BIM Library, a free resource for the UK construction industry:

- The free [NBS for Revit](http://constructioncode.blogspot.co.uk/2012/08/nbs-for-autodesk-revit-plug-in-now-live.html) tool has been released.- Report from the first [BIM user day](http://www.thenbs.com/topics/BIM/articles/feedbackOnTheNBL.asp) with a number of big Autodesk users such as HOK, BDP, LOR, Balfour Beatty, Ryder, Ramboll...

Thank you, Stephen, for the note.

#### dbStuff Add-ins

Finally, I took a quick look at the free dbStuff Revit add-ins
[ChangeTextCase](http://feedproxy.google.com/~r/DpStuff/~3/cw4HYzzxgMs/changetextcase-alter-case-of-your-revit.html) and
[NumberStuffByDirection](http://dimak1999.blogspot.de/2012/08/numberstuffbydirection-revit-api-plugin.html),
which look both useful and interesting.
Thank you, Dima Chiriacov, for sharing these!