---
post_number: "0044"
title: "RealDWG and Object Enablers"
slug: "realdwg_object_enablers"
author: "Jeremy Tammik"
tags: ['revit-api', 'views']
source_file: "0044_realdwg_object_enablers.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0044_realdwg_object_enablers.html"
---

### RealDWG and Object Enablers

Revit uses the RealDWG toolkit to implement the DWG file export and import functionality.
A frequent question is whether there is any way to hook into this functionality, and how this relates to custom objects and DBX object enablers, like for instance:
Is there a way to load a DBX application into Revit as a RealDWG extension?
It would be handy to be able to reuse the functionality of some custom ObjectDBX classes.

First of all, let us clarify some terms required in this context:

- **ObjectARX:** The AutoCAD Runtime Extension, originally just ARX.
- **ObjectDBX:** The AutoCAD-independent subset of ObjectARX that deals only with the AutoCAD database, the DWG file.
- **RealDWG:** An AutoCAD-independent library for reading and writing DWG files using ObjectDBX in a stand-alone application. Requires a separate license.
- **Custom object:** An instance of a C++ class that can be stored in an AutoCAD database, implemented by an ObjectARX or ObjectDBX plug-in.
- **Custom entity:** An custom object that defines graphics and can reside in the block table of an AutoCAD database.
- **Object enabler:** An AutoCAD-independent plug-in module, i.e. an ObjectDBX plug-in, that defines the properties and behaviour of a custom object or entity.
- **Proxy graphics:** Hard-coded graphics data stored in a DWG file and used to render a custom object in the absence of its object enabler.

ObjectARX supports extending AutoCAD during runtime with plug-in modules implemented in C++.
An ObjectDBX plug-in makes use of only an AutoCAD independent subset of ObjectARX.
It can therefore be loaded both inside of AutoCAD, and outside into other non-AutoCAD host applications such as TrueView.

More details about RealDWG are provided in the
[RealDWG Developer Center](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=770257).
Thanks to Adam Nagy from our DevTech team in Europe,
we also have a DevTV recording explaining how to make use of this API,
available for
[download](http://download.autodesk.com/media/adn/DevTV-Introduction-to-RealDWG-Programming.zip)
(97.5 MB) or for
[online viewing](http://download.autodesk.com/media/adn/DevTV-Introduction-to-RealDWG-Programming). Kean Walmsley provides a list of
[more DevTV recordings](http://through-the-interface.typepad.com/through_the_interface/devtv/index.html)
on various Autodesk APIs.

RealDWG itself does include a full ObjectARX runtime kernel which provides the ability to load an object enabler.
It needs it to manage the ARX class relationships, and it includes the acrxLoadModule API call to load a DBX module.
It obviously cannot load an ARX module, i.e. a module that depends on AutoCAD, because it is not AutoCAD.

On the other hand, RealDWG within Revit is a different story. In that context, no object enablers are loaded.
Revit only uses RealDWG to display the graphics of the primitive entities within the DWG file, without bothering to load object enablers for custom classes.
It does not load object enablers, even though RealDWG itself is theoretically capable of doing so.

For export, Revit uses RealDWG to generate DWG files with primitive built-in ObjectARX entities.
For importing DWG files, it uses it to read and display entities and other data from the DWG file.
It does not use any additional classes or do anything to manage the object hierarchy, so it does not load object enablers. All it uses is primitives already available in the RealDWG system.
All custom objects just render their proxy graphics.

When you call an ObjectARX entity's worldDraw function and its object enabler is not loaded, it just returns its proxy graphics.
If the enabler is loaded, the real entity calculates and returns its proper live graphics.
This functionality is automatically handled by worldDraw.

Revit does not register itself as a RealDWG application, and so the object enabler installers do not see it and do not set up any object enabler autoloading.

The result is that you cannot use the RealDWG functionality built in to Revit to load an object enabler or make use of ObjectDBX functionality.

You may be able to implement your own .NET based RealDWG application inside a Revit plug-in. This combination has not been tested, however, and is not explicitly supported. You may run into problems where the RealDWG licensing mechanism interferes with the Revit plug-in security features.

If this combination works, it would enable you to make use of the AcGe library within your Revit application.
It would also give you access to the acrxLoadModule, thus providing the complete required kernel functionality in able to load a DBX object enabler or handle custom objects in DWG files.
If anybody has succesfully made use of this combination, it would be interesting to hear from you and discuss further details.
However, there is another easy way to achieve the same result ...

A related topic is the possibility to run AutoCAD as a hidden process and make full use of all its API functionality from within Revit, which we will look at next.