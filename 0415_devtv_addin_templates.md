---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:13.911583'
original_url: https://thebuildingcoder.typepad.com/blog/0415_devtv_addin_templates.html
post_number: '0415'
reading_time_minutes: 4
series: general
slug: devtv_addin_templates
source_file: 0415_devtv_addin_templates.htm
tags:
- csharp
- references
- revit-api
- selection
- vbnet
- windows
title: DevTV Add-In Templates
word_count: 738
---

### DevTV Add-In Templates

Here are the add-in templates prepared by my colleague Augusto Gonçalves of Autodesk Brazil for the upcoming DevTV presentations.

I really love both the implementation and the description, both are so short and sweet and yet complete!

They save you a lot of typing and clicking when setting up a new add-in, and especially help avoid all the potential errors that insist on creeping in when you set things up manually.
Support is provided for both external commands and external applications.

These are the same templates we used during DevCamp, but with several changes and improvements, especially on the add-in manifest file creation.

#### How to use

Installation could hardly be simpler.
No need to unzip, just save the following two files in the specified locations on your local system:

- [TemplateRevitArchAddinCS.zip](zip/TemplateRevitArchAddinCS.zip): [My Documents]\Visual Studio 2008\Templates\ProjectTemplates\Visual C#- [TemplateRevitArchAddinVB.zip](zip/TemplateRevitArchAddinVB.zip): [My Documents]\Visual Studio 2008\Templates\ProjectTemplates\Visual Basic

You obviously replace '2008' by '2010' for Visual Studio 2010.

#### Benefits

- Works on all versions of VS, including Express.- Simple project with one ExternalApplication and one ExternalCommand, plus commonly used namespaces.- References to Revit assemblies (RevitAPI and RevitAPIUI) with Copy Local set to False.- Debug features configured and enabled (important for Express version).- Creates the required .addin including the
          [add-in manifest ClientId](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html),
          copies it to the appropriate Revit\Addins\2011 user specific folder (through post-build events) and deletes it again during Clean (through after-clean event), e.g. on using Visual Studio Build > Clean Solution.
          The <Assembly> mark is initially created with the Bin\Debug build.

#### Weaknesses

- The references path is partially hard coded, so Revit \*must\* be under [Program Files]\Autodesk\Revit Architecture 2011 (or the template requires changes).- As the path is hard coded, it does not work for other flavours. The workaround is create a separate template for each flavour (easy, just change the path).- Visual Express (only this version) creates the project at a temporary location ("as designed"), therefore the .addin file requires manual edit on the <Assembly> mark.

#### Example

The Visual Studio IDE will automatically detect, pick up and unpack the files when they are present in these directories, so the full functionality is available immediately after they have been placed there.
When you next create a new project, the template is presented for selection:

![Template in new project dialogue](img/devtv_template_new_project.png)

The newly created project is fully populated with the Revit API references, a sample command, application, and add-in manifest file:

![DevTV template project contents](img/devtv_template_project.png)

It is immediately ready for compilation.
On successful compilation, the
[add-in manifest](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html)
is created and copied to the proper location:

![DevTV template project build output](img/devtv_template_build.png)

Here is the build output in text format (copy to an editor to see the truncated lines):

```
------ Rebuild All started: Project: RevitNETAddin2, Configuration: Debug Any CPU ------
c:\WINDOWS\Microsoft.NET\Framework\v3.5\Csc.exe /noconfig /nowarn:1701,1702 /errorreport:prompt /warn:4 /define:DEBUG;TRACE /reference:"C:\Program Files\Autodesk\Revit Architecture 2011\Program\RevitAPI.dll" /reference:"C:\Program Files\Autodesk\Revit Architecture 2011\Program\RevitAPIUI.dll" /reference:"c:\Program Files\Reference Assemblies\Microsoft\Framework\v3.5\System.Core.dll" /reference:c:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\System.Data.dll /reference:c:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\System.Deployment.dll /reference:c:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\System.dll /reference:c:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\System.Drawing.dll /reference:c:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\System.Windows.Forms.dll /reference:c:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\System.Xml.dll /debug+ /debug:full /filealign:512 /optimize- /out:obj\Debug\RevitNETAddin2.dll /target:library Application.cs Commands.cs Properties\AssemblyInfo.cs

Compile complete -- 0 errors, 0 warnings
RevitNETAddin2 -> C:\a\doc\revit\devtv\test\RevitNETAddin2\RevitNETAddin2\bin\Debug\RevitNETAddin2.dll
copy "C:\a\doc\revit\devtv\test\RevitNETAddin2\RevitNETAddin2\RevitNETAddin2.addin" "C:\Documents and Settings\tammikj\Application Data\Autodesk\REVIT\Addins\2011\RevitNETAddin2.addin"
        1 file(s) copied.
========== Rebuild All: 1 succeeded, 0 failed, 0 skipped ==========
```

On cleaning the solution, the add-in manifest file is removed again, so that Revit does not complain about the missing add-in assembly DLL.