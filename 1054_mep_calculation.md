---
post_number: "1054"
title: "User MEP Calculation Sample on GitHub"
slug: "mep_calculation"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api']
source_file: "1054_mep_calculation.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1054_mep_calculation.html"
---

### User MEP Calculation Sample on GitHub

I published the
[user MEP calculation sample](http://thebuildingcoder.typepad.com/blog/2013/07/user-mep-calculation-sample.html) a
couple of months ago as an example of making use of the
[external services framework](http://thebuildingcoder.typepad.com/blog/2012/05/the-revit-2013-mep-api-and-external-services.html#3) to
redefine the algorithms used for internal calculations in Revit.

Nmulder is happily making use of this to redefine pipe pressure loss calculations, but running into a
[generic add-in error on HTML generation](http://forums.autodesk.com/t5/Revit-API/Generic-add-in-error-on-HTML-generation/td-p/4601525) that
I hoped to be able to sort out:

**Question:** I have been playing around with the Revit API recently in order to provide alternative pipe pressure loss calculations for some of our users.
Much thanks to the code provided by
[Jeremy Tammik](http://thebuildingcoder.typepad.com/blog/2013/07/user-mep-calculation-sample.html) or
this never would have been possible for me.
I have made the necessary code changes for our calculation needs which is working.

My problem is really more of an annoyance.
We cannot export a pipe pressure loss report to HTML.
It reports this error:

![XSLT not found error](img/user_mep_calc_xslt_error.png)

It is complaining that the required .xslt file it is looking for is not at the specified location but it is:

![Add-in location](img/user_mep_calc_addin_location.png)

I have done what debugging and code checking I know how to do but can't fix it.

We can export to .csv just fine.
I just don't like a fully working piece of code.

Thanks!

**Answer:** You don't like fully working code?

Sounds like M$ and windoze to me... :-)

Anyway, since this is an important piece of base functionality, I took your question as an opportunity to re-examine the code and publish it on GitHub in order to simplify future enhancements.

It now lives in the
[UserMepCalculation GitHub repository](https://github.com/jeremytammik/UserMepCalculation).

I immediately created
[release 1.0.0.0](https://github.com/jeremytammik/UserMepCalculation/releases/tag/1.0.0.0) to
reflect the version published as a static archive file in the original
[user MEP calculation sample](http://thebuildingcoder.typepad.com/blog/2013/07/user-mep-calculation-sample.html),
then bumped the version to
[release 2014.0.0.0](https://github.com/jeremytammik/UserMepCalculation/releases/tag/2014.0.0.0) while
exploring your XSLT path issue.

The following code in the HtmlStreamWriter constructor generates the XSLT file path:

```csharp
  xmlFileName = System.IO.Path.GetDirectoryName(
    PressureLossReportHelper.instance.Doc
      .Application.RecordingJournalFilename );

  if( xmlFileName != null && xmlFileName.Length > 0 )
    xmlFileName = xmlFileName
      + "\\UserPressureLossReport"
      + DateTime.Now.Millisecond.ToString() + ".xml";

  string strPath = typeof(
    UserPressureLossReport.WholeReportSettingsDlg )
      .Assembly.Location;

  xsltFileName = Path.Combine(
    Path.GetDirectoryName( Path.GetDirectoryName(
      strPath ) ),
    "output", "UserPressureLossReport.xslt" );

  xmlWriter = XmlWriter.Create( xmlFileName );
```

On my system, the resulting file path ends up being
"C:/a/vs/UserMepCalculation/output/UserPressureLossReport.xslt".

Accordingly, I created a subfolder named 'output' in my UserMepCalculation root directory and copied the original PressureLossReport.xslt file included with Revit in the AddIns/MEP
Calculation directory to that location.

The following lines of code in the SaveDataToHTML.save method check the existence of this file:

```csharp
  // Check if the xslt file exists

  if (!File.Exists(writer.XsltFileName))
  {
    string subMsg = ReportResource.xsltFileSubMsg
      .Replace("%FULLPATH%", writer.XsltFileName );

    UIHelperFunctions.postWarning(
      ReportResource.htmlGenerateTitle,
      ReportResource.xsltFileMsg, subMsg );

    return false;
  }
```

For a moment, it appeared to me as if I could reproduce your problem, since I saw the following error message:

![My own XSLT renaming error](img/user_mep_calc_xslt_error_jt.png)

Alas, that problem was simply due to the fact that I forgot to rename the file properly:

```
  C:\a\vs\UserMepCalculation\output >
    ren PressureLossReport.xslt UserPressureLossReport.xslt
```

After renaming, everything worked as expected.

The full path of the missing XSLT file is not shown in your error message above.
Did you use the debugger to explore where exactly it really is looking?

You show the location of your XSLT file in the Revit add-ins folder.
Did you change the code **not** to look for it in an 'output' subdirectory?

I would agree that the code is somewhat inscrutable, and beg your pardon for that.

It was extracted from internal structures and still carries some excess baggage.

I removed as much of that as I could during my original cleanup.

The end result of this is that I cannot reproduce your issue or provide any help in resolving it, beyond asking you to look at the places mentioned above in the debugger and simplify them or adapt them better to your needs to resolve this issue.

One thing I would suggest trying out is to simply skip this error message once by using the 'set next statement' option in the debugger, and see whether the output is correctly produced anyway.
The error statement might be erroneous in itself, you know.

In any case, please feel free to fork or branch the
[UserMepCalculation GitHub repository](https://github.com/jeremytammik/UserMepCalculation) to
add any enhancements you would like to share, and I will hopefully be able to merge them into the main stream.

Thank you!