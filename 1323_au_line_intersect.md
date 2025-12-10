---
post_number: "1323"
title: "Connect Desktop and Cloud & Intersecting Lines or Grid Segments"
slug: "au_line_intersect"
author: "Jeremy Tammik"
tags: ['csharp', 'geometry', 'revit-api', 'views']
source_file: "1323_au_line_intersect.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1323_au_line_intersect.html"
---

### Connect Desktop and Cloud & Intersecting Lines or Grid Segments

The call for Autodesk University proposals closes today, May 26th, and I quickly submitted my second one before the deadline falls.

I also have another contribution from Magson Leone to share with you, a little helper function for calculating (unbounded) grid line intersections:

- [Autodesk University proposal – connect desktop and cloud](#2)
- [Intersection of Unbounded Lines](#3)

#### Autodesk University Proposal – Connect Desktop and Cloud

As said, the AU 2015 call for proposals ends today, so now is your last chance to join the AU community of experts, contribute to industry excellence, creative design, and the future of making things around the world by submitting your OWN class proposal at [autode.sk/CFPAU2015](http://au.autodesk.com/speaker-resource-center/call-for-proposals).

I presented my first
[proposal for the annual 90-minute Revit API expert panel](http://thebuildingcoder.typepad.com/blog/2015/05/autodesk-university-q1-adn-labs-and-wizard-update.html#3) last week.

Here is my second one:

> **Connect Desktop and Cloud: Analyse, Visualise and Report Universal Component Usage**
>
> We describe the nitty-gritty programming details to implement a cloud-based system to analyse, visualise and report on universal component usage. The components can be Revit families used in BIM or any other kind of assets in any other kind of system. The focus is on the cloud-based database used to manage the component occurrences, either in global or project based coordinate systems. Searches can be made based on both keywords and geographical location. Models are visualised using pure WebGL, Three.js and the Autodesk View and Data API, providing support for online viewing and model navigation. The web app can be used in any browser and on any mobile device with no need to install any additional software whatsoever. This is an advanced class for experienced programmers.

I will be presenting this class together with my friend, colleague and manager
[Cyrille Fauvel](http://around-the-corner.typepad.com/adn/cyrille-fauvel.html),
and am looking forward to that very much.

As always, I plan to discuss and share my progress as we go along, with most of the web related stuff on
[The 3D Web Coder](http://the3dwebcoder.typepad.com/), of course.

#### Intersection of Unbounded Lines

Another contribution from Magson Leone, who recently provided the
[Revit API compatibility helper methods](http://thebuildingcoder.typepad.com/blog/2015/05/compatibilizar-entre-vers%C3%B5es-api-compatibility-helper.html):

Eu quero lhe fazer mais uma sugestão de postagem para seu blog.
A postagem que eu quero sugerir é sobre como pegar a intersecção entre dois seguimentos de reta.
Eu procurei exaustivamente sobre este assunto e não achei nada.
Até que eu resolvi colocar a mão na massa e criar uma solução para isto.

Here is another blog post suggestion, on calculating the intersection point between two grid segments.
I searched extensively for a solution with no results, so I decided to create my own.

Eu precisei desta solução quando estava programando uma função para o comando Aparar:

I need this to implement a stretch command:

![Stretch command](img/intersect_lines_3.png)

Na imagem abaixo está um exemplo: Temos dois seguimentos de reta ou duas curvas, mas não temos o ponto p5 onde as duas retas se cruzam:

Here is an example with two grid segments or two lines, and determining the point `p5` where they cross:

![Line intersection](img/intersect_lines_2.png)

Aplicando os conhecimentos de Geometria Analítica, eu criei o método abaixo:

Using some basic analytic geometry, I created this method:

```csharp
  private XYZ Intersection( Curve c1, Curve c2 )
  {
    XYZ p1 = c1.GetEndPoint( 0 );
    XYZ p2 = c1.GetEndPoint( 1 );
    XYZ p3 = c2.GetEndPoint( 0 );
    XYZ p4 = c2.GetEndPoint( 1 );

    double x = p1.X + ( p2.X - p1.X )
      \* ( ( ( p4.X - p3.X ) \* ( p3.Y - p1.Y )
      - ( p4.Y - p3.Y ) \* ( p3.X - p1.X ) )
      / ( ( p4.X - p3.X ) \* ( p2.Y - p1.Y )
      - ( p4.Y - p3.Y ) \* ( p2.X - p1.X ) ) );

    double y = p1.Y + ( p2.Y - p1.Y )
      \* ( ( ( p4.X - p3.X ) \* ( p3.Y - p1.Y )
      - ( p4.Y - p3.Y ) \* ( p3.X - p1.X ) )
      / ( ( p4.X - p3.X ) \* ( p2.Y - p1.Y )
      - ( p4.Y - p3.Y ) \* ( p2.X - p1.X ) ) );

    XYZ p5 = new XYZ( x, y, 0 );

    return p5;
  }
```

Many thanks to Magson for this neat little helper function!

I added it to
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[Util class](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs) and
simplified it a bit like this:

```csharp
  /// <summary>
  /// Return the intersection point between two
  /// unbounded lines defined by the start and end
  /// points of the two given curves. By Magson Leone.
  /// https://en.wikipedia.org/wiki/Line%E2%80%93line\_intersection
  /// </summary>
  public static XYZ Intersection( Curve c1, Curve c2 )
  {
    XYZ p1 = c1.GetEndPoint( 0 );
    XYZ q1 = c1.GetEndPoint( 1 );
    XYZ p2 = c2.GetEndPoint( 0 );
    XYZ q2 = c2.GetEndPoint( 1 );
    XYZ v1 = q1 - p1;
    XYZ v2 = q2 - p2;
    XYZ w = p2 - p1;

    double c = ( v2.X \* w.Y - v2.Y \* w.X )
      / ( v2.X \* v1.Y - v2.Y \* v1.X );

    double x = p1.X + c \* v1.X;
    double y = p1.Y + c \* v1.Y;

    XYZ p5 = new XYZ( x, y, 0 );

    return p5;
  }
```

As always, the most up-to-date version is provided in
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples),
and the version containing this new method described above is
[release 2016.0.120.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.120.1).