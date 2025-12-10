---
post_number: "0872"
title: "ToElementIds Performance"
slug: "toelementids_perf"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'levels', 'python', 'revit-api', 'views']
source_file: "0872_toelementids_perf.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0872_toelementids_perf.html"
---

### ToElementIds Performance

Conversion of a filtered element collector to an explicit .NET collection of elements or element ids is always costly and mostly avoidable and unnecessary.

Before getting to the details of that issue, here is another quick snapshot from our travels to the West European developer conferences.
As you either know or might guess, one of the main topics of these is cloud and mobile development, e.g. looking at topics such as the
[BIM 360 Glue REST API and SDK](http://thebuildingcoder.typepad.com/blog/2012/12/the-bim-360-glue-viewer-and-rest-api.html) and
my hands-on exploration of how to
[access and use it via Python](http://thebuildingcoder.typepad.com/blog/2012/12/bim-360-glue-rest-api-authentication-using-python.html).

Funnily enough, the lighting in the Paris Charles de Gaulle airport is designed along a cloud scheme, so here is very low quality picture of us almost touching the clouds at the airport gate preparing to board the flight to Milano:

![Jeremy in the clouds in Charles de Gaulle in Paris](file:///j/photo/jeremy/2012/2012-12-12_devdays_paris/jeremy_under_cloud_charles_de_gaulle_paris.jpeg)

Ora siamo arrivati in Milano per un'altra conferenza di sviluppatori.

Returning to the Revit API, I recently mentioned a couple of
[FindElement and collector optimisations](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html).

One of them is the fact that the conversion from a filtered element collector to a .NET collection is costly and should be avoided if possible.

Here is another observation related to that, brought up by [Guy Robinson](http://redbolts.com) and answered by Scott Conover of the Revit API development team:

**Question:** I sometimes use the ToElementIds method, e.g. like this:

```
  var fc = new FilteredElementCollector( doc )
    .OfSomething()
    .ToElementIds();

  foreach( var elemId in fc )
  {
    var element = doc.GetElement( elemId );
  }
```

I noticed that this is about 25% slower than accessing the elements directly as follows:

```
  var fc = new FilteredElementCollector( doc )
    .OfSomething();

  foreach (var elem in fc)
  {
      var element = elem;
  }
```

I am wondering why the ToElementIds approach is so much slower?

Shouldn't the two approaches be equivalent?

Is there any good reason to use ToElementIds at all?

Not surprisingly, this ~25% slower figure has been the same on my tests ever since I started comparing in Revit 2012.

**Answer:** This is not unexpected.

ToElementIds and ToElements allocate a new concrete .NET collection containing the elements passing the filter.
Then it returns this allocated collection to you, which involves a conversion from a native level collection to a managed object.
This will take extra time as compared to a direct iteration.

Why might you use the ToElementIds variants?
Well, you might need a concrete collection to store in memory.
Or you might need a collection to pass as input to an API method like Delete – which you should definitely not call from the middle of a foreach iteration unless you like unexpected behaviour.
This provides a shortcut to building this allocation.

For any sort of operation of the type "I want to read properties of each passing element" you can skip ToElements and the extra collection allocation.

Thank you Scott for these insights!

For one example of how this can be achieved, look at the aforementioned
[FindElement and collector optimisations](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html).
There are lots of other examples in many of The Building Coder discussions, since I consciously and consistently avoid this conversion whenever possible, i.e. almost always.