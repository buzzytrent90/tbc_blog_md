---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.8
content_type: qa
optimization_date: '2025-12-11T11:44:16.248928'
original_url: https://thebuildingcoder.typepad.com/blog/1540_q4r4_lookup.html
post_number: '1540'
reading_time_minutes: 6
series: general
slug: q4r4_lookup
source_file: 1540_q4r4_lookup.md
tags:
- elements
- filtering
- levels
- parameters
- revit-api
- rooms
- sheets
title: Q4r4 Lookup
word_count: 1214
---

### Elasticsearch-Head, RevitLookup and Area Schemes
I ran the first query on the collection
of [tbc](https://github.com/jeremytammik/tbc) blog
posts imported into Elasticsearch to experiment for the question answering
system [Q4R4 \*Question Answering for Revit API\*](http://thebuildingcoder.typepad.com/blog/2017/03/q4r4-revit-api-question-answering-system.html).
No spectacular results to report so far, but at least it works.
I installed the [elasticsearch-head](https://github.com/mobz/elasticsearch-head) web
front end to better explore and understand my local Elasticsearch cluster.
Alexander made a small correction to the latest RevitLookup enhancements, reverting one of the changes made yesterday.
Lots of interesting solutions on
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
including a nice little filtering sample that I picked up:
- [Elasticsearch text field mapping](#2)
- [Elasticsearch-head web front end](#3)
- [More RevitLookup updates](#4)
- [Get area scheme from an area](#5)
#### Elasticsearch Text Field Mapping
Yesterday, I described
the [q4r4 tbc import script `tbcimport.py`](http://thebuildingcoder.typepad.com/blog/2017/03/q4r4-tbc-import-and-revitlookup.html) that
I implemented to import all The Building Coder blog posts
into [Elasticsearch](https://www.elastic.co/products/elasticsearch) to
start experimenting with queries on them.
By default, the blog post `text` field was apparently imported and populated as a type `keyword` field:

```
$ curl -XGET 'localhost:9200/tbc/_mapping?pretty'
{
  "tbc" : {
    "mappings" : {
      "blogpost" : {
        "properties" : {
          "date" : {
            "type" : "date"
          },
          "nr" : {
            "type" : "long"
          },
          "text" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "title" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "url" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
}
```

Since I want to run a full text search on the blog post text, I need to change that mapping.
Actually, I might as well change it for the `title` field as well:

```
$ curl -XPUT 'localhost:9200/tbc?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "blogpost": {
      "properties": {
        "text": { "type":  "text" },
        "title": { "type":  "text" }
      }
    }
  }
}
'
```

Now the mapping looks more suitable:

```
$ curl -XGET 'localhost:9200/tbc/_mapping?pretty'
{
  "tbc" : {
    "mappings" : {
      "blogpost" : {
        "properties" : {
          "date" : {
            "type" : "date"
          },
          "nr" : {
            "type" : "long"
          },
          "text" : {
            "type" : "text"
          },
          "title" : {
            "type" : "text"
          },
          "url" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
}
```

#### Elasticsearch-Head Web Front End
Typing `curl` scripts on the command line is probably not the most effective way to explore my local Elasticsearch cluster.
I installed the [elasticsearch-head](https://github.com/mobz/elasticsearch-head) web front end to improve and simplify my interactive access.
As explained in its readme documentation, I must add two CORS settings to the elasticsearch config file `elasticsearch.yml` to allow elastic-head to connect to it:

```
  http.cors.enabled: true
  http.cors.allow-origin: "*"
```

With those settings in place, I can browse the cluster contents and start experimenting with queries on them:
![Elasticsearch-head](img/elasticsearch-head.png)
#### More RevitLookup Updates
Two minor RevitLookup updates today, each encapsulated in an own new version.
Alexander Ignatovich, [@CADBIMDeveloper](https://github.com/CADBIMDeveloper), aka Александр Игнатович,
disagreed with one of the changes made yesterday and reverted that in his pull request
[#31 try-catch for each element in cycle is bad idea and looks ugly](https://github.com/jeremytammik/RevitLookup/pull/31).
Many thanks to Alexander for paying attention and fixing this!
That change is merged
in [RevitLookup release 2017.0.0.21](https://github.com/jeremytammik/RevitLookup/releases/tag/2017.0.0.21).
I was unhappy with a couple of warning messages during compilation and fixed those
in [RevitLookup release 2017.0.0.22](https://github.com/jeremytammik/RevitLookup/releases/tag/2017.0.0.22).
The most up-to-date version is always provided in the master branch of
the [RevitLookup GitHub repository](https://github.com/jeremytammik/RevitLookup).
If you would like to access any part of the functionality that was removed when switching to the `Reflection` based approach, please grab it
from [release 2017.0.0.13](https://github.com/jeremytammik/RevitLookup/releases/tag/2017.0.0.13) or earlier.
I am also happy to restore any other code that was removed and that you would like preserved.
Simply create a pull request for that, explain your need and motivation, and I will gladly merge it back again.
#### Get Area Scheme from an Area
A [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) question
on [getting the area scheme from an area](https://forums.autodesk.com/t5/revit-api-forum/get-area-scheme-from-an-area/m-p/6949212) turned
out to be a pretty trivial matter of accessing and evaluating a simple series of parameter values on the area and area scheme:
\*\*Question:\*\* I'm not sure if this is possible but I've been trying to get the AREA_SCHEME_NAME from a collection of areas.
I've tried several ways without luck.
Can an area report what Area Scheme (Gross, Rentable) it belongs to?
\*\*Answer:\*\* Have you tried this?
```csharp
Area area;
AreaScheme scheme = doc.GetElement(
area.get_Parameter(
BuiltInParameter.AREA_SCHEME_ID ).AsElementId() )
as AreaScheme;
string areaSchemeName = scheme.get_Parameter(
BuiltInParameter.AREA_SCHEME_NAME ).AsString();
```
\*\*Response:\*\* Thanks.
Here's the code for a little test to get the areas that are on the 'Gross Building' scheme:
```csharp
IList areas = new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_Areas )
.OfClass( typeof( SpatialElement ) )
.Cast()
.ToList();
foreach( Element e in areas )
{
AreaScheme _scheme = doc.GetElement(
e.get_Parameter( BuiltInParameter.AREA_SCHEME_ID )
.AsElementId() ) as AreaScheme;
string _AreaSchemeName = _scheme.get_Parameter(
BuiltInParameter.AREA_SCHEME_NAME ).AsString();
if( _AreaSchemeName.ToString() == "Gross Building" )
{
TaskDialog.Show( "Revit", _AreaSchemeName );
double ox = e.LookupParameter( "Area" ).AsDouble();
TaskDialog.Show( "Revit", ox.ToString() );
}
else { continue; }
}
```
\*\*Answer:\*\* First, there are a couple of unnecessary inefficiencies in the sample code snippet.
There is no need for the `Cast<>`, and more importantly, `ToList` adds no value for this use case and consumes both time and memory, cf.:
- [FindElement and collector optimisation](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html)
- [Collect all rooms on a given level](http://thebuildingcoder.typepad.com/blog/2017/03/events-uv-coordinates-and-rooms-on-level.html#6)
I refactored the parameter accessing code as a separate little method to retrieve the area scheme name from the area element like this:
```csharp
///
/// Return the area scheme name of a given area element
/// using only generic Element Parameter access.
/// summary>
static string GetAreaSchemeNameFromArea( Element e )
{
if( !( e is Area ) )
{
throw new ArgumentException(
"Expected Area element input argument." );
}
Document doc = e.Document;
Parameter p = e.get_Parameter(
BuiltInParameter.AREA_SCHEME_ID );
if( null == p )
{
throw new ArgumentException(
"element lacks AREA_SCHEME_ID parameter" );
}
Element areaScheme = doc.GetElement( p.AsElementId() );
p = areaScheme.get_Parameter(
BuiltInParameter.AREA_SCHEME_NAME );
if( null == p )
{
throw new ArgumentException(
"area scheme lacks AREA_SCHEME_NAME parameter" );
}
return p.AsString();
}
```
With that in hand, the retrieval of all areas matching a given area scheme can we rewritten like this:
```csharp
///
/// Retrieve all areas belonging to
/// a specific area scheme.
/// summary>
public IEnumerable GetAreasInAreaScheme(
Document doc,
string areaSchemeName )
{
return new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_Areas )
.OfClass( typeof( SpatialElement ) )
.Where( e => areaSchemeName.Equals(
GetAreaSchemeNameFromArea( e ) ) );
}
```
I added these two methods to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
in [release 2017.0.132.10](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.132.10).
You can see the new code
by [comparing with the preceding release 2017.0.132.9](https://github.com/jeremytammik/the_building_coder_samples/compare/2017.0.132.9...2017.0.132.10).