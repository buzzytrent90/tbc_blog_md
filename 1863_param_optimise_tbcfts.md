---
post_number: "1863"
title: "Param Optimise Tbcfts"
slug: "param_optimise_tbcfts"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'parameters', 'revit-api', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1863_param_optimise_tbcfts.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1863_param_optimise_tbcfts.html"
---

### Optimising Parameters and Full-Text Search
I have been dabbling with the Go programming language in the past week, besides optimising and answering Revit API questions:
- [Optimising setting shared parameters](#2)
- [Full-text search for The Building Coder posts](#3)
- [Decimal point woe](#4)
#### Optimising Setting Shared Parameters
I took a successful stab at optimising the setting of a large number of shared parameters in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [modifying shared parameters for a high number of family instances](https://forums.autodesk.com/t5/revit-api-forum/modify-shared-parameters-for-high-number-of-family-instance/m-p/9727166):
\*\*Question:\*\* I have the code below working fine when my collection contains less than 30 objects; above 30, the transaction commit doesn't affect the document and there is no error or exception raised.
```csharp
void IExternalEventHandler.Execute( UIApplication app )
{
UIDocument uidoc = app.ActiveUIDocument;
Document doc = uidoc.Document;
View view = doc.ActiveView;
using( Transaction tr = new Transaction( doc ) )
{
Parameter param1 = null;
Parameter param2;
Parameter param3;
Parameter param4 = null;
Parameter param5 = null;
Parameter param6 = null;
Parameter param7 = null;
tr.Start( "synchro" );
// Main.idForSynchros is the collection of data
for( int i = 0; i < Main.idForSynchros.Count; i++ )
{
IdForSynchro i = Main.idForSynchros[ i ];
Element element = doc.GetElement( new ElementId(
Convert.ToInt32( i.RevitId ) ) );
param1 = element.LookupParameter( "PLUGIN_PARAM1" );
param2 = element.LookupParameter( "PLUGIN_PARAM2" );
param3 = element.LookupParameter( "PLUGIN_PARAM3" );
param4 = element.LookupParameter( "PLUGIN_PARAM4" );
param5 = element.LookupParameter( "PLUGIN_PARAM5" );
param6 = element.LookupParameter( "PLUGIN_PARAM6" );
param7 = element.LookupParameter( "PLUGIN_PARAM7" );
if( param1.AsInteger() < 1 )
{
param1.Set( i.param1 );
}
string param6Value = DateTime.Parse(
i.data.param6, CultureInfo.InvariantCulture )
.ToLongDateString();
string param7Value = DateTime.Parse(
i.data.param7, CultureInfo.InvariantCulture )
.ToLongDateString();
Utils.SetParameterValueString( param2, i.data.param2 );
Utils.SetParameterValueString( param3, i.data.param2 );
Utils.SetParameterValueString( param4, i.data.param4 );
Utils.SetParameterValueString( param5, i.data.param5 );
Utils.SetParameterValueString( param6, param6Value );
Utils.SetParameterValueString( param7, param7Value );
}
tr.Commit();
}
}
// Method of Utils class that set Parameter value
public static void SetParameterValueString(
Parameter parameter, string value )
{
if( parameter != null )
{
parameter.Set( value );
}
}
```
Does anyone have any idea to solve this?
\*\*Answer:\*\* `LookupParameter` is unnecessarily costly.
There is no need to re-execute it for every parameter on every single element.
That is costing you a lot of time and hurting performance.
Every shared parameter is equipped with a GUID to identify it.
Determine the GUID once only on the first element.
Then, you can use the GUID to retrieve each parameter directly from the element without any further need for `LookupParameter` to search through the entire list of parameters each time.
Here is my suggestion for an improvement:
```csharp
class IdForSynchro
{
public ElementId RevitId { get; set; }
public int Param1 { get; set; }
public string Param2 { get; set; }
public double Param3 { get; set; }
}
void modifyParameterValues( Document doc, IList data )
{
using( Transaction tr = new Transaction( doc ) )
{
Guid guid1 = Guid.Empty;
Guid guid2 = Guid.Empty;
Guid guid3 = Guid.Empty;
tr.Start( "synchro" );
foreach( IdForSynchro d in data )
{
Element e = doc.GetElement( d.RevitId );
if( Guid.Empty == guid1 )
{
guid1 = e.LookupParameter( "PLUGIN_PARAM1" ).GUID;
guid2 = e.LookupParameter( "PLUGIN_PARAM2" ).GUID;
guid3 = e.LookupParameter( "PLUGIN_PARAM3" ).GUID;
}
e.get_Parameter( guid1 ).Set( d.Param1 );
e.get_Parameter( guid2 ).Set( d.Param2 );
e.get_Parameter( guid3 ).Set( d.Param3 );
}
tr.Commit();
}
}
```
I added this code in the original and updated forms
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).
Here is the [diff between the two](https://github.com/jeremytammik/the_building_coder_samples/commit/fea7381f51b660fc9b5660c2e64f548623b11d8b).
and the [current implementation code](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCreateSharedParams.cs#L547-L584).
Please equip your code with a little benchmark timer stopwatch and let us know whether this helps and how it affects the processing time required before and after making this modification.
\*\*Response:\*\* I applied the solution and the execution time improved.
Here are some results before and after the modification, correlating the number of family instances with the execution time required in milliseconds before and after the enhancement:

| Instances | ms before | ms after |
| --- | --- | --- |
| 58 | 896 | 794 |
| 375 | 1958 | 1549 |
| 1052 | 16777 | 8152 |

But there are still times when changes are not applied to the document.
\*\*Answer:\*\* Not sure if you caught it when you refactored, but in your original code you used `param2` to set `2` and `3`.
Could that be affecting what your results are?
May be good to throw in some Try/Catch block to try and see where or if it's failing or at least a task dialog to tell user if a param is missing.
#### Full-Text Search for The Building Coder Posts
I discovered a nice article by Artem Krylysov
suggesting [let's build a full-text search engine](https://artem.krylysov.com/blog/2020/07/28/lets-build-a-full-text-search-engine)
using the [Go programming language](https://golang.org),
with an accompanying [simplefts GitHub repository](https://github.com/akrylysov/simplefts) sharing the final result.
That prompted me to dabble a bit with Go and use it to implement full text search functionality for The Building coder blog posts as well.
The work in progress is available from my [tbcfts GitHub repository](https://github.com/jeremytammik/tbcfts).
[The Building Coder blog source `tbc`](https://github.com/jeremytammik/tbc) is available from GitHub as well, so you have all you need to play with it yourself, if you please.
Here is a sample run searching for the word 'dabble', which includes today's draft post, still lacking the public URL:

```
/a/src/go/tbcfts $ ./tbcfts -q "dabble"
2020/09/09 10:31:40 Starting tbcfts, p=/a/doc/revit/tbc/git/a, q=dabble
2020/09/09 10:31:41 Loaded 1863 documents in 377.397917ms
2020/09/09 10:31:44 Indexed 1863 documents in 2.876775333s
2020/09/09 10:31:44 Search for 'dabble' found 5 documents in 9.703µs
2020/09/09 10:31:44 582 [Wiki API Help, View Event and Structural Material Type](http://thebuildingcoder.typepad.com/blog/2011/05/wiki-api-help-view-event-and-structural-material-type.html)
2020/09/09 10:31:44 906 [Export Wall Parts Individually to DXF](http://thebuildingcoder.typepad.com/blog/2013/03/export-wall-parts-individually-to-dxf.html)
2020/09/09 10:31:44 961 [Super Insane MP3 and Songbird Playlist Exporter](http://thebuildingcoder.typepad.com/blog/2013/06/super-insane-mp3-and-songbird-playlist-exporter.html)
2020/09/09 10:31:44 1008 [Open MEP Connector Warning](http://thebuildingcoder.typepad.com/blog/2013/08/open-mep-connector-warning.html)
2020/09/09 10:31:44 1863 [Optimising Parameters and Full-Text Search](http thebuildingcoder.typepad.com not yet published)
```

According to `wc`, the current blog post HTML source consists of 355233 lines, 2230690 words and 20676311 characters, including markup.
As you can see, loading the documents and storing their body text in memory costs ca. 400 ms.
The indexing is costly, clocking in at ca. 3 seconds.
Once indexing is complete, the lookup is very fast, consuming just 10 microseconds.
Obviously, the next feature to address would be caching the index.
Another important enhancement would be to split the documents into smaller sections.
For instance, I could create much smaller and more targeted documents to index by using the `h4` tags that delimit individual sections of text within each blog post instead of retaining each blog post in its entirety as a single document.
#### Decimal Point Woe
A few of my Revit sample add-ins have been promoted to full-fledged commercial applications.
One of them is the OBJ importer, which Eric Boehlke has published to the AppStore and continuously enhanced.
A new little issue arose with it that is useful to be aware of, since it applies to many other contexts as well:
\*\*Question:\*\* I had a strange phenomenon happen.
I have an OBJ Import app customer using Czech language on Windows.
His Revit is English 2020.
The problem is that with the same OBJ file, and the same Revit version, on my computer it works fine, and on his the app runs but fails and imports 0 faces.
Have you ever seen an add-in fail because the OS was a language that had non-English characters?
I really don't know what is causing the problem.
\*\*Answer:\*\* Yes, often!
Some language cultures use comma instead of decimal point and then crash when trying to read floating point numbers.
\*\*Response:\*\* Aha!
Yes, I see, e.g., in a different OBJ software,
[OBJ export is broken on locales with comma instead of dot](https://github.com/keenanwoodall/Deform/issues/17).
That was the problem.
I told the client and updated
the [Troubleshooting \*Mesh Import from OBJ Files\* for Revit](https://truevis.com/troubleshoot-revit-mesh-import) with the workaround:
\*\*Region Number Format\*\*
Some regions use commas instead of dots for the decimal place.
That may cause the import to fail.
A workaround is to set Window’s Region to United States before importing the OBJ.
![Windows regional number format](img/windows_regional_number_format.png "Windows regional number format")