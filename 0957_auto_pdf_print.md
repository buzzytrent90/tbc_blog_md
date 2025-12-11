---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.987587'
original_url: https://thebuildingcoder.typepad.com/blog/0957_auto_pdf_print.html
post_number: 0957
reading_time_minutes: 5
series: general
slug: auto_pdf_print
source_file: 0957_auto_pdf_print.htm
tags:
- references
- revit-api
- schedules
- sheets
- views
- windows
title: Auto PDF Print from Revit 2014
word_count: 1045
---

﻿

### Auto PDF Print from Revit 2014

Below, we will take a look at a Revit multi-project batch PDF printer project driven by the Windows task scheduler and the Revit Idling event.

Before getting to that, though, let me share some of my Sunday activities.

#### Contact Impro Jam and Tech Summit Arrival

From the
[Fells](http://en.wikipedia.org/wiki/Middlesex_Fells_Reservation), I
continued along the Mystic Valley Parkway and stopped to enjoy the beautiful
[Mystic Lakes](http://en.wikipedia.org/wiki/Mystic_Lakes_%28Boston%29) on
my way to the weekly Sunday
[contact improvisation jam](http://contactimprovboston.com/sunday_jam) in the
[Arlington Center](http://www.arlingtoncenter.org),
followed by a magnificent lunch at
[Sabzi Persian restaurant](http://www.sabzikabab.com) and
wonderful conversation with Mike Klinger of
[Anzu Global](http://www.anzuglobal.com),
who actually used to work with Revit at Autodesk, as it turns out.
Small world.

In the evening I checked in to the Tech Summit hotel on the Charles River in Cambridge and went for a very nice dinner with Adam, Manu and Mikako to the Spanish
[Tapeo](http://www.tapeo.com) restaurant on Newbury Street in Boston.
That was the first time I set foot in the city since my arrival.

As Mikako pointed out, it is quite a novelty for me to be pleased with restaurant food – I normally prefer to cook myself – and on Sunday I had two good experiences in a row! Yay!

Back to the Revit API.

#### Batch Processing RVT Files Overnight

This is a contribution from Dan Tartaglia of
[design technology@NBBJ](http://www.nbbj.com).

It started off from the following query:

**Question:** I created a tool that uses a UIControlledApplication Idling event when Revit launches.
It works perfectly correctly in Revit Architecture 2013.
However, if I use the same code on the same computer in Revit Architecture 2012, the Idling event does not fire.

**Answer:** In Revit 2012, the Idling event is not fired until a document is opened.

**Response:** Is it possible to open launch Revit then open a RVT file programmatically in Revit 2012?
I can do this in Revit 2013 via the Idling event.

**Answer:** Yes, definitely.
Simply specify the project filename on the Revit.exe command line.

**Response:** I got it to work after all.
I just open a small dummy RVT before processing the RVTs for the users.
I am using this to implement a batch PDF printer than can run from the Windows Task Scheduler overnight.

Based on this approach, Dan implemented AutoPDFPrint, a Revit multi-project batch PDF printer than can be driven by the Windows Task Scheduler overnight.

#### Task Scheduler and Revit Idling

Dan underlines that the Task Scheduler and Revit API Idling event allow you to automate almost any task.
A few that help my firm are:

- PDF creation- DWG creation- Audit RVT files- Compact RVT files- Standards compliance checking- Extracting data from RVT files

Some new features in the Revit 2014 API make the above tasks easier still, e.g. the OpenDocumentFile method that now enables you to open workshared RVT files detached from central and determine which user created worksets are open or closed.

There are a couple of things you need to consider before running any of the above tasks:

- Make sure the computer you use has enough RAM. Process one RVT file at a time to determine how much time and RAM it takes to finish. You may need to process one RVT file per Task Scheduler Task since Revit does not free memory until it closes.- Warnings can cause the process to fail. The tool currently clicks OK for any dialog that opens. If a warning opens during the open process that cannot be closed by clicking OK, the file is skipped. This can be a good thing since it will inform you if an RVT has an issue while opening you need to investigate.

#### AutoPDFPrint Components

Looking in more detail at the PDF printer system, one its parts is the AutoPDFPrint\_Bridge executable run from the task scheduler:

![AutoPDFPrint_Bridge executable](img/auto_pdf_print_bridge.png)

The second component and the main actor in the process is the AutoPDFPrint Revit add-in:

![AutoPDFPrint Revit add-in](img/auto_pdf_print_addin.png)

Here are the detailed steps required to set up the system:

1. [Adobe PDF print preferences](#6)
2. [CFG file](#7)
3. [Task scheduler](#8)

All components presented are available for [download](#9) below.

#### Step #1 – Set Up the Adobe PDF Print Preferences

1. Click the Windows 'Start' button and select 'Devices and Printers'.- Right-click the 'Adobe PDF printer' and select 'Printing Preferences'.- Change these settings:
       1. 'Adobe PDF Page Size' to user Documents folder.- 'View Adobe PDF results': disable.

![Adobe PDF Print Preferences](img/auto_pdf_print_1.png)

#### Step #2 – Set Up the CFG File

1. The CFG file goes here:

   ```
   C:\AutoPDFPrint2014\Scripts
   ```

   - CFG file example name:

     ```
     100XXX.00 - My Project.cfg
     ```

     - CFG file format (comma delimited):

       ```
       <view/sheet set>,<print set>,<RVT file>,<output folder for PDFs>
       ```

![CFG File](img/auto_pdf_print_2.png)

Notes:

- Multiple files/print jobs can be processed in the same CFG file.- Make sure enough hard drive space and RAM exists on the computer.

#### Step #3 – Set Up the Task Scheduler

1. Open the Windows Task Scheduler:

   Click the Windows 'Start' button and select All Programs > Administrative Tools > Task Scheduler.- Create a new Task.

     In the Actions tab:
     1. Program/script:

        ```
        C:\AutoPDFPrint2014\Program\AutoPDFPrint_Bridge.exe
        ```

        - Add arguments (optional):

          ```
          “100XXX.00 - My Project.cfg”
          ```

![Task Scheduler](img/auto_pdf_print_3.png)

Notes:

- No guarantee the tool will work if a user is not logged on to the computer.
  This is related to the issue specified in the add-in flowchart above concerning PrintManager.PrintToFileName.- Make sure the value for 'Add Arguments (optional)' is surrounded by quotes.

#### Download

Here is [Auto\_PDF\_Print\_from\_Revit\_2014\_example.zip](zip/Auto_PDF_Print_from_Revit_2014_example.zip) containing the following:

0. ReadMe.txt (this list).
1. Auto PDF Print from Revit 2014.vsd (pseudo logic).
2. Auto PDF Print from Revit 2014.docx (instructions for use).
3. Folders.zip (copy the Projects and AutoPDFPrint folders to the root C:).
4. AutoPDFPrint.zip (Revit 2014 add-in Application).
5. AutoPDFPrint\_Bridge.zip (EXE run from Task Scheduler).

Thank you very much, Dan, for implementing and sharing this powerful tool.