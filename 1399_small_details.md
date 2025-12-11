---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.2
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.939374'
original_url: https://thebuildingcoder.typepad.com/blog/1399_small_details.html
post_number: '1399'
reading_time_minutes: 2
series: general
slug: small_details
source_file: 1399_small_details.md
tags:
- csharp
- doors
- elements
- family
- python
- revit-api
- sheets
- views
- walls
- windows
title: Small Details
word_count: 398
---

### Modelling Small Details
I am back from
the [BIM Programming](http://www.bimprogramming.com)
[conference and workshops in Madrid](http://thebuildingcoder.typepad.com/blog/2016/01/bim-programming-madrid-and-spanish-connectivity.html) and
rather flooded with overdue stuff, so here is just a quick note on how to you can model small details in Revit, if you really need to, courtesy
of Jose Ignacio Montes of [Avatar BIM](http://avatarbim.com).
As you are perfectly well aware, Revit will not allow you to model things smaller than 1/8th of an inch directly in the project environment, as we have seen by trying to create smaller and smaller line and direct shape elements until an exception is thrown:
- [Think Big in Revit](http://thebuildingcoder.typepad.com/blog/2009/07/think-big-in-revit.html)
- [DirectShape Performance and Minimum Size](http://thebuildingcoder.typepad.com/blog/2014/05/directshape-performance-and-minimum-size.html)
Jose presents a simple workaround using an imported DWG file:
Este es un veierteaguas de un remate de cubierta. Lleva dos tornillos que se repiten en más sitios, por lo que son 'detail items' insertados.
\*This shows a gutter cover. It has two little screws that are repeated in other places, so they are inserted as detail items.\*
![Revit design with small details](img/jim_small_detail_1.png)
El tornillo está formado por una máscara de líneas invisibles y un DWG importado con todo su detalle.
\*The screw is formed by a mask of invisible lines and an imported DWG with all its detail.\*
![Masking lines](img/jim_small_detail_2.png)

![Detailed screw DWG](img/jim_small_detail_3.png)
La familia de Clestra Metropoline usa el mismo sistema, pero dentro de un perfil de muro cortina. Este es un truco muy útil porque a poca gente se le ocurre meter un DWG dentro de un profile!
\*Our Clestra Metropoline family uses the same system, but within a curtain wall profile. This is a very useful trick that few people are aware of, to embed a DWG within a profile!\*
Lo mismo puede hacerse con ventanas, puertas o cualquier otro elemento que tenga detalles muy finos.
\*You can use the same idea with windows, doors or any other element that includes very fine detail.\*
Samples:
- [MODULO_VIERTEAGUAS.rfa](zip/MODULO_VIERTEAGUAS.rfa) – gutter cover family
- [CLESTRA_METROPOLINE.rvt](zip/CLESTRA_METROPOLINE.rvt) – Clestra Metropoline curtain wall profile
Many thanks to Jose for sharing this nice approach!