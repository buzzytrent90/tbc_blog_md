---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.3
content_type: qa
optimization_date: '2025-12-11T11:44:16.725261'
original_url: https://thebuildingcoder.typepad.com/blog/1777_tls_pdf_dae.html
post_number: '1777'
reading_time_minutes: 8
series: general
slug: tls_pdf_dae
source_file: 1777_tls_pdf_dae.md
tags:
- geometry
- revit-api
- rooms
- sheets
- views
title: Tls Pdf Dae
word_count: 1667
---

### TLS Requirement, PDF Data and Collada DAE Exporter
An important Revit add-in requirement regarding Transport Layer Security (TLS) settings, a short note on accessing PDF image data from an import instance, and an update of the Collada DAE custom exporter for use in Revit 2020:
- [Required Transport Layer Security (TLS) settings](#2)
- [Background](#2.1)
- [Requirement](#2.2)
- [Recommendation](#2.3)
- [Accessing imported PDF image data in Revit](#3)
- [Custom Collada exporter updated and fixed](#4)
![Transport Layer Security](img/transport_layer_security.png)
#### Required Transport Layer Security (TLS) Settings
The following is an official communication from the Revit Architect Team and API Guild to the Revit Developer Community on a \*Requirement and Recommendation for Transport Layer Security (TLS) Setting in Revit Add-ons\*:
##### Background
During a short period following August 3, 2019, when Autodesk Identity and Revit Cloud Worksharing Services moved to TLS 1.2 and stopped supporting TLS 1.0 and TLS 1.1, several Revit users reported that their Revit failed to communicate with Revit Cloud Worksharing Services, even though the required updates for Revit had been correctly applied. There were dumps in Revit journals like below
- `IOException`: Unable to read data from the transport connection: An existing connection was forcibly closed by the remote host.
- `IOException` Authentication failed because the remote party has closed the transport stream.
- `SocketException`: An existing connection was forcibly closed by the remote host from System.
The root cause turned out to be in a specific add-on, which exclusively specified TLS 1.0 in its code.
This setting overrode the TLS configuration in Revit and caused Revit to lose the capability of using TLS 1.2.
Therefore, we suggest developers follow the requirement and recommendation below when a specific TLS version is required in an add-in.
##### Requirement
Do not hard-code any TLS version exclusively, by directly assigning the version to the application-wide property `ServicePointManager.SecurityProtocol`.
This will override Revit’s native TLS configuration, which inherits from the targeted .NET Framework, and supports TLS 1.0, 1.1 and 1.2.
The native TLS configuration is critical for Revit to communicate with various Autodesk cloud services.
Here is an example of a problematic setting:

```
  System.Net.ServicePointManager.SecurityProtocol
    = System.Net.SecurityProtocolType.Tls12;
```

You may specify a TLS version in your add-on, if the add-on could be possibly running on a version of Revit that doesn’t have an appropriate TLS security update. Please make sure the setting is inclusive, by using bitwise OR (logical OR) on the application-wide property `ServicePointManager.SecurityProtocol`.
Here is an example of a correct setting:

```
  System.Net.ServicePointManager.SecurityProtocol
    |= System.Net.SecurityProtocolType.Tls12;
```

##### Recommendation
We recommend you don’t specify the desired TLS version in the add-in, but always rely on Revit and the .NET Framework’s TLS support, if any of the following conditions is fulfilled:
- The add-in is targeted on Revit 2015 to 2019 and the desired TLS version is 1.0 or 1.1.
- The add-in is targeted on Revit 2020 or a future release and the desired TLS version is 1.0, 1.1, or 1.2.
#### Accessing Imported PDF Image Data in Revit
Here is a quick note on how to access image data from an imported PDF file:
\*\*Question:\*\* I'm fairly new to the Revit API and was wondering if there's a way to access the image data of an imported PDF in Revit, i.e., the image data contained of an `ImageType` and `ImageInstance`.
\*\*Answer:\*\* Please install [RevitLookup](https://github.com/jeremytammik/RevitLookup) and see what you can find out that way first.
\*\*Response:\*\* I did that. It was super helpful to learn and find the `ImageType` class. I don't see how to access the image data, though.
\*\*Answer:\*\* Does the import create an `ImportInstance`? Look at the [ImportInstance members](https://www.revitapidocs.com/2020/fcedeca0-0e52-6a5f-b716-1d92c0fbac62.htm).
\*\*Response:\*\* I found the `ImageType.GetImage` method thanks to RevitLookup.
So, I can just call `imageType.GetImage().Save(filepath)`.
#### Custom Collada Exporter Updated and Fixed
I conversed with Bernhard Van Renssen in the past few days, in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [export to DAE file format](https://forums.autodesk.com/t5/revit-api-forum/export-to-dae-file-format/m-p/9004878) and
in a [comment](https://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html#comment-4598898712) on
the [article on \*Graphics Pipeline Custom Exporter\*](https://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html).
That prompted me to migrate
the [Collada custom exporter](https://github.com/jeremytammik/CustomExporterCollada) to
Revit 2020, including some minor enhancements that ended up resolving a larger problem:
\*\*Question:\*\* Does the custom collada exporter work with Revit 2019 and 2020?
I did not manage to get it work on 2019; the plugin is there, but it does nothing when executed.
When debugging, it calls the `exporter.Export` function in the Command.cs file, but that always throws `ExternalApplicationException`.
By the way, is Collada still the best format to export to if you need the materials, uvs and textures?
\*\*Answer:\*\* Are you talking about
the [Custom Exporter to Collada](https://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html#5) for
Revit 2014, published in July 2013?
I would assume it should work pretty much unchanged.
Regarding the exception that it may throw, that issue has come up a couple of times in the past and is surprisingly easy to handle.
Simply add an exception handler to catch and handle the exception, and then ignore it.
In spite of throwing the exception, the custom exporter seems to work perfectly all right.
Here is a more detailed discussion and example of this:
- [Custom exporter execute may throw](https://thebuildingcoder.typepad.com/blog/2018/12/dynamo-symbol-vs-type-and-exporter-exception.html#5)
- [Wrap call to `exporter.Export` in an exception handler and all is well](https://github.com/jeremytammik/CustomExporterAdnMeshJson/commit/23a95aad8f4a3cca85a72b32e2b699bde1d46bcb)
Can you confirm that this solves the issue in your case as well?
Later:
I just noticed that I created a [GitHub repository for the Collada custom exporter](https://github.com/jeremytammik/CustomExporterCollada).
Its list of releases includes one saying, [added exception handler around call to exporter.Export](https://github.com/jeremytammik/CustomExporterCollada/releases).
Maybe you are already looking at this version including the exception handler wrapping the call to `Export`?
Regarding your question on whether Collada is the best format to export to:
I am anything but an expert on this, but I would probably go for the
new [glTF or GL Transmission Format](https://en.wikipedia.org/wiki/GlTF) nowadays.
Afaik, it supports everything that Collada does and more, plus is more modern, streamlined, simple and efficient.
In fact, I recently implemented a [room volume exporter that creates data for use in a glTF visualisation](https://thebuildingcoder.typepad.com/blog/2019/06/room-volume-gltf-generator.html).
\*\*Response:\*\* I use the GitHub version, both the one with the exception handler (2017.0.0.1) and the 2018.0.0.0 releases, and both of them for some reason do not work neither on Revit 2018 nor 2019 (it does nothing after I click the custom add-in in a 3D view).
Here are the steps that I took to recreate the scenario:
- Downloaded the 2018 zip release file, unzipped and opened the .sln file in Visual Studio 2017.
- Reimported the RevitAPI.dll and RevitAPIUI.dll file from the Revit folder, as they were not found when opening the .sln file initially
- Changed the target framework to .NET Framework 4.7.1 (latest that I have downloaded on this machine)
- Built the project, added the CustomExporterCollada.dll as well as the CustomExporterCollada.add-in to the add-in folder of Revit 2019
- Launched Debug for Revit 2019 in Visual studio, opened a demo project and launched the add-in from the external tools icon with a breakpoint on the exporter.Export line of Command.cs in Visual Studio
- Stepped through the code from here until the exception message is called.
I hope there is something basic that I'm missing? Anyway, thank you for your time thus far, I appreciate every bit of your work and help, makes it really accessible for a newcomer to get started with developing plugins!
Update:
When debugging the project, in the `MyExportContext` file, in the `Start` method, I can step through the code in the debugger up to the line initialising the `StreamWriter`:

```
  streamWriter = new StreamWriter(...);
```

After that, the `WriteXmlColladaBegin` and `WriteXmlAsset` methods are never called.
The code then jumps to the `Finish` method and only executes `WriteXmlLibraryGeometriesEnd` before returning to the Command script and throwing the exception.
The file thus never goes through the process of writing any geometry data to the file.
Would you perhaps know why this is?

```
  public bool Start()
  {
    CurrentPolymeshIndex = 0;
    polymeshToMaterialId.Clear();

    streamWriter = new StreamWriter(_filepath);

    WriteXmlColladaBegin();
    WriteXmlAsset();

    WriteXmlLibraryGeometriesBegin();

    return true;
  }

  public void Finish()
  {
    WriteXmlLibraryGeometriesEnd();

    WriteXmlLibraryMaterials();
    WriteXmlLibraryEffects();
    WriteXmlLibraryVisualScenes();
    WriteXmlColladaEnd();

    streamWriter.Close();
  }
```

\*\*Answer:\*\* Try creating the `StreamWriter` earlier on, already in the constructor, so that it is already set up and ready to go before beginning the actual export. Maybe there is some strange conflict between the .NET `StreamWriter` creation and the Revit API export context.
I moved the `StreamWriter` initialisation into the custom exporter constructor and tested successfully my end.
All works well now for me now.
I captured this state in [CustomExporterCollada release 2020.0.0.2](https://github.com/jeremytammik/CustomExporterCollada/releases/tag/2020.0.0.2).
Can you confirm that it works for you too? Thank you!
\*\*Response:\*\* Yes. Thank you Jeremy, the 2020.0.0.2 works perfect now.
Thank you for your help, really appreciate it!
\*\*Answer:\*\* Thank you for the confirmation.
I just did what came to mind as you described your debugging observation and the weird jump from the stream writer initialisation to the end of processing.
What luck!