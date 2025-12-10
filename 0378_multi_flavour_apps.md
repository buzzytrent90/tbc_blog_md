---
post_number: "0378"
title: "Add-In Applications for Multiple Revit Products"
slug: "multi_flavour_apps"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api']
source_file: "0378_multi_flavour_apps.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0378_multi_flavour_apps.html"
---

### Add-In Applications for Multiple Revit Products

We recently discussed the
[add-in visibility mode](http://thebuildingcoder.typepad.com/blog/2010/05/addin-visibility-mode.html),
which only applies to external commands.
The Revit API does not provide a similar built-in facility to control the loading and availability of an external application.

This turned out to be an issue for Dave Echols of
[Hankins and Anderson, Inc.](http://www.ha-inc.com),
who very friendlily is sharing his solution to this lack with us here:

#### Managing Custom Applications for Multiple Revit Products

As a follow-up on and complement to the
[Add-In Visibility Mode](http://thebuildingcoder.typepad.com/blog/2010/05/addin-visibility-mode.html) posting,
our company faces a different issue. We have multiple Revit products installed on our workstations and use IExternalApplication custom applications to load our applications and commands. I use a modified version of the RvtSamples menu system included with the SDK to load our custom commands and build the Ribbon panels. Moving to the new .addin manifest installation process in Revit 2011 caused some initial concern. Our custom applications are organized by major discipline and each discipline is contained in its own class within out ApplicationLoader assembly. All the applications defined in the .addin file were being loaded in all three Revit products. As a result of this new organization, MEP Ribbon panels and commands were visible in the Architecture and Structural products. Our .addin manifest is shown below.
```csharp
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<RevitAddIns>
  <AddIn Type="Application">
    <Assembly>C:\Program Files\HA Programs\Revit 2011 AddIns\HA.ApplicationLoader.dll</Assembly>
    <ClientId>2ec9c787-d029-460b-8fd9-003ec80254ab</ClientId>
    <FullClassName>HA.ApplicationLoader.EventManager</FullClassName>
    <Name>Global Event Manager</Name>
  </AddIn>
  <AddIn Type="Application">
    <Assembly>C:\Program Files\HA Programs\Revit 2011 AddIns\HA.ApplicationLoader.dll</Assembly>
    <ClientId>44e53d5e-3b1d-4eb9-a29c-18c93f86ad04</ClientId>
    <FullClassName>HA.ApplicationLoader.HaLoader</FullClassName>
    <Name>Global Commands</Name>
  </AddIn>
  <AddIn Type="Application">
    <Assembly>C:\Program Files\HA Programs\Revit 2011 AddIns\HA.ApplicationLoader.dll</Assembly>
    <ClientId>f7ae3726-1987-40df-9aa5-225aa1e9c95f</ClientId>
    <FullClassName>HA.ApplicationLoader.ElectricalLoader</FullClassName>
    <Name>Electrical Commands</Name>
  </AddIn>
  <AddIn Type="Application">
    <Assembly>C:\Program Files\HA Programs\Revit 2011 AddIns\HA.ApplicationLoader.dll</Assembly>
    <ClientId>7d43fa76-f249-4453-9bd5-91b7601fb486</ClientId>
    <FullClassName>HA.ApplicationLoader.HvacLoader</FullClassName>
    <Name>HVAC Commands</Name>
  </AddIn>
  <AddIn Type="Application">
    <Assembly>C:\Program Files\HA Programs\Revit 2011 AddIns\HA.ApplicationLoader.dll</Assembly>
    <ClientId>a5b99559-a5ff-4fb5-863e-3031e5e715a2</ClientId>
    <FullClassName>HA.ApplicationLoader.LifeSafetyLoader</FullClassName>
    <Name>Life Safety Commands</Name>
  </AddIn>
  <AddIn Type="Application">
    <Assembly>C:\Program Files\HA Programs\Revit 2011 AddIns\HA.ApplicationLoader.dll</Assembly>
    <ClientId>0155a264-2682-4c62-9f15-c4a28fbc4417</ClientId>
    <FullClassName>HA.ApplicationLoader.PlumbingLoader</FullClassName>
    <Name>Plumbing Commands</Name>
  </AddIn>
</RevitAddIns>
```

(Copy and paste to an editor to see the complete truncated lines.)

When I first contacted ADN about this anomaly, I was pointed to the VisibilityMode tag as shown in the above posting. I pointed out I was working with applications, which do not support the VisibilityMode tag (maybe they should). As a consequence, I needed to find a code solution that would allow me to use the new installation process. After several attempts of trial and error, I was able to produce a solution that works well and is very simple to implement. Our OnStartup methods in each of our custom applications looked like the following in Revit 2010.
```csharp
public Result OnStartup(
  UIControlledApplication uiControlledApp )
{
  try
  {
    // add code here to load custom commands
    // and build the Ribbon Panel

    return Result.Succeeded;
  }
  catch( Exception e )
  {
    ErrorMsg( e.Message );
  }
  return Result.Failed;
}
```

This is very standard stuff and does not need to be changed if the application is global to all Revit products.
I made the following additions to check for the currently running Revit product and base the loading of the application on that result.
The code below should only create the Ribbon panel and load the commands if the application is running in the Revit MEP product.
```csharp
public Result OnStartup(
  UIControlledApplication uiControlledApp )
{
  Result rc = Result.Failed;

  try
  {
    // Control the loading of the AddIn
    // based on the product we are running

    ProductType pt = uiControlledApp
      .ControlledApplication.Product;

    // always check this

    if( pt == ProductType.Unknown )
    {
      return Result.Cancelled;
    }

    // we must be in RAC to continue.
    // Comment out if running MEP or RST

    //if( pt != ProductType.Architecture)
    //{
    // return Result.Cancelled;
    //}

    // we must be in MEP to continue.
    // Comment out if running RAC or RST

    if( pt != ProductType.MEP )
    {
      return Result.Cancelled;
    }

    // we must be in RST to continue.
    // Comment out if running RAC or MEP

    //if(pt != ProductType.Structure)
    //{
    // return Result.Cancelled;
    //}

    // add code here to load custom commands
    // and build the Ribbon Panel

    return Result.Succeeded;
  }
  catch( Exception e )
  {
    ErrorMsg( e.Message );
  }
  return Result.Failed;
}
```

By making the above changes in our custom application code and using the functionality in the new RevitAddInUtility.dll, I was able to create a single install script for all three Revit products and control the installation and uninstallation of our applications in a seamless manner.

Thank you very much, Dave, for sharing this solution with us!