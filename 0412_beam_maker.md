---
post_number: "0412"
title: "Beam Maker Using a Void Extrusion to Cut"
slug: "beam_maker"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'revit-api', 'transactions']
source_file: "0412_beam_maker.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0412_beam_maker.html"
---

### Beam Maker Using a Void Extrusion to Cut

Here is an interesting and more than average advanced sample application
[BeamMaker](zip/BeamMaker.zip) by
Marcelo Quevedo of
[hsbSOFT](http://www.hsb-cad.com) demonstrating
how to create a void in a beam in a family file.

The family file is created and loaded into the current working project on the fly.

The final result of the application is the following beam family instance created in the currently active document, which in this case was a new project:

![Beam with a void extrusion cut out](img/beammaker.png)

Here is Marcelo's description of his project:

Using the default template that comes with Revit, the sample creates a 'structural framing' family that contains a void extrusion that cuts the solid (existing extrusion in the Revit template).

- The sample looks for the template, opens it, and creates a void extrusion in the position (0,0,0). It means that it is located in the center of the solid.- In order that the void extrusion can cut the solid, I am using the brilliant solution that Joe recommended me based on the CombineElements method.- After that, it loads the new family into an existing Revit project via the LoadFamily method.- Finally, using this new family, it creates a new instance in the existing project with the NewFamilyInstance method.

Here are some quick and dirty notes of my own from stepping through the application code in the debugger:

- Check if the current document is a family document.- If not, a new family document based on the 'Structural Framing - Beams and Braces' family template file is created. The template file already contains one extrusion representing the beam element solid body.- Start a transaction.- Create a void extrusion by calling the NewExtrusion method with its first argument, bool isSolid, set to false to create a void. This requires the prior creation of the appropriate profile curve array array and sketch plane.- Collect all extrusions in the family document, e.g. the solid beam element and the void cut-out extrusion.- Apply the CombineElements method to them to cut the void part out of the solid part.- Load the newly created family into the current project.- Commit the transaction.- Insert a new family instance using the new family definition.

BeamMaker includes a whole bunch of useful utility functions, a real treasure trove of gems is contained in there.

As a very simple example, here is the GetFamilyTemplatePath which determines the full path name of a given family template file and makes use of the FamilyTemplatePath property on the Revit application object to do so.
It returns the full path of a default template for a given family template name:
```csharp
private string GetFamilyTemplatePath(
  string strFamilyTemplateName )
{
  string strPathForTemplates
    = \_app.FamilyTemplatePath;

  string strTemplateFile
    = Path.Combine( strPathForTemplates,
      strFamilyTemplateName );

  if( !File.Exists( strTemplateFile ) )
  {
    strTemplateFile = Path.Combine(
      strPathForTemplates,
      "Metric " + strFamilyTemplateName );
  }

  if( !File.Exists( strTemplateFile ) )
  {
    strTemplateFile = "";
  }

  return strTemplateFile;
}
```

Please download the
[BeamMaker project](zip/BeamMaker.zip) including
the source code, Visual Studio solution and add-in manifest files to look at the complete code.

To wrap this up, here is the original discussion between Marcelo and Joe which prompted Marcelo to develop and provide this sample:

**Question:** Using the family API, I am creating a new structural framing family.
I am using the template that comes with Revit "Metric Structural Framing - Beams and Braces.rft".
I am creating a void extrusion to cut the existing extrusion, but it does not cut the existing one.
If you manually edit the family, you can see the existing extrusion and the new void extrusion, but the void extrusion does not subtract the geometry from the other one.
How can I achieve this?

**Answer:** Simply creating the void solid does not automatically cause it to cut any existing ones.
In the Revit user interface, one needs to explicitly invoke the Cut Geometry command for this to happen, i.e. to make the void cut the solid.
It is accessible through the Modify tab and Geometry panel.
In a Revit plug-in, one can use the CombineElements method to make the void objects cut the solid objects.

Many thanks to Joe and above all to Marcelo for providing this useful example and code!