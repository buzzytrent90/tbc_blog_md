---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: code_example
optimization_date: '2025-12-11T11:44:15.590119'
original_url: https://thebuildingcoder.typepad.com/blog/1242_gen_sel_filt_adjac.html
post_number: '1242'
reading_time_minutes: 4
series: general
slug: gen_sel_filt_adjac
source_file: 1242_gen_sel_filt_adjac.htm
tags:
- elements
- family
- filtering
- levels
- python
- references
- revit-api
- rooms
- selection
- views
- walls
title: Selection Filters, Adjacency and the Good Universe
word_count: 805
---

### Selection Filters, Adjacency and the Good Universe

Today, let's look at:

- [A generic selection filter implementation](#2)
- [Determining adjacent rooms and spaces](#3)
- [The good universe](#4)

#### A Generic Selection Filter Implementation

Yesterday, I presented my new
[JtPairPicker element pair selection utility class](http://thebuildingcoder.typepad.com/blog/2014/11/picking-pairs-and-dimensioning-family-instance-origin.html#4).
It included a templated selection filter class.
I later realised that I could make use of that in several other places as well, replacing the existing explicit Wall, CurveElement and Pipe selection filters by a generic JtElementsOfClassSelectionFilter<T> one instead:

```python
  /// <summary>
  /// Allow selection of elements of type T only.
  /// </summary>
  class JtElementsOfClassSelectionFilter<T>
    : ISelectionFilter where T : Element
  {
    public bool AllowElement( Element e )
    {
      return e is T;
    }

    public bool AllowReference( Reference r, XYZ p )
    {
      return true;
    }
  }
```

Pretty trivial, but still, it pleases me to replace three other classes with this single one.

It also took a moment to figure out how to correctly string together the sequence `... : ISelectionFilter where T : Element`...

As always, the most up to date version of The Building Coder samples is provided in
[its GitHub repository](https://github.com/jeremytammik/the_building_coder_samples),
and the version including the new JtElementsOfClassSelectionFilter class described above is
[release 2015.0.116.3](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.116.3).

#### Determining Adjacent Rooms and Spaces

Another question that regularly comes up is on determining the relationship between neighbouring rooms and spaces:

**Question:** I am looking at the acquisition of energetic data from Revit building models.
For a given room or space, I would like to obtain all the adjacent rooms and spaces on the same level and the ones directly above and below, together with their respective intersecting areas.

I was thinking of gathering all of them with their respective boundary faces, then searching for adjacencies and calculating potential neighbouring intersection areas in a second step. However, I am afraid that this could quickly get quite complicated and consequently slow.

Is there a quicker way to find adjacent rooms and spaces within a certain range? Could bounding box intersection filters help, for instance?

**Answer:** Congratulations on starting work on such an interesting problem.

You should definitely do some additional exploration from an end user point of view, asking product usage experts and application engineers.

As far as I know, some of this kind of data is available from the Revit BIM with no programming required, e.g. via the gbXML file.

Please also take a deep look at the tools and results provided by the
[BPA team](http://autodesk.typepad.com/bpa) for
building performance analysis.

As far as I know, [Dynamo](http://dynamobim.org) also provides a good entry point for that.

I would be surprised – and impressed – if you are aiming to implement anything – in the beginning stages or your exploration, in any case – that has not already been addressed by the BPA and Dynamo communities.

Finally, from the pure Revit API point of view, The Building Coder provides a topic list 5.2 on
[2D Booleans and Adjacent Areas](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.2) that I just updated for you and lists some previous discussions and explorations in this area.

#### Arno Gruen on the Good Universe

Almost the only regular journalistic media production that I enjoy reading is
[Das Magazin](http://blog.dasmagazin.ch),
a Saturday supplement included with several major Swiss daily newspapers.
I love it.

The last issue,
[nr. 45, November 8, 2014](http://blog.dasmagazin.ch/aktuelles_heft/n-45-2)
([PDF](http://blog.dasmagazin.ch/wp-content/uploads/2014/11/ma1445.pdf)),
included an interview with
[Arno Gruen](http://en.wikipedia.org/wiki/Arno_Gruen),
*Ueber das Boese*.

It ends on such a nice optimistic note that I would like to share my translation from German of it with you:

May I ask you a final crucial question: what is your stand on religion?   — I have nothing against religious feelings. I am not in favour of religions maintaining a power structure with the purpose of controlling people and distorting their thinking.

Aren't all religions like that?   — No. In America, C.G. Jung met some Pueblo Indians who prayed every morning towards a neighbouring mountain. The chief told him, "Our prayers help God heave the sun over the mountain." Jung aptly commented that these people felt equal to their God. That is something completely different from becoming a slave of one's religion. You have to find God in yourself, as already stated by many mystics in the past.

And how is it for you personally?   — I agree with Einstein that the universe is not without meaning. That is quite a religious feeling.

And what is that meaning?   — The world is not random. It is constructed to encourage development of the good.