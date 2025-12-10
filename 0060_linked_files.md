---
post_number: "0060"
title: "Linked Files"
slug: "linked_files"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'revit-api', 'rooms', 'views']
source_file: "0060_linked_files.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0060_linked_files.html"
---

### Linked Files

Here is pretty thorough exploration by Joe Ye on how to retrieve linked Revit files in a project.

The built-in category of a linked Revit file is BuiltInCategory.OST\_RvtLinks, and the class storing it in the Revit database is Instance.
Therefore, we can retrieve all linked Revit files in a document like this:

```csharp
Autodesk.Revit.Creation.Filter cf
  = app.Create.Filter;

BuiltInCategory bic
  = BuiltInCategory.OST\_RvtLinks;

Filter f1
  = cf.NewCategoryFilter( bic );

Filter f2
  = cf.NewTypeFilter( typeof( Instance ) );

Filter f3
  = cf.NewLogicAndFilter( f1, f2 );

List<Element> links = new List<Element>();

Document doc = app.ActiveDocument;
doc.get\_Elements( f3, links );
```

We simply set up filters for the category and class, which is also the System.Type, combine them in a Boolean expression, and ask the document for the elements matching this filter.

Revit 2008 did not provide a valid category property for the linked model, and at that time the API did not support filtering for specific elements, so one had to use a slower and more complex approach such as this:

```csharp
BuiltInParameter bip
  = BuiltInParameter.ELEM\_TYPE\_PARAM;

Document doc = app.ActiveDocument;
ElementIterator it = doc.Elements;

List<Element> links = new List<Element>();

while( it.MoveNext() )
{
  Instance inst = it.Current as Instance;
  if( null != inst )
  {
    Parameter p = inst.get\_Parameter( bip );
    ElementId id = p.AsElementId();
    Element e = doc.get\_Element( ref id );
    string n = e.Name;
    string s = n.Substring( n.Length - 4 );
    if( s.ToLower().Equals( ".rvt" ) )
    {
      links.Add( inst );
    }
  }
}
```

In this loop we check all elements whose class is Instance.
On those, we check for the built-in parameter ELEM\_TYPE\_PARAM, which gives us the family.
The parameter is an element id.
With the id, we can ask the document to retrieve the element 'e' and ask it for its name 'n'.
The linked file's element name is the file name, which ends in ".rvt".

#### Linked File Path

This quickly leads to a new question: Having the file name, is it possible to also get the file path for a given Revit link?

For that, we can make use of the fact that Application.Documents manages all imported Revit files.
The linked Revit files' full path can be retrieved from the Document.PathName property.
Here is some code to set up a dictionary mapping each Revit link's file name to its full path:

```csharp
DocumentSet docs = app.Documents;
int n = docs.Size;

Dictionary<string, string> dict
  = new Dictionary<string, string>( n );

foreach( Document doc in docs )
{
  string path = doc.PathName;
  int i = path.LastIndexOf( "\\" ) + 1;
  string name = path.Substring( i );
  dict.Add( name, path );
}
```

Unfortunately, the question still has some thorny bits in it, because it remains unclear how to get the linked files in a Revit project when there are two open projects, both referencing a files with the
same name are but at different paths. For example:

- Open a Revit project: C:\tmp\a\link.rvt with a link to C:\tmp\a\x.rvt.
- Open another Revit project in the same Revit session: C:\tmp\b\x.rvt.

Now we want the Revit links for the project "link.rvt".
From the elements collection we can find the name of the linked file "x.rvt".
But there are two files with the same name in the documents collection and we don't know which one of them to choose.
The files in the documents collection are:

- C:\tmp\a\link.rvt
- C:\tmp\a\x.rvt
- C:\tmp\b\x.rvt

For this situation, you can distinguish between the opened Revit document and the imported one.
When you retrieve a document from Application.Documents and it is currently open in Revit, its property Document.ActiveView is not null.
If the document is imported, its Document.ActiveView is null.
This can be used to tell which one is imported.

Unfortunately, this still does not solve the problem if two opened documents import two Revit files with the same file name in different folders.

Here is the mainline of an external command making use of these methods:

```csharp
Application app = commandData.Application;

Dictionary<string, string> dict
  = GetFilePaths( app );

List<Element> links
  = GetLinkedFiles( app );

int n = links.Count;
Debug.WriteLine( string.Format(
  "There {0} {1} linked Revit model{2}.",
  (1 == n ? "is" : "are"), n,
  Util.PluralSuffix( n ) ) );

string name;
char[] sep = new char[] { ':' };
string[] a;

foreach( Element link in links )
{
  name = link.Name;
  a = name.Split( sep );
  name = a[0].Trim();

  Debug.WriteLine( string.Format(
    "Link '{0}' full path is '{1}'.",
    name, dict[name] ) );
}
return CmdResult.Succeeded;
```

It makes use of the fact that the link instance element names use " : " to delimit the file name, as in:

```
"two_rooms.rvt : 2 : location "
```

Here is the output from running this command in a simple sample file:

```
There are 2 linked Revit models.
Link 'roof.rvt' full path is 'C:\tmp\roof.rvt'.
Link 'two_rooms.rvt' full path is 'C:\tmp\two_rooms.rvt'.
```

Joe suggests that yet another situation also needs to be considered. Assuming the following situation:

- C:\temp\a\file1.rvt is opened.
- C:\temp\a\main.rvt is also opened.
- C:\temp\b\file1.rvt is imported into main.rvt.

In this case, the GetFilePaths method retrieves two file paths:

1. C:\temp\a\file1.rvt
2. C:\temp\b\file1.rvt

From the link name file1.rvt alone, we cannot tell which path is the imported one.
To do so, we need to remove the non-imported files from the mapping.
In this case, GetFilePaths should remove the opened Revit file and only contain non-opened ones, which represent the imported files.
This can be achieved by using the ActiveView property, for instance by enhancing GetFilePaths like this:

```csharp
Dictionary<string,string> GetFilePaths(
  Application app,
  bool onlyImportedFiles )
{
  DocumentSet docs = app.Documents;
  int n = docs.Size;

  Dictionary<string, string> dict
    = new Dictionary<string, string>( n );

  foreach( Document doc in docs )
  {
    if( !onlyImportedFiles
      || ( null == doc.ActiveView ) )
    {
      string path = doc.PathName;
      int i = path.LastIndexOf( "\\" ) + 1;
      string name = path.Substring( i );
      dict.Add( name, path );
    }
  }
  return dict;
}
```

I hope that this discussion is useful for exploring situations requiring handling of linked files in Revit.
Many thanks to Joe for his thorough exploration!

Here is
[version 1.0.0.17](http://thebuildingcoder.typepad.com/blog/files/bc10017.zip)
of the complete Visual Studio solution with the new command CmdLinkedFiles discussed here.