---
post_number: "0100"
title: "Utilizing Revit API Resources"
slug: "utilizing_resources"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'parameters', 'revit-api', 'schedules', 'sheets', 'views']
source_file: "0100_utilizing_resources.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0100_utilizing_resources.html"
---

### Utilizing Revit API Resources

#### Happy Birthday to The Building Coder

![Happy Birthday to The Building Coder](img/100_birthday.gif)

Hooray!
This is the hundredth posting to The Building Coder.
I am celebrating by having a wonderful cup of chai and a muesli with my dear friend
[Andreas](http://www.tanzdichganz.ch)
in the middle of a snowstorm way out in the Swiss countryside.
I hope you are happy as well.
I am looking forward to feedback, suggestions, and questions to fuel future topics.
Please be aware that I do not want to do any of your work for you, but I love supporting you in doing it yourself.
Here is a prime example of such a process coming to full fruition.

A couple of weeks ago,
[Dan Levesque](mailto:dan.levesque@stantec.com)
of
[Stantec](http://www.stantec.com)
approached me asking for support in rapidly implementing a complex process for managing elements in a large structural Revit project.
We decide to handle this step by step and on a do-it-yourself basis, and the end result is really pretty spectacular.
Dan made extremely effective use of a number of pre-existing Revit SDK and ADN samples that address similar issues, asking brief questions on how to adapt them for his own use and with minimal further guidance.

For this special issue, I asked Dan to describe his project as a perfect example of making use of the large base of existing sample code to create a complex and professional customer solution. Here is Dan's project description:

#### Company

[Stantec](http://www.stantec.com)
provides professional consulting services in planning, engineering, architecture, interior design, landscape architecture, surveying, environmental sciences, project management, and project economics for infrastructure and facilities projects. Stantec supports public and private sector clients in a diverse range of markets, at every stage, from initial concept and financial feasibility to project completion and beyond. There services are offered through over 10,000 employees operating out of more than 150 locations in North America.

#### Contact

[Dan Levesque](mailto:dan.levesque@stantec.com)
is a Senior Structural Technologist at Stantec in Scarborough, Maine. He has over 13 years of experience in the use of 3D modelling software, programming, network administration and web development. At Stantec Scarborough office, Mr. Levesque is responsible for Revit Structure integration, file exchange, training, programming and software evaluation.

#### Software

Revit Structure 2009 SP2, Add-in Manager, Visual Studio 2008 Express, Navisworks.

#### Main Project

This is an overview of the project requiring additional Revit customization:

- Industrial power steel structure, new & existing phased.
- Composed of 14,000 tons of structural steel.
- Created in Revit Structure 2009.
- Divided into 4 projects.
- Analytical analysis performed with 3rd party software.
- Visualization, collision, and construction sequencing performed with Navisworks.

#### Revit Customization

These are the goals and requirements of the project specific Revit extension through coding and customization:

- Create and apply new shared parameters to specified structural category elements automatically.
- Export new shared parameters and required predefined parameters to a database based on Revit Element ID.
- Export XYZ analytical coordinates from all structural elements to a database.
- Quickly and accurately number all Revit structural elements utilizing the Mark parameter based on z elevation and category.
- Import and return new parameter data values to respective Revit project elements.

#### Process Overview

Here is an overview of the general process steps and some comments:

Our real-time Revit project consists of a total 14,000 elements divided in 4 projects. With the expertise, help and guidance of Jeremy Tammik and SDK examples, I was able to create a Structural\_ID program based on the SDK FireRating example and a Revit\_XYZ\_Export program based upon the RstLinkRevitClient sample.

**1a.**
Structural\_ID Module 1: Apply Parameters – derived from FireRating:

- Create and apply new shared parameter data per category to Framing & Columns: Structural\_ID\_Frame, Structural\_ID\_Column, and Sequence Number. Defined as user entry text fields.
- Revit designer isolates all structural framing per project and enters 'Projno.F'. All framing elements are assigned.
- Revit designer isolates all structural Columns per project and enters 'Projno.C'. All columns elements are assigned.
- Revit designer isolates all structural framing and columns per project and enters 'Sequence no.' per PM assignment. All selected structural elements are assigned.

**1b.**
Structural\_ID Module 2: Export Parameters – derived from FireRating:

- Export Structural\_ID\_Frame and Structural\_ID\_Column shared parameters. As noted before, these parameters hold a value to define the project category identification such as 'Proj-1F' or 'Proj-1C'. From this we can tell its project origin. You could have duplication of the Revit Element ID amongst the 4 separate projects, so using Structural\_IDs and Element ID as dual validation will effectively provide data integrity.
- Export Sequence Number shared parameter. This user entered value is utilized for establishing a grouping of structural elements for construction sequence which is later interpreted in Navisworks and used for sorting purposes.
- Export predefined parameters Element ID, Mark, and Level.
- End result is Database1, a tabulated spreadsheet defining each row by Revit Element ID, Level, Mark, and Structural ID.

**2.** The Revit projects are then analytically checked and the vertical projection property of the horizontal elements are assigned to the correct T.O.S. level to ensure accuracy on Z. Columns are set to Top and Bottom analytically.

**3.** RST\_XYZ\_Export – derived from RstLinkRevitClient:

- RST\_XYZ\_Export, a second individual program exports all the XYZ, Type, Element ID and other various parameters. The resulting xml data is opened, converted and saved as Database2.
- The Structural ID Database1 and RST\_XYZ\_Export Database2 are then combined and transferred per project through Revit Element ID validation to a Master Database.

**4.** A separate analytical model that has been manually kept in sync is exported to Database3 with its beam numbers and XYZ coordinates.

**5.** A VBA Excel program designed by our structural engineers compares analytical Database3 XYZ values and the Master Database XYZ Revit values and determines the match. This results in the analytical beam numbers placed in a row along with the Revit Element ID and related parameters. Engineering Team coordinates loads and connection data as well in Master Database.

**6.** All data in the Master Database is complete with the exception of the Mark field. Project Supervisor will assign a number sequence to the Mark field that is incremented based upon the Z coordinate of the members. This results in lower levels having smaller values and higher levels high numbers. This is a quick way to number 3,600 plus elements per project. Due to the fact that the Mark parameter is utilized, we can also detect duplicates when importing data brought back into Revit or if the Designer enters a number twice.

**7.** From the Master Database file the Revit Element ID and Mark values are validated and transferred back to the first Database1 file.

**8.** Structural\_ID Module 3: Import Parameters – derived from FireRating:

- The Import application is executed to return the Mark number values to each Revit project from each per project updated Database1 file.

**9.** Revit project tags are modified to depict Mark values and represent element numbering in each project.

**10.** The remainder and continuation of the project is a manual input for number coordination. All numbers forward are incremented higher and previous numbers are not to be re-used. Schedules are created based on ascending mark number listing to ensure next number used.

#### Summary

There is obviously a lot going on here, but if kept regimented it works well. Time as always, will dictate how much effort is put in making a program practical, clean, accurate, and deliver the scope intended. Within one weeks time (We stopped counting the hours) we were able to complete this overall team effort. As I look in hind sight, I am grateful that I had the required information and support by Autodesk and its professional staff. Without it, it would have been a grave disappointment. I am now aware of my available resources to further our development and utilization of the Revit API. Its amazing how much you can learn from the various samples provided in the SDK with supported documentation. Just imagine the possibilities for the upcoming future. I intend to bring this program example to a more consolidated level and transform it to an internal Revit Auto Mark Numbering validation program with various user select export options ranging from shared parameters, predefined parameters, and XYZ coordinates.

Thanks to Jeremy Tammik
of
[Autodesk](http://www.autodesk.com)
and
Elizabeth Shulok of
[Structural Integrators](http://www.structuralintegrators.com)
for all your help and sharing the knowledge.

#### Back to Jeremy

Thank you very much Dan, for this contribution.
I would like to express my appreciation and respect for your hard work and enthusiasm, and gratitude for documenting and sharing the end result, hopefully as an inspiration for others as well.
It was an honour and a pleasure to work with Dan and see with him make such efficient use of every little snippet of information provided.