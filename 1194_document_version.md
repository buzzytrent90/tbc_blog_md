---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: code_example
optimization_date: '2025-12-11T11:44:15.487965'
original_url: https://thebuildingcoder.typepad.com/blog/1194_document_version.html
post_number: '1194'
reading_time_minutes: 3
series: general
slug: document_version
source_file: 1194_document_version.htm
tags:
- elements
- family
- parameters
- python
- revit-api
- transactions
- views
- windows
title: Document Version, GUID and Number of Saves
word_count: 610
---

### Document Version, GUID and Number of Saves

Alexander Buschmann of the
[IDAT](http://www.idat.de) Ingenieurbüro für Datenverarbeitung in der Technik GmbH
added a new
[comment](http://thebuildingcoder.typepad.com/blog/2014/07/ifc-guid-algorithm-update-and-family-modification.html?cid=6a00e553e16897883301a73e051b6e970d#comment-6a00e553e16897883301a73e051b6e970d) to
the interesting discussion on
[detecting family modification](http://thebuildingcoder.typepad.com/blog/2014/07/ifc-guid-algorithm-update-and-family-modification.html#3):

> I'm a little bit late, but still:
>
> Since Revit2015 there is a class 'DocumentVersion' – it provides a GUID and the number of saves of the document.
>
> Together, this information should make it possible to identify changes to a document.
>
> Store the data in a read-only parameter, and you should be set, without the need to implement a custom checksum thing.
>
> The DocumentVersion can be obtained by calling BasicFileInfo.GetDocumentVersion.
>
> I have not tried it, but it looks promising.

Please refer to the
[original discussion thread](http://thebuildingcoder.typepad.com/blog/2014/07/ifc-guid-algorithm-update-and-family-modification.html#comments) to
see the other interesting suggestions provided to detect family modification.

#### CmdDocumentVersion

Based on Alexander's suggestion, I implemented this new short and sweet read-only external command CmdDocumentVersion in The Building Coder samples:

```python
  [Transaction( TransactionMode.ReadOnly )]
  class CmdDocumentVersion : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData revit,
      ref string message,
      ElementSet elements )
    {
      UIApplication uiapp = revit.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Document doc = uidoc.Document;

      string path = doc.PathName;

      BasicFileInfo info = BasicFileInfo.Extract(
        path );

      DocumentVersion v = info.GetDocumentVersion();

      int n = v.NumberOfSaves;

      Util.InfoMsg( string.Format(
        "Document '{0}' has GUID {1} and {2} save{3}.",
        path, v.VersionGUID, n,
        Util.PluralSuffix( n ) ) );

      return Result.Succeeded;
    }
  }
```

Running it on my trusty old HVAC sample project that I have been maintaining ever since Revit 2008 produces the following result:

```
  Document 'Z:\a\rvt\hvac_project_2015.rvt' has GUID
  6359c822-15f3-44c3-83c5-f9bc258a3f90 and 338 saves.
```

This information is presented in a dialogue box like this:

![DocumentVersion GUID and NumberOfSaves](img/document_version.png)

By the way, another noteworthy aspect of this command is that it proves that the basic file info can be read successfully from a currently open document.

Thank you very much, Alexander, for this important hint!

#### Other Enhancements and Viewing Diffs in GitHub

I implemented a couple of other minor enhancements in The Building Coder samples that have not been explicitly pointed out here yet, e.g. a small check for the angle to true north
([release 2015.0.110.3](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.110.3))
to answer Frank Halliday's
[comment](http://thebuildingcoder.typepad.com/blog/2012/03/rotate-true-north.html?cid=6a00e553e16897883301a73e03c977970d#comment-6a00e553e16897883301a73e03c977970d) and
starting work on removing all obsolete API usage.

So far, in
[release 2015.0.110.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.110.2), I reduced the warning count from 71 to 67.

The changes made for that can be seen by navigating to
[release 2015.0.110.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.110.1) and
clicking on the label saying
[N commits](https://github.com/jeremytammik/the_building_coder_samples/compare/2015.0.110.1...master).

After that, you can edit the exact versions being compared to narrow down the differences to your window of interest, e.g. from 2015.0.110.1 to 2015.0.110.2.

#### Download The Building Coder Samples

The complete source code, Visual Studio solution and RvtSamples include file is provided in
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples).

The version discussed above is
[release 2015.0.111.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.111.0).