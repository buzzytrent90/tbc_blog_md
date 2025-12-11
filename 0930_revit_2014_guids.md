---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.918393'
original_url: https://thebuildingcoder.typepad.com/blog/0930_revit_2014_guids.html
post_number: 0930
reading_time_minutes: 6
series: general
slug: revit_2014_guids
source_file: 0930_revit_2014_guids.htm
tags:
- csharp
- revit-api
- windows
title: Revit 2014 Product GUIDs and GUID Algorithm
word_count: 1152
---

### Revit 2014 Product GUIDs and GUID Algorithm

Every year anew, people ask for the Revit product GUIDs used to identify the Revit installation and stored in the Windows registry under

- HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows\CurrentVersion\Uninstall

I already answered several queries from developers lately wishing to update their installers for the new version.

I discussed this topic on a yearly basis in the past for
[Revit 2010 and previous versions](http://thebuildingcoder.typepad.com/blog/2009/02/revit-install-path-and-product-guids.html),
[Revit 2011](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-product-guids.html),
[Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/04/product-guids-for-revit-2012-and-forever-more.html),
when the 'perpetual GUID algorithm' was introduced, and
[Revit 2013](http://thebuildingcoder.typepad.com/blog/2012/04/revit-2013-product-guids-and-guid-algorithm.html).

The install path is less of an issue since the introduction of the
[add-in manifest](http://thebuildingcoder.typepad.com/blog/2010/08/network-access-to-add-in-manifest-and-icons.html) to
inform Revit how to load your add-in, and the add-in manifest path is well known.

It is actually recommended to install .NET add-ins for both AutoCAD and Revit
[in or under the same directory as acad.exe and revit.exe](http://through-the-interface.typepad.com/through_the_interface/2009/11/updated-version-of-screenshot-now-available.html),
respectively: "It's safest to place assemblies at a location beneath the calling executable."

The best way to determine the Revit install location is to use the
[RevitAddInUtility](http://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html),
which provides an API for an installer to query that and also to generate an add-in manifest file from the relevant properties you specify.
The same RevitAddInUtility assembly DLL works transparently on both 32 and 64 bits.

#### Revit 2014 and Previous Product GUIDs

In spite of all this, the product GUID is still important, so here is a list of the Revit GUIDs up to and including Revit 2014:

|  |  |  |  |
| --- | --- | --- | --- |
| 2014 | 64 bit | RAC | 7346B4A0-1400-0110-0409-705C0D862004 |
| RST | 7346B4A0-1400-0210-0409-705C0D862004 |
| RME | 7346B4A0-1400-0310-0409-705C0D862004 |
| LT | 7346B4A0-1400-0410-0409-705C0D862004 |
| One-Box | 7346B4A0-1400-0510-0409-705C0D862004 |
| 32 bit | RAC | 7346B4A0-1400-0100-0409-705C0D862004 |
| RST | 7346B4A0-1400-0200-0409-705C0D862004 |
| RME | 7346B4A0-1400-0300-0409-705C0D862004 |
| LT | 7346B4A0-1400-0400-0409-705C0D862004 |
| One-Box | 7346B4A0-1400-0500-0409-705C0D862004 |
| 2013 | 64 bit | RAC | 7346B4A0-1300-0110-0409-705C0D862004 |
| RST | 7346B4A0-1300-0210-0409-705C0D862004 |
| RME | 7346B4A0-1300-0310-0409-705C0D862004 |
| One-Box | 7346B4A0-1300-0510-0409-705C0D862004 |
| 32 bit | RAC | 7346B4A0-1300-0100-0409-705C0D862004 |
| RST | 7346B4A0-1300-0200-0409-705C0D862004 |
| RME | 7346B4A0-1300-0300-0409-705C0D862004 |
| One-Box | 7346B4A0-1300-0500-0409-705C0D862004 |
| 2012 | 64 bit | RAC | 7346B4A0-1200-0110-0409-705C0D862004 |
| RST | 7346B4A0-1200-0210-0409-705C0D862004 |
| RME | 7346B4A0-1200-0310-0409-705C0D862004 |
| 32 bit | RAC | 7346B4A0-1200-0100-0409-705C0D862004 |
| RST | 7346B4A0-1200-0200-0409-705C0D862004 |
| RME | 7346B4A0-1200-0300-0409-705C0D862004 |
| 2011 | 64 bit | RAC | 94D463D0-2B13-4181-9512-B27004B1151A |
| RST | 23853368-22DD-4817-904B-DB04ADE9B0C8 |
| RME | C31F3560-0007-4955-9F65-75CB47F82DB5 |
| 32 bit | RAC | 4AF99FCA-1D0C-4D5A-9BFE-0D4376A52B23 |
| RST | 0EE1FCA9-7474-4143-8F22-E7AD998FACBF |
| RME | CCCB80C8-5CC5-4EB7-89D0-F18E405F18F9 |
| 2010 | 64 bit | RAC | 2A8EEE2F-4A9E-43d8-AA07-EC8A316B2DEB |
| RST | BC9C0A08-DEA4-4138-A7FB-8F68866DB0C1 |
| RME | A1BD042B-8A6F-4e37-92A3-78921BB45B05 |
| 32 bit | RAC | 572FBF5D-3BAA-42ff-A468-A54C2C0A17C3 |
| RST | 939D29FC-B82D-42a7-BB1E-8E3F121505CC |
| RME | 5C8281B1-B927-495a-A0FF-AB4BDFAE505C |
| 2009 | 64 bit | RAC | D2466208-7348-4214-B01E-7BC8729E2BD3 |
| RST | B354FCF5-CF64-4fa2-AA84-9D9B2A6FA649 |
| RME | 4A98F976-01B5-40e8-A496-AEFD85C3A446 |
| 32 bit | RAC | A3A37DA6-70C0-497C-BCB1-148E9EC1D32E |
| RST | C4B3B3C3-2EE9-48D3-9BF5-4443F7ECF759 |
| RME | E3781DCB-A650-4E66-9B74-67A1B17F052C |
| 2008 | RAC | 4A11206C-4377-49E8-911E-B11548658FF3 |
| RST | 8D0AE0BB-4FE5-491D-A284-3B363F02E639 |
| RME | 60A2743E-C881-4880-94ED-96445E38616F |
| 9.0 | Revit Building | D11DB6CB-0332-4735-B312-B919741D975E |
| 3 | RST | 3F11CEE0-D30D-41ce-8522-922B5D8BB324 |
| 8.1 | Revit Building | 7EBC0489-5E47-498D-BE31-B094484612E9 |
| 2 | RST | BE814F63-629D-4fd8-B628-1437AC10F9D4 |

#### Perpetual Revit Product GUID Algorithm

As you can see above, all the Revit GUIDs from Revit 2012 onwards have a strong similarity.

They are actually generated according to the following pattern:
```csharp
{7346B4At-uuvv-wwxy-zzzz-705C0D862004}
```

The alphabetical characters above stand for:

20. [0/1]: ProductCode/UpgradeCode- [12, 13, 14, ...]: Major release version [12, 13, 14]- [00, 01, ...]: Release subversion- [01/02/03/04/05/06/07]: RAC/RST/RME/LT/One-Box/Server/Vasari- [0/1]: Win32/x64- [0/1]: Global/Language Pack- [0409...]: Language

---

Internal only:

20. [0/1/2/3/...]: ProductCode/UpgradeCode/Patch Version (2 == UR1)

---

The 'ww' item corresponds to the ProductType enumeration provided by the Revit API, with the following members:

- Architecture
- Structure
- MEP
- Revit
- LT
- Unknown

The last item, represented by 'z', identifies the language.
It uses the hexadecimal
[LCID](http://msdn.microsoft.com/en-us/library/cc233965.aspx) or
language code identifier.
The ones supported by Revit are listed in the following table:

| lang | .NET code | HEX LCID | DEC LCID | Language | Codepage |
| --- | --- | --- | --- | --- | --- |
| chs | zh-CN | 0804 | 2052 | Chinese Simpl. | 936 |
| cht | zh-TW | 0404 | 1028 | Chinese Trad. | 950 |
| csy | cs-CZ | 0405 | 1029 | Czech | 1250 |
| deu | de-DE | 0407 | 1031 | German | 1252 |
| enu | en-US | 0409 | 1033 | English | 1252 |
| esp | es-ES | 040A | 1034 | Spanish | 1252 |
| fra | fr-FR | 040C | 1036 | French | 1252 |
| hun | hu-HU | 040E | 1038 | Hungarian | 1250 |
| ita | it-IT | 0410 | 1040 | Italian | 1252 |
| jpn | ja-JP | 0411 | 1041 | Japanese | 932 |
| kor | ko-KR | 0412 | 1042 | Korean | 949 |
| plk | pl-PL | 0415 | 1045 | Polish | 1250 |
| ptb | pt-BR | 0416 | 1046 | Portuguese Brazil | 1252 |
| rus | ru-RU | 0419 | 1049 | Russian | 1251 |

Applying this algorithm to the different flavours of Revit from Revit 2012 onwards produces the values listed in the table above.

#### Windows Registry Keys

The installer creates the following registry keys:

```
  HKEY_LOCAL_MACHINE\SOFTWARE\Autodesk\Revit
  \[RELEASEYEAR]\REVIT-[PRODID]:[LANGID]

  HKEY_LOCAL_MACHINE\SOFTWARE\Autodesk\Revit
  \Autodesk Revit [RELEASEYEAR]
  \ServicePack[SERVICEPACK#]
```

RELEASEYEAR is the major version number, i.e. 2013, 2014, etc.,
PRODID specifies the flavour, listed as 'ww' above, and
LANGID the locale, 'zzzz' above,
SERVICEPACK# would be 1 for Update Release 1, etc.

For Revit 2014, the Windows registry key is

```
  HKEY_LOCAL_MACHINE\SOFTWARE
  \Autodesk\Revit\2014\REVIT-05:0409
```

#### MUI: Multiple Language Interface Update

**Addendum:** Revit 2014 is using a new Multiple Language Interface, MUI.

Therefore, you may encounter all-zero language codes besides the ones listed above.

For instance, the following GUID refers to the global MSI which is used for the base product in all locales:

- 7346B4A0-1400-0110-0000-705C0D862004

The corresponding ENU Language Pack MSI would be installed by either an ENU base install or installing the ENU Language Pack on top of a non-ENU base install and create this GUID instead:

- 7346B4A0-1400-0110-0409-705C0D862004

Appropriate keys are added as each additional language pack is installed.