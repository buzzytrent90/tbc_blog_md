---
post_number: "0072"
title: "Hello F# via C#"
slug: "hello_f_sharp_via_c_sharp"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'revit-api', 'windows']
source_file: "0072_hello_f_sharp_via_c_sharp.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0072_hello_f_sharp_via_c_sharp.html"
---

### Hello F# via C#

Still in Barcelona, after a very interesting day spent with Alex Vila Ortega, who gave us on a tour of the
[Sagrada Familia](http://en.wikipedia.org/wiki/Sagrada_Fam%C3%ADlia)
cathedral by the famous Catalan architect
[Antoni Gaudi](http://en.wikipedia.org/wiki/Antoni_Gaud%C3%AD)
to discuss the interesting mathematical and geometrical aspects of his architecture, and what we can do together to implement some effective support for them in Revit.
I plan to write more on that as soon as possible, but I have something more urgent to discuss first.

Kean's F# programming contest has a deadline set for the end of February, so the sooner we have access to an F# implementation of a Revit external command, the better.
I will discuss my first steps in that direction, even though the result I have to show for it is not yet perfect. At least it does provide access to the Revit API within an application written in F#.

Since I have not yet been able to execute an external command written in F# directly from Revit, I opted for an intermediate solution with an external command written in C# instead, which calls into the F# implementation.

This is what it looks like so far; the C# external command implementation is completely standard and simply passes its arguments on the F# method:

```csharp
namespace CallFs
{
  public class Command : IExternalCommand
  {
    public IExternalCommand.Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      Module1.BuildingCoder.Command6
        c = new Module1.BuildingCoder.Command6();

      return c.Execute(
        commandData, ref message, elements );
    }
  }
}
```

The current F# class implementation looks like this:

```csharp
#light
open System.Windows.Forms
open Autodesk.Revit
module BuildingCoder =
let show\_message( msg : string ) =
MessageBox.Show( msg, "Building Coder F# Test" ) |> ignore
type public Command6() =
interface IExternalCommand with
member x.Execute( commandData, message, elements ) =
show\_message "Kilroy was here"
IExternalCommand.Result.Succeeded
member public x.Execute( commandData, message : string byref, elements ) =
(x :> IExternalCommand).Execute( commandData, ref message, elements )
```

The reason for the slightly convoluted syntax is that the direct method implementation causes F# to decorate the Execute method name with an explicit interface prefix, so the result is named 'Autodesk-Revit-IExternalCommand-Execute' and thus will not match the expected method name called by the Revit API. This is explained in more detail in a discussion on
[implementing interfaces in F#](http://bugsquash.blogspot.com/2009/01/implementing-interfaces-in-f.html)
which Kean was kind enough to hunt down and point out to me.

Another confusing detail is that
[Reflector](http://thebuildingcoder.typepad.com/blog/2008/10/converting-between-vb-and-c-and-net-decompilation.html)
is reporting the fully qualified method name as 'Module1+BuildingCoder+Command6', whereas the Visual Studio object browser displays it as 'Module1.BuildingCoder.Command6'.
I can well imagine that the former might be confusing for the Revit API loader.
Unfortunately, it does not report any errors if something is wrong, but just quits.

#### Facade and Art

Anyway, I seem to have adapted well to the Spanish eating habits, did not even set foot in the restaurant tonight until about half past nine.
The food is slightly too haute cuisine for my taste, I prefer it simple and natural and tasty with no frills, no facade.
Funny thing, how so much of Gaudi's work deals with the facade, but is so true and deep it moves me.
Normally, I am rather anti-facade, no haute cuisine, no make-up, no frills, just the real thing, but for me, Gaudi created art, real thing, not just facade.
Maybe art is the point where facade becomes the real thing, somehow.
I really liked Alex' very apt categorisation of the various aspects of Gaudi's architecture into bones, muscles and skin.
The bones are like the stick elements in the structural analytical model, the muscles are their profiles, providing strength, and the skin is the facade, which uses some impressively effective yet simple combination of geometry and mathematics to create natural and organic looking and yet relatively easily constructible surfaces, almost none of which are ever planar.

I am still working at getting the direct and full-fledged F# connection up and running and will post a complete solution as soon as that works.