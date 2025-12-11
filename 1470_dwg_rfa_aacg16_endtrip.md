---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: qa
optimization_date: '2025-12-11T11:44:16.098620'
original_url: https://thebuildingcoder.typepad.com/blog/1470_dwg_rfa_aacg16_endtrip.html
post_number: '1470'
reading_time_minutes: 5
series: family
slug: dwg_rfa_aacg16_endtrip
source_file: 1470_dwg_rfa_aacg16_endtrip.md
tags:
- elements
- family
- geometry
- revit-api
- sheets
- views
title: The Building Coder
word_count: 908
---

### Avoid CAD Import in RFA, AAG16 and Endtrip
Some interesting topics of the day for this first cloudy one after a truly wonderful Indian summer:
- [Think twice before importing CAD into RFA](#2)
- [Advances in Architectural Geometry 2016](#3)
- [Endtrip full sound replacement](#4)
#### Think Twice Before Importing CAD into RFA
I was in a meeting with Sasha Crotty, Revit API Product Manager, last week, and she mentioned an important aspect of content creation that I was not previously fully aware of.

I initially summarised that under the title \*Avoid Imported CAD Content in RFA Family Definitions\* as follows:
Here is some general advice on creating Revit families: do not import DWG instances into a family definition.
CAD import into a family is considered bad content.
As much as possible, you would want to define a family using native Revit geometry native lines etc., not imported CAD data.
An imported CAD object in a family definition will cause performance issues and slow down Revit.
It gets duplicated in the project for every family instance placed.
Obviously the degradation will depend on the detail and complexity of the CAD content, and the replication of the family instances.
In general, though, it is to be avoided.
As if we couldn't have guessed...
Sasha asked me to correct that, saying:

Suggestion – Think twice before importing CAD into RFA
Here is some general advice on creating Revit families: consider if there is a better solution than importing DWG instances into a family definition.
Some CAD imports into a family may cause performance issues in the project.
When importing CAD, consider using native Revit geometry instead, particularly for meshes with a high vertex count.
An imported CAD object in a family definition will cause performance issues and slow down Revit.
The view navigation performance impact of an unnecessarily complex import grows in the project for every family instance placed.
Obviously the degradation will depend on the detail and complexity of the CAD content, and the replication of the family instances.
In general, if you're seeing performance degradations, consider whether there is a simpler way to model the geometry.
#### Advances in Architectural Geometry 2016
The [AAG16](http://www.aag2016.ch) conference on advances in architectural geometry at the ETH Zurich just completed.
For some exciting computational generative architectural innovation idea, you can take a look at the wonderful, impressive, beautiful and
exciting [Advances in Architectural Geometry 2016](http://vdf.ch/advances-in-architectural-geometry-2016-e-book.html) e-book.
To give you a quick first impression, here is its table of contents:
- Analysis and Design of Curved Support Structures
- Measuring and Controlling Fairness of Triangulations
- Face-Offsetting Polygon Meshes with Variable Offset Rates
- Marionette Mesh: From Descriptive Geometry to Fabrication-Aware Design
- Designing with Curved Creases: Digital and Analog Constraints
- A Double-Layered Timber Plate Shell: Computational Methods for Assembly, Prefabrication, and Structural Design
- On the Hierarchical Construction of SL Blocks: A Generative System that Builds Self-Interlocking Structures
- Tree Fork Truss: Geometric Strategies for Exploiting Inherent Material Form
- Textile Fabrication Techniques for Timber Shells: Elastic Bending of Custom-Laminated Veneer for Segmented Shell Construction Systems
- Bending-Active Plates: Form and Structure
- Underwood Pavilion: A Parametric Tensegrity Structure
- Safra Neuron Screen: Design and Fabrication
- Scissor Mechanisms for Transformable Structures with Curved Shape: The 'Jet d'Eau' Movable Footbridge in Geneva
- Mastering the 'Sequential Roof': Computational Methods for Integrating Design, Structural Analysis, and Robotic Fabrication
- Adaptive Meshing for Bi-directional Information Flows: A Multi-Scale Approach to Integrating Feedback between Design, Simulation, and Fabrication
- Dimensionality Reduction for Parametric Design Exploration
- Force Adaptive Hot-Wire Cutting: Integrated Design, Simulation, and Fabrication of Double-Curved Surface Geometries
- Designing for Hot-Blade Cutting: Geometric Approaches for High-Speed Manufacturing of Doubly-Curved Architectural Surfaces
- Cuttable Ruled Surface Strips for Milling
- The Armadillo Vault: Computational Design and Digital Fabrication of a Freeform Stone Shell
- CASTonCAST Shell Structures: Realisation of a 1:10 Prototype of a Post-Tensioned Shell Structure from Precast Stackable Components
- Lightweight Conical Components for Rotational Parabolic Domes: Geometric Definition, Structural Behaviour, Optimisation and Digital Fabrication
![AAG16 BRG](img/aag16_brg.png)
#### Endtrip Full Sound Replacement
Let's round off for today with an impressive demonstration of the importance of an audio track complementing a video experience.
One of my offspring studied sound design at the [Vancouver Film School](http://vfs.edu/programs/sound-design) last year.
They then launched a career as an independent computer game sound designer, only to veer off into a more programming oriented direction once they realised that the most efficient approach to accomplishing their goals would require them to implement their own tools in the Unity gaming engine first anyway.
Anyway, back to the VFS:
For the final exam, they picked a five-minute video equipped with a sound track and implemented a full sound replacement for it.
This involves music, a cast of voice characters, background sound replacement, synchronisation, etc.
It is a bit harder than you might initially imagine :-)
Here is the original video and the full sound replacement version side by side:
- [Endtrip – Original version by The Outpost](https://vimeo.com/73537360)
- [Endtrip – full sound replacement](https://vimeo.com/148054745)

[Endtrip (full sound replacement)](https://vimeo.com/148054745) on [Vimeo](https://vimeo.com).

For the sake of completeness, yet another version exists as well, [Endtrip Rescored by Sergio Ramis](https://www.youtube.com/watch?v=WjLWsTdISfY).
I found the comparison utterly fascinating; maybe you will too?