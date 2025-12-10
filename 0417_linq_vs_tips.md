---
post_number: "0417"
title: "Linq Methods and Visual Studio Tips"
slug: "linq_vs_tips"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'python', 'references', 'revit-api', 'selection', 'transactions']
source_file: "0417_linq_vs_tips.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0417_linq_vs_tips.html"
---

### Linq Methods and Visual Studio Tips

Yesterday I completed the Revit API training in Munich, and here are some more interesting tips and tricks that I learned there besides the one I wrote about last on how to set up
[debugging using Visual Studio 2010 Express](http://thebuildingcoder.typepad.com/blog/2010/07/debugging-in-visual-studio-2010-express.html):

- Using Intellisense to
  [subscribe to an event](#1) and generate the
  [event handler stub implementation](#1).- Using Intellisense to
    [generate a using statement with Ctrl + '.'](#2).- Linq provides a
      [Contains method for an array](#3).

#### Automatically Subscribing to an Event and Generating the Event Handler Code

Before describing the new tricks that I learned, let me explain something that I actually knew before and have made quite a lot of use of in my own coding as well as previous trainings: how to use Intellisense to set up a Revit event handler with a minimum of keystrokes and potential errors.

As an example, I will subscribe to the Idling event and implement the event handler stub using exactly seven keystrokes.

Here is some minimal initial code of a newly created external command to use as a starting point:
```csharp
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

namespace RevitAddin
{
  [Transaction( TransactionMode.Automatic )]
  [Regeneration( RegenerationOption.Manual )]
  public class Commands : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIApplication a = commandData.Application;

      return Result.Succeeded;
    }
  }
}
```

At this point, I wish to subscribe to the UIApplication Idling event.
I type 'a' followed by '.' and scroll to the Idling entry suggested by the Visual Studio Intellisense:

![Visual Studio Intellisense lists UIApplication members](img/vs_tips_01.png)

After selecting that by pressing <Tab>, I type '+=' to subscribe to the event, and Intellisense automatically suggest the appropriate handler to attach:

![Visual Studio Intellisense suggests pressing <Tab> to create code to attach event handler](img/vs_tips_02.png)

At this point, I can simply press <Tab> to add the code to subscribe to the event.
However, I have not yet implemented the handler for it.
That is perfectly appropriate, since Intellisense can it for me automatically.
First, here is the result of pressing <Tab> once, with some code generated and a tooltip sugesting that I press <Tab> again to create an event handler skeleton method:

![Visual Studio Intellisense suggests pressing <Tab> again to create event handler stub implementation](img/vs_tips_03.png)

The final result of this operation with some line breaks added to avoid overly long lines looks like this:
```csharp
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

namespace RevitAddin
{
  [Transaction( TransactionMode.Automatic )]
  [Regeneration( RegenerationOption.Manual )]
  public class Commands : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIApplication a = commandData.Application;

      a.Idling
        += new System.EventHandler<
          Autodesk.Revit.UI.Events.IdlingEventArgs>(
            a\_Idling );

      return Result.Succeeded;
    }

    void a\_Idling(
      object sender,
      Autodesk.Revit.UI.Events.IdlingEventArgs e )
    {
      throw new System.NotImplementedException();
    }
  }
}
```

Here are the seven keystrokes I made to subscribe to the event and add the handler stub method: 'a', '.', <Tab>, '+', '=', <Tab>, <Tab>.

#### Generating a using Statement with Ctrl + '.'

Here is a tip from Alexander Buschmann of
[IDAT Gmbh](http://www.idat.de) to
automatically generate using statements for namespaces that we would like to use.

Previously, when I created a new Revit project, I manually added the references to the Revit API assemblies RevitAPI.dll and RevitAPIUI.dll.

Part of that manual and error prone labour can be easily avoided making use of Augusto's
[Revit add-in templates](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html).
Even so, when I was still doing some typing work to add using statements for additional namespaces to the project module.
For instance, in the situation described above, I would normally want to make the automatically generated event handling code more succinct and readable by removing the explicit namespace prefix 'Autodesk.Revit.UI.Events.' from IdlingEventArgs and adding a 'using' statement for that namespace at the head of the module instead.
That would involve some copy and paste work, jumping to the top of the file, editing, and navigating back to where I started from.

Alexander now pointed out that if you place the cursor over type the name of a class in the Visual Studio IDE and hit Ctrl + '.', Visual Studio will automatically scan all the project references to determine the namespace providing the class definition and offer to either add the namespace prefix to fully qualify the class name, or add the appropriate using statement for it to the head of the file.

One of the advantages of the latter is that there is no longer any need to jump to the top of the module, type 'using' and add the namespace, and then navigate back to where you came from.

A very neat trick indeed that I was immediately able to use repeatedly during the rest of the training, once Alexander had pointed it out to me.

I can make use of that functionality in the situation above like this:

- Delete the namespace prefix from the event handler IdlingEventArgs argument.
  Note that the class name colour changes from teal to black, since it is now undefined, lacking the namespace prefix and the required using statement.- Place the cursor over the now undefined IdlingEventArgs class name.- Press Ctrl + '.'.

At this point, the Visual Studio IDE displays the following Intellisense menu options prompting me to choose whether to fully qualify the class name or add the appropriate using statement for it:

![Visual Studio Intellisense suggests adding a namespace prefix or a using statement](img/vs_tips_04.png)

Pressing <Tab> at this point adds the using statement at the head of the module with no need to jump there and navigate back again.

#### Linq Defines Array.Contains

At a certain point in the sample code that we were developing together, I wanted to check whether the category of a selected element was contained in a specific list.
The use case was the implementation of a Revit API selection filter which restricts the interactive selection process to certain structural elements.

To check this, I implemented a list of integer values from the built-in categories I was interested in and instantiated a List<int> from them in order to make use of its Contains method.
Here is the entire implementation of the selection filter using this approach:
```python
/// <summary>
/// Selection filter allowing only structural elements.
/// </summary>
class StructuralSelectionFilter : ISelectionFilter
{
  static int[] \_categories = new int[] {
    (int) BuiltInCategory.OST\_StructuralColumns,
    (int) BuiltInCategory.OST\_StructuralFoundation,
    (int) BuiltInCategory.OST\_StructuralFraming
  };

  static List<int> \_list
    = new List<int>( \_categories );

  public bool AllowElement( Element e )
  {
    return null != e.Category
      && \_list.Contains(
        e.Category.Id.IntegerValue );
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    return true;
  }
}
```

In this situation, Alexander provided another useful tip by pointing out that the Linq namespace provides a Contains extension method directly for the native array type.
By adding a 'using System.Linq' statement, we can eliminate the intermediate list helper and use Contains directly on the array instead:
```python
class StructuralSelectionFilter : ISelectionFilter
{
  static int[] \_categories = new int[] {
    (int) BuiltInCategory.OST\_StructuralColumns,
    (int) BuiltInCategory.OST\_StructuralFoundation,
    (int) BuiltInCategory.OST\_StructuralFraming
  };

  public bool AllowElement( Element e )
  {
    return null != e.Category
      && \_categories.Contains(
        e.Category.Id.IntegerValue );
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    return true;
  }
}
```

Many thanks to Alexander for these useful tips!