---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.5
content_type: code_example
optimization_date: '2025-12-11T11:44:13.372921'
original_url: https://thebuildingcoder.typepad.com/blog/0106_list_linked_elements.html
post_number: '0106'
reading_time_minutes: 5
series: elements
slug: list_linked_elements
source_file: 0106_list_linked_elements.htm
tags:
- csharp
- elements
- family
- filtering
- python
- revit-api
- selection
- views
title: List Linked Elements
word_count: 918
---

### List Linked Elements

We already had a look at the issue of
[linked files](http://thebuildingcoder.typepad.com/blog/2008/12/linked-files.html)
and
[how to hide them](http://thebuildingcoder.typepad.com/blog/2009/01/hiding-linked-files.html).
A frequent question in this context is how to access the elements in linked files.
This can be very simple, actually, as we will demonstrate.
I had a discussion on this with Joel Karr of
[Environmental Systems Design, Inc](http://www.esdesign.com).
He is implementing a command to order to monitor and compare the lighting fixtures in an MEP model with the ones defined in a linked in architectural model.
He very kindly provided a sample application listing the lighting fixture elements contained in a linked file, as a starting point for implementing a comparison of those with the ones in the current model.
I rewrote and significantly simplified his application and find that we can achieve a lot of functionality with minimal effort.

For this purpose we implemented a new external command CmdLinkedFileElements which iterates over all open documents, which includes the linked ones as well, and displays selected properties for all lighting fixture elements in a data grid.

#### Displaying element properties

We define a class ElementData to manage the properties we wish to display:

- Document, the project containing the element.
- Element, the element name.
- Id, the element id.
- X, Y and Z, the family instance location point coordinates.
- UniqueId, the unique id.
- Folder, the document folder.

These data items are stored in individual private members.
We define a constructor in order to initialise all the members for a given Revit element.
By defining public properties for accessing each data item to display, we can save the effort of transferring data manually into the form.
Here is the class implementation including member data, constructor, and accessors:

```python
public class ElementData
{
  string \_document;
  string \_elementName;
  int \_id;
  double \_x;
  double \_y;
  double \_z;
  string \_uniqueId;
  string \_folder;

  public ElementData(
    string path,
    string elementName,
    int id,
    double x,
    double y,
    double z,
    string uniqueId )
  {
    int i = path.LastIndexOf( "\\" );
    \_document = path.Substring( i + 1 );
    \_elementName = elementName;
    \_id = id;
    \_x = x;
    \_y = y;
    \_z = z;
    \_uniqueId = uniqueId;
    \_folder = path.Substring( 0, i );
  }

  public string Document {
    get { return \_document; }
  }
  public string Element {
    get { return \_elementName; }
  }
  public int Id {
    get { return \_id; }
  }
  public string X {
    get { return Util.RealString( \_x ); }
  }
  public string Y {
    get { return Util.RealString( \_y ); }
  }
  public string Z {
    get { return Util.RealString( \_z ); }
  }
  public string UniqueId {
    get { return \_uniqueId; }
  }
  public string Folder {
    get { return \_folder; }
  }
}
```

With these properties defined, displaying the data in the form is handled completely automatically by one single line in the form constructor:

```csharp
public CmdLinkedFileElementsForm(
  List<ElementData> a )
{
  InitializeComponent();
  dataGridView1.DataSource = a;
}
```

#### Collecting element data from linked files

We have discussed how to display the element data in a data grid.
Before we can display it, we need to retrieve it from the Revit database.
To do so, we can simply iterate over all the open documents in the application.
If desired, we can implement a filter to eliminate the documents that do not represent a linked file.

We implement a utility method to filter for elements matching a given category and type.

```csharp
public List<Element> GetElements(
  BuiltInCategory bic,
  Type elemType,
  Application app,
  Document doc )
{
  CreationFilter cf = app.Create.Filter;
  Filter f1 = cf.NewCategoryFilter( bic );
  Filter f2 = cf.NewTypeFilter( elemType );
  Filter f3 = cf.NewLogicAndFilter( f1, f2 );
  List<Element> elements = new List<Element>();
  doc.get\_Elements( f3, elements );
  return elements;
}
```

This can be used both to extract the lighting fixtures to display in the form as well as the linked files instances, if we need those.
We will only use it for the lighting fixtures in the real code.
For completeness sake, this would be the call to retrieve the link instances:

```csharp
  List<Element> links = GetElements(
    BuiltInCategory.OST\_RvtLinks,
    typeof( Instance ), app, doc );
```

The external command performs the following steps:

- Iterate over the application documents.
- Select all lighting fixtures in each linked document.
- Instantiate and populate an ElementData instance for each fixture.
- Display the form with the data collected.

Here is the mainline for the command:

```csharp
List<ElementData> data = new List<ElementData>();
Application app = commandData.Application;
DocumentSet docs = app.Documents;
foreach( Document doc in docs )
{
  List<Element> elements = GetElements(
    BuiltInCategory.OST\_LightingFixtures,
    typeof( FamilyInstance ), app, doc );

  foreach( FamilyInstance e in elements )
  {
    string name = e.Name;
    LocationPoint lp = e.Location as LocationPoint;
    if( null != lp )
    {
      XYZ p = lp.Point;
      data.Add( new ElementData( doc.PathName, e.Name,
        e.Id.Value, p.X, p.Y, p.Z, e.UniqueId ) );
    }
  }
}
using( CmdLinkedFileElementsForm dlg = new CmdLinkedFileElementsForm( data ) )
{
  dlg.ShowDialog();
}
return CmdResult.Cancelled;
```

Notice how short and sweet this is?

Here is an example of the command displaying the data in the sample model provided by Joel:

![Linked file element data](img/LinkedFileElements.png)

This can obviously easily be adapted to handle other element types or more general selections than just lighting fixtures.
For the moment, of course, we are ignoring all the thorny issues to do with transformations and stuff.
I am looking forward to your comments on this one.

Here is
[version 1.0.0.27](http://thebuildingcoder.typepad.com/blog/files/bc10027.zip)
of the complete Visual Studio solution with the new CmdLinkedFileElements command.
It also includes the new command CmdNewRailing which unfortunately does **not** create a new railing instance.
The reasons for this can be found in the
[discussion with Berria](http://thebuildingcoder.typepad.com/blog/2009/02/list-railing-types.html#comments).