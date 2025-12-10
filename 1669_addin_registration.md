---
post_number: "1669"
title: "Addin Registration"
slug: "addin_registration"
author: "Jeremy Tammik"
tags: ['revit-api', 'sheets', 'views']
source_file: "1669_addin_registration.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1669_addin_registration.html"
---

### Add-In Registration – VendorId and Signature
I am taking lots of time off in July, so this may be my last post for a while.
Before leaving, I will share some answers to a list of pertinent questions on add-in registration, especially how to populate the add-in manifest `VendorId` tag and handle the trusted digital DLL signature:
- [Add-In Registration – `VendorId`](#2)
- [Add-In Registration – Trusted Digital Add-in Signature](#3)
- [Vacation in July](#4)
![Identification](img/identification.jpg)
#### Add-In Registration – VendorId
\*\*Question:\*\* What should we be specifying for our `VendorId`?
Can it be something like the explicit app id used for iOS and Android?
Apple recommends using a 'reverse-domain style' string for the app id suffix, e.g., 'com.yourcompany.yourapp'.
\*\*Answer:\*\* Yes, exactly! You can see this very recommendation in
the [developer guide instructions on Add-in Registration](http://help.autodesk.com/view/RVT/2019/ENU/?guid=Revit_API_Revit_API_Developers_Guide_Introduction_Add_In_Integration_Add_in_Registration_html):
- `VendorId`: A unique vendor identifier that may be used by some operations in Revit (such as identification of extensible storage). This must be unique, and thus we recommend using a reversed version of your domain name, for example, com.autodesk or uk.co.autodesk.
\*\*Question:\*\* Does this need to be registered with Autodesk somewhere?
\*\*Answer:\*\* No.
You can use any symbol you like, and you are responsible yourself for its uniqueness.
There used to be a different system, the \*Autodesk Registered Developer Symbol, **RDS**\*, limited to four characters and registered with Autodesk. That system was invented by Jeremy Tammik in the timeframe of ADGE, the AutoCAD Developers Group Europe, in the early 1980's. It had to be short and somewhat user compatible, since it was included in numerous AutoCAD symbol names, which were limited to 32 characters at that time. Therefore, it required a centralised registration agency. It has since been terminated.
Using the inverted Internet URL requires no registration, since the real Internet URL is unique in itself.
#### Add-In Registration – Trusted Digital Add-in Signature
\*\*Question:\*\* Do we need to digitally sign the app if we are going through the Autodesk App Store to generate our installer?
\*\*Answer:\*\* Not necessarily, and yes, personally, I would highly recommend doing so.
The main repository of all public knowledge on this topic, the trusted digital add-in signature, is in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [code signing of Revit addins](http://forums.autodesk.com/t5/revit-api/code-signing-of-revit-addins/m-p/5981560).
Please also refer to the help documentation on the topic, including a section in the developer guide
on [digitally signing your add-in and making your own certificate for testing and internal use](http://help.autodesk.com/view/RVT/2019/ENU/?guid=Revit_API_Revit_API_Developers_Guide_Introduction_Add_In_Integration_Digitally_Signing_Your_Revit_Add_in_html).
#### Vacation in July
I am off on vacation in July.
I'll start next week, spending some time with some friends in a hut in the Swiss mountains in Sulwald in the Lauterbrunnental.
Later, I will do some leisurely travelling and camping in France, on my way to a one-week visit to practice awareness, care and attentiveness in
the [Buddhist monastery Plum Village](https://plumvillage.org) near Bordeaux, founded
by the Vietnamese monk and Zen master [Thich Nhat Hanh](https://plumvillage.org/about/thich-nhat-hanh).
Please take good care of yourself during my absences :-)
![View of the Jungfrau Mountain from Sulwald](img/jungfrau_mountain.jpg)