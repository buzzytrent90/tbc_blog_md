---
post_number: "1169"
title: "Technical Summit Day 1 and Removing References"
slug: "ts_1_remove_reference"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'references', 'revit-api', 'views']
source_file: "1169_ts_1_remove_reference.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1169_ts_1_remove_reference.html"
---

### Technical Summit Day 1 and Removing References

Today was the first day of the Autodesk internal Technical Summit in Toronto, and pretty exciting.

Also, I just had a note from Norway on how to [remove references](#3) that looks worthwhile sharing.

So here is a list of today's multifaceted topics:

- [Recent rendering research](#3)
- [JavaScript optimisation](#4)
- [Spherical harmonics](#5)
- [Web app packages](#6)
- [Removing RVT links](#7)

#### Technical Summit Day 1

This being an internal meeting, I cannot talk openly about all the topics covered.

A couple of titbits that I found especially exciting are based on openly available research, though, so I can't do much harm mentioning those.

#### R3 – Recent Rendering Research

One talk discussed viewing and rendering and mentioned these exciting recent results that are all being incorporated:

- Normalized Blinn Shading – Hoffman, Naty, 'Crafting Physically Motivated Shading Models for Game Development', SIGGRAPH 2010
- HDR image based diffuse and reflection, with pre-computed BRDF stored in mipmaps – Lagarde, S.,
  [AMD Cubemapgen for physically based rendering](http://seblagarde.wordpress.com/2012/06/10/amd-cubemapgen-for-physically-based-rendering)
- Alchemy Ambient Obscurance – McGuire et al, “The Alchemy Screen-Space Ambient Obscurance Algorithm”, HPG 2011

#### JavaScript Optimisation

The same presentation also mentioned some impressive tips and tricks for enhancing JavaScript performance, e.g. encoding metadata requiring quick lookup in SQL tables, then mapping those tables to JSON arrays.

Further fundamental JavaScript performance optimisation recommendations include:

- Use worker threads for expensive operations
- Avoid allocating objects
- Avoid expensive data representation that requires heavy processing

- Use ArrayBuffer or Array instead of Object (hashmap) to hold collections
- Use
  [SoA instead of AoS representation](http://divergentcoder.com/Coding/2011/02/22/aos-soa-explorations-part-1.html) for
  large lists of things

- Dereference unused objects to get them garbage collected
- Cache the invariants (avoid repeated calls/references)
- The browsers have excellent profilers – use them!

#### Rotation Invariant Spherical Harmonics for Shape Classification

Check out this fascinating paper on
[Tools for 3D-Object Retrieval: Karhunen-Loeve Transform and Spherical Harmonics](http://kops.ub.uni-konstanz.de/bitstream/handle/urn:nbn:de:bsz:352-231423/Vranic_231423.pdf?sequence=3) by
D.V. Vranić, D. Saupe, and J. Richter.

For practical use, it obviously needs to implement some kind of grouping of buckets of similar shapes, which is a completely different additional story...

#### Highly Recommended Web App Packages

Are you building any kind of web application at all?

You certainly should be, nowadays, if you are a professional application developer.

If not, you should be aware that most bits and pieces of functionality of those 2 GB footprint desktop applications can be rebuilt in a browser with much smaller footprint in very short time. So if you don't do it, somebody else is going to pretty soon.

Here is a pretty comprehensive list of useful packages that help a lot in supporting numerous best practices for a web app:

- Angular JS – [angularjs.org](https://angularjs.org)
- Require JS – [requirejs.org](http://requirejs.org)
- Twitter Bootstrap – [getbootstrap.com](http://getbootstrap.com)
- CSS Less – [lesscss.org](http://lesscss.org)
- Yeoman – [yeoman.io](http://yeoman.io), including:

- Bower – [bower.io](http://bower.io) – Package Manager for managing dependencies for the web (i.e. Maven/NuGet)
- Grunt – [gruntjs.com](http://gruntjs.com) – Building Web Apps and Task Runner, e.g. for:

- Minification (JavaScript, HTML, CSS)
- Concatenate JavaScript Files
- JSDoc
- ESLint
- Less to CSS Conversion
- Running Unit Tests

- Testing

- Jasmine – [jasmine.github.io](http://jasmine.github.io) – Testing framework
- Karma – [karma-runner.github.io](http://karma-runner.github.io/0.12/index.html) – Test Runner
- Protractor – [github.com/angular/protractor](https://github.com/angular/protractor) – End to end test framework for Angular JS

By the way, Angular also provides the best tutorial for anything I have ever seen,
[Shaping up with Angular.js](http://angular.codeschool.com).

You absolutely must check it out, if only to be impressed by the level of didactics that is achievable today.

There was a lot more, of course... I am really looking forward to day 2!

#### Removing RVT Links

Back to the everyday Revit API, here is a question raised and immediately answered by Håvard Dagsvik of
[Cad Quality as](http://www.cad-q.com/no):

**Question:** I'm trying to remove RVT links from a project.

This code works fine:

```csharp
  public void RemoveRvtLinks( Document doc )
  {
    FilteredElementCollector elemColl
      = new FilteredElementCollector( doc )
        .OfClass( typeof( RevitLinkType ) );

    IList<ElementId> eIds = new List<ElementId>();

    foreach( RevitLinkType r in elemColl )
    {
      if( r is RevitLinkType )
      {
        RevitLinkType rvtLink = r as RevitLinkType;
        if( null == rvtLink || rvtLink.IsNestedLink )
        {
          continue;
        }
        eIds.Add( rvtLink.Id );
      }
    }
    doc.Delete( eIds );
  }
```

I'm using RevitLinkType instead of RevitLinkedInstance to avoid the Revit UI message saying that 'the last instance has been removed; do you want to remove the reference'.

The catch is that if the link is Unloaded, Not Found or for some reason not present both will throw an exception.

The API doesn’t seem to provide a 'Remove' method for links, only Document.Delete.

But in such cases there is nothing to delete.

Is this possible to do at all?

**Answer:** Apparently yes.

After further testing, I can confirm that the code above works both on unloaded and not found links.

Here is a cleaned-up and slightly more succinct version using LINQ avoiding the duplicate casts above:

```csharp
  public void RemoveRvtLinks2( Document doc )
  {
    FilteredElementCollector links =
      new FilteredElementCollector( doc )
        .OfClass( typeof( RevitLinkType ) );

    ICollection<ElementId> ids
      = links.Cast<RevitLinkType>()
        .Where<RevitLinkType>( r => !( r.IsNestedLink ) )
        .Select<RevitLinkType, ElementId>( r => r.Id )
        .ToList<ElementId>();

    doc.Delete( ids );
  }
```