---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: news
optimization_date: '2025-12-11T11:44:14.155900'
original_url: https://thebuildingcoder.typepad.com/blog/0551_debug_verbosity.html
post_number: '0551'
reading_time_minutes: 3
series: general
slug: debug_verbosity
source_file: 0551_debug_verbosity.htm
tags:
- csharp
- elements
- references
- revit-api
- rooms
- windows
title: Reducing Revit Debug Verbosity
word_count: 697
---

### Reducing Revit Debug Verbosity

Many people have complained about the amount of noise caused by the ribbon in the Visual Studio debug output window.

Kean Walmsley just pointed out that you can easily
[make AutoCAD less noisy when debugging](http://through-the-interface.typepad.com/through_the_interface/2011/03/making-autocad-less-noisy-when-debugging.html) by
adding a couple of lines to the *acad.exe.config* file, which lives in the same folder as the *acad.exe* you are debugging, such as *C:\Program Files\Autodesk\AutoCAD 2011*, to ask the WPF binding trace provider to lower the volume.

Since Revit uses the same internal ribbon principles as AutoCAD, I thought this might help for Revit as well.

Adding the lines suggested by Kean to my Revit MEP 2011 config file *Revit.exe.config* in the Program subfolder of the Revit MEP installation directory, by default *C:\Program Files\Autodesk\Revit MEP 2011\Program*, does indeed achieve a similar effect.

Here is my updated *Revit.exe.config* with the added lines in bold (to see the truncated lines in full, copy to a text editor):
```csharp
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <section name="uri" type="System.Configuration.UriSection, System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"/>
  </configSections>
  <system.net>
    <settings>
      <servicePointManager expect100Continue="false"/>
    </settings>
  </system.net>
  <runtime>
    <generatePublisherEvidence enabled="false"/>
  </runtime>
  <uri>
    <idn enabled="All"/>
    <iriParsing enabled="true"/>
  </uri>
  <system.serviceModel>
    <bindings>
      <basicHttpBinding>
        <binding name="BasicHttpBinding\_IThorService" closeTimeout="00:01:00"
          openTimeout="00:01:00" receiveTimeout="00:10:00" sendTimeout="00:01:00"
          allowCookies="false" bypassProxyOnLocal="false" hostNameComparisonMode="StrongWildcard"
          maxBufferSize="1572864" maxBufferPoolSize="524288" maxReceivedMessageSize="1572864"
          messageEncoding="Text" textEncoding="utf-8" transferMode="Buffered"
          useDefaultWebProxy="true">
          <readerQuotas maxDepth="32" maxStringContentLength="8192" maxArrayLength="1572864"
            maxBytesPerRead="4096" maxNameTableCharCount="16384" />
          <security mode="Transport">
            <transport clientCredentialType="None" />
          </security>
        </binding>
      </basicHttpBinding>
    </bindings>
    <client>
      <endpoint address="https://climateserver.autodesk.com/ThorService.svc"
                binding="basicHttpBinding" bindingConfiguration="BasicHttpBinding\_IThorService"
                contract="ThorReference.IThorService" name="BasicHttpBinding\_IThorService" />
    </client>
  </system.serviceModel>
  <appSettings>
    <add key="Provider" value="https://climateserver.autodesk.com/ThorService.svc"/>
    <add key="consumerKey" value="sampleconsumer"/>
    <add key="consumerSecret" value="samplesecret"/>
  </appSettings>
  <!-- added by jeremy, cf. http://through-the-interface.typepad.com/through\_the\_interface/2011/03/making-autocad-less-noisy-when-debugging.html?utm\_source=feedburner&utm\_medium=feed&utm\_campaign=Feed%3A+typepad%2Fwalmsleyk%2Fthrough\_the\_interface+%28Through+the+Interface%29 -->
  <system.diagnostics>
    <sources>
      <source name="System.Windows.Data" switchName="SourceSwitch">
        <listeners>
          <remove name="Default" />
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

It obviously works fine for Revit as well.
Here are two text files containing the debug output
[before](zip/debug_verbosity_default.txt) and
[after](zip/debug_verbosity_quieter.txt) reducing verbosity, with the following line, word and character counts produced by the
[wc, the Unix word count command](http://en.wikipedia.org/wiki/Wc_%28Unix%29):
```csharp
174 4019 50362 debug\_verbosity\_default.txt
56 630 11583 debug\_verbosity\_quieter.txt
```

The line and byte count is reduced to less than a third, and the word count to less than a sixth!

I immediately added these lines to my RAC and RST config files as well :-)

Many thanks to Kean for this useful discovery!

#### Room Renumbering Feedback

Adam Nagy received some very positive feedback on his
[recently mentioned](http://thebuildingcoder.typepad.com/blog/2011/03/room-renumbering-plugin-of-the-month.html)
[Room Renumbering tool](http://labs.blogs.com/its_alive_in_the_lab/2011/02/march-adn-plugin-of-the-month-roomreumbering-for-revit-now-available.html),
the first ever Revit plugin of the month:

- Great little plugin. - Nice tool, Room Renumbering. - Glad to see Revit apps showing up here! I can see this not only being quite useful as is, but merely the beginning of a lot of future possibilities. - Any help would be great as I go crazy renumbering & this would be a great pluggin I hoped.- Thank you very much, I can see why you guy's are the best... Revit Rocks...  *(after Adam quickly pointed out what might be causing installation problems)*- The tool is really user friendly unlike the Element Positioning subscription tool...
            Great work guys! Been waiting a long time for this one.

Based on the feedback and wishes received so far, Adam is already thinking about some possible enhancements...