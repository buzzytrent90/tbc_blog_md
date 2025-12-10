---
post_number: "0272"
title: "UK Electrical Schedule Sample"
slug: "uk_elec_panel_schedule"
author: "Jeremy Tammik"
tags: ['csharp', 'parameters', 'references', 'revit-api', 'schedules', 'sheets']
source_file: "0272_uk_elec_panel_schedule.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0272_uk_elec_panel_schedule.html"
---

### UK Electrical Schedule Sample

Here is my final Christmas present and last post of the year.
Over the next few days, I will do my best not to answer any email or comments  :-)

This final gift before I leave is from Simon Jones, Technical Sales Manager of Autodesk UK.
Simon has been strongly and enthusiastically involved in all things AEC related for several decades now and was maybe the main early driving force of the use of AutoCAD in the architectural realm in the UK.
Even though his job is mainly management nowadays, he still likes to keep up with the API side of things.

The automatic numbering system and schedules for electrical panels built into Revit MEP are designed according the conventions used in the USA, and do not fit some other countries conventions at all.
Simon now presents an almost complete application for generating a UK style panel schedule from Revit MEP.
It defines the following two commands:

- Add Circuit References to assign UK style circuit references to an existing model as described by Simon below.- Panel Schedule Export to export the circuits to an Excel spreadsheet.

To demonstrate the functionality, we start off with a sample model with two panels:

![Sample model](img/uk_panels_1.png)

Running the first command prompts us to select which panel to process.
Selecting Panel 1 updates the model:

![Circuit references added](img/uk_panels_2.png)

Running the second command prompts us to select a panel again, and a spreadsheet template file to populate.
Here is the resulting output spreadsheet for Panel 1 in this model using the sample template file provided with Simon's code:

![Panel schedule spreadsheet](img/uk_panels_3.png)

Here is Simon's description of the application:

#### Revit MEP UK DB Schedule Sample

*By Simon Jones, Autodesk UK December 2009*

This is a far as I got for generating a UK Style Panel Schedule from Revit MEP.

The first problem to get over is to match the RME circuit numbers with the conventions. Consider the simple panel below that has 18 slots. In Revit MEP the odd number circuits (1,3,5,) are listed on the left side of the panel, whilst the even number circuits (2,4,6,) are listed on the right. In the UK the convention is to use circuit codes that group in threes the match the phasing, and are typically named: 1L1, 1L2, 1L3, 2L1, 2L2, 2L3, 3L1... and a 3 phase circuit example would be 6L123 for circuits 14, 16 & 18, as illustrated below:

| Left | | Right | |
| --- | --- | --- | --- |
| Num | Code | Num | Code |
| 1 | 1L1 | 2 | 4L1 |
| 3 | 1L2 | 4 | 4L2 |
| 5 | 1L3 | 6 | 4L3 |
| 7 | 2L1 | 8 | 5L1 |
| 9 | 2L2 | 10 | 5L2 |
| 11 | 2L3 | 12 | 5L3 |
| 13 | 3L1 | 14 | 6L123 |
| 15 | 3L2 | 16 |
| 17 | 3L3 | 18 |

#### Commands

The C#.NET sample comes with two commands:

- Add Circuit References – assigns UK style circuit references.- Panel Schedule Export – exports the circuits to an Excel spreadsheet.

#### Add Circuit References

To provide the circuit number to code mapping, this sample sets the value of the 'Circuit Ref' shared parameter that has been added to circuits and light fittings. Ideally, the 'Circuit Ref' parameter should be added to all objects with an electrical connector as well as wires, so that you can tag what circuit they are on.

After running the command you will simply be asked to select the appropriate panel.

If youre using the supplied dataset, this shared parameter has already been added to circuits and light fittings through the Project Parameters, but if you use an alternative Revit project, youll have to add this parameter manually.

For this to work, the panel needs to have an even number of circuits that is also divisible by 3, for example 6, 12, 18, 24, 48 etc.

If any changes are made to the number of circuits or their position in the panel, this command will need to be re-run to re-assign the codes.

#### Panel Schedule Export

Once the circuit references have been set-up, you can then use the Panel Schedule Export command to extract the data to an Excel spreadsheet. As with the Add Circuit References command, you will be prompted to select the appropriate panel.

After picking the panel, you will then be prompted to select an Excel spreadsheet.
This sample is hard coded to use the cell positions in the PanelSchedule.xls that is included with the sample. For example, the following cell positions are used:

- O3 – Panel Name- H5 – Number of ways (slots)- A12, A13, A14 etc. – Circuit Reference- B12, B13, B14 etc. – Circuit description- J12, J13, J14 etc. – Number of points (devices)- K12, K13, K14 etc. – Cable Size PH (saved as shared parameter 'CableSize\_PH')- L12, L13, L14 etc. – Cable Size CPC (saved as shared parameter 'CableSize\_CPC')- Q12, Q13, Q14 etc. – Circuit rating

This could be improved by adding tags that are searched for and replaced, but if you want to change the current hardcoded method, look for calls to exportToExcel in CreateDbSchedule.cs, for example:

```
exportToExcel(ws, "O", 3, panelName);
```

Here is the complete
[UK\_DB\_Schedule](zip/UK_DB_Schedule.zip) source code, Visual Studio solution, spreadsheet template, shared parameter file, and documentation.

Very many thanks to Simon for this great and useful real-world example!

#### Merry Christmas and Happy New Year!

So, finally, I wish you all a Very Merry Christmas and a Happy New Year!

I am celebrating with myself and my kids for a couple of days and then heading off for a total break, the most exciting part of my upcoming holidays, a long and deep voyage to an exotic place in exotic company: into my own self with my own self.

Straight after Christmas I am participating in a
[Vipassana](http://en.wikipedia.org/wiki/Vipassan%C4%81) meditation retreat, cf.
[www.dhamma.org](http://www.dhamma.org).

Apparently, this will mean not speaking with and not even looking at another person, getting up at four in the morning, not eating anything at all after midday, for ten days.
It has nothing at all to do with religion or esoteric beliefs, it is a purely practical matter of getting into a state of calm and peace which allows me to really observe myself and what is going on in me, explore the interrelationship between my thoughts and emotions and body in unprecedented depth.

And forget about the Revit API blogging for a while  :-)
I will be back in the second week of January.
I wish you all the best until then.