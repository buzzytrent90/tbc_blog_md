---
post_number: "0746"
title: "DevCamp and Refresh Display for a Kinetic Facade"
slug: "kinetic_facade"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'revit-api', 'transactions', 'views', 'walls', 'windows']
source_file: "0746_kinetic_facade.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0746_kinetic_facade.html"
---

### DevCamp and Refresh Display for a Kinetic Facade

Welcome back from the Easter break, if you had one.
We are pretty busy right now preparing for our next conference, the DevCamps.

Ah yes, and before that I have to get ready for the
[Revit API Training in Munich](http://thebuildingcoder.typepad.com/blog/2012/04/migrating-vsta-macros-to-sharpdevelop.html#1) coming up soon.

#### AEC DevCamp and DevLab – Join us at Camp this Summer!

The bi-annual Autodesk AEC DevCamp is taking place near Boston on June 6-7.

DevCamps are held every other year where software developers like you can learn to get the most from your time and effort working with Autodesk platform technologies. The two industry specific Developer Camps include several tracks catering to the learning needs of beginner and expert software developers – as well as a business track for start-ups looking to develop and leverage their relationship with Autodesk. Come to Camp and get two days of face to face learning and help from Autodesk software engineers – the same Autodesk software engineers developing and supporting the Autodesk technology you work with every day.

You can also join us for a third day at DevLab, where you can work on your hardest coding challenges with an Autodesk software engineer looking over your shoulder giving you immediate help and advice. Bring your laptop and work on your code as you'd normally do in your own office – the difference is that you have a team of DevTech experts (the same people who answer your questions through DevHelp Online) available to review your code if you'd like, answer your questions, and make suggestions.

**Cloud and Mobile Classes:** New for this year's DevCamps are several classes on helping you get started developing apps based on Cloud and Mobile technologies. Learn how to build your first Cloud app integrated with your favourite Autodesk product – and how you can make it available 'everywhere' through browser and mobile user interfaces. Have you been considering building your first app for the iPhone, iPad or Android device but holding back because of fear of lost hours thrashing while learning the new technology? DevCamp includes classes that will show you how to quickly and easily create your first 'Hello World' app for iOS and Android - with a design and graphics twist.

AEC DevCamp is taking place June 6-7 near Boston.

You can attend beginner and expert classes on developing solutions with Revit, Civil 3D, Infrastructure Map Server, Vault, Autodesk BIM 360, Project Vasari, Green Building Studio, Navisworks, and other Autodesk AEC technologies. Hear from Autodesk's AEC Leadership including Vice President Jim Lynch. Learn firsthand about developing a new cloud based business leveraging Autodesk technologies from the co-founder of Horizontal Systems (recently acquired by Autodesk) Jordan Brandt.

Here is a
[complete list of classes](https://custom.cvent.com/FDBB345248B94F40BFFFCEF2FBE054E4/files/645f182b028d480281ebdda12bae6576.pdf) offered
at AEC DevCamp, and you can
[register here](http://www.cvent.com/d/ycq03r).
Register by April 30th to get the $100 early bird discount!

If you would like to participate in the optional AEC DevLab on June 8, please
[complete a separate registration](http://usa.autodesk.com/adsk/servlet/item?id=6703509&siteID=123112&cname=DevLab%20AEC,%20Waltham,%20Jun%2008%202012,%20201253).

#### Refreshing a View

Meanwhile, another issue on the Revit API front: I have often been asked how to refresh a view, and the only last resort I was aware of until recently was to commit a transaction and save the document to an external file.

Now this question cropped up again, asked by Prof. Jos Luis Menegotto of
[Desenho Computacional](https://sites.google.com/a/poli.ufrj.br/dc_menegotto/home) at
the University of Rio de Janeiro.
This time around, I received an important new hint: the UIDocument class provides the RefreshActiveView method.

Here is the initial query:

**Question:** I need to show a sequence of changes in a Curtain Wall Panels facade.
I create a main transaction and 4 sub-transactions to show the changes with a delay of 2 seconds between them, but the result always shows me the last transaction only.
What am I doing wrong?
```csharp
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  FilteredElementCollector colecao
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .OfCategory(
        BuiltInCategory.OST\_CurtainWallPanels );

  ICollection<ElementId> Paneis
    = colecao.ToElementIds();

  Transaction tran = new Transaction( doc );

  SubTransaction sTra1 = new SubTransaction( doc );
  SubTransaction sTra2 = new SubTransaction( doc );
  SubTransaction sTra3 = new SubTransaction( doc );
  SubTransaction sTra4 = new SubTransaction( doc );

  tran.Start( "Muda a fachada" );

  sTra1.Start();
  Calcula\_Padrao( doc, Paneis, 0.10 );
  sTra1.Commit();

  Thread.Sleep( 2000 );

  sTra2.Start();
  Calcula\_Padrao( doc, Paneis, 0.4 );
  sTra2.Commit();

  Thread.Sleep( 2000 );

  sTra3.Start();
  Calcula\_Padrao( doc, Paneis, 0.05 );
  sTra3.Commit();

  Thread.Sleep( 2000 );

  sTra4.Start();
  Calcula\_Padrao( doc, Paneis, 0.4 );
  sTra4.Commit();

  tran.Commit();

  return Result.Succeeded;
```

**Answer:** First, I can reproduce what you observe.

Secondly, I tried several alternative approaches using transactions to force the model to redisplay after each modification, besides your initial implementation:

- Start and commit a SubTransaction: no intermediate updates.- Start and commit a SubTransaction plus regenerate the document: no intermediate updates.- Start and commit a Transaction (this includes a doc regen): no intermediate updates.- Start and commit a Transaction plus regenerate the document, for safety's sake: no intermediate updates.- Start and commit a Transaction plus save the document to a file: this does update the model in between each step.

One thing to point out here is that sub-transactions have no effect on the UI at all and therefore are completely useless for this purpose.

This does show that you can actually achieve what you want by using transactions instead of subtransactions, though, and saving the document to a file after each step.

However, a much better solution is provided by the UIDocument.RefreshActiveView method.

You can start a new transaction for each step and call RefreshActiveView after each transaction successfully commits instead of saving to an external file.

If the command is supposed to appear as a single step on the undo stack in the user interface once the command is finished, the transactions can simply be wrapped (assimilated) in a transaction group.

Here is the final code implementing this:
```csharp
  TransactionGroup group = new TransactionGroup( doc );
  Transaction tran = new Transaction( doc );

  group.Start( "Muda a fachada" );

  tran.Start( "Step 1 " );
  Calcula\_Padrao( doc, Paneis, 0.10 );
  tran.Commit();

  uidoc.RefreshActiveView();

  Thread.Sleep( 2000 );

  tran.Start( "Step 2 " );
  Calcula\_Padrao( doc, Paneis, 0.4 );
  tran.Commit();

  uidoc.RefreshActiveView();

  Thread.Sleep( 2000 );

  tran.Start( "Step 3 " );
  Calcula\_Padrao( doc, Paneis, 0.05 );
  tran.Commit();

  uidoc.RefreshActiveView();

  Thread.Sleep( 2000 );

  tran.Start( "Step 4 " );
  Calcula\_Padrao( doc, Paneis, 0.4 );
  tran.Commit();

  uidoc.RefreshActiveView();

  group.Assimilate();
```

Here is
[FachadaCinetica.zip](zip/FachadaCinetica.zip) containing
the complete Revit 2013 source code, Visual Studio solution and add-in manifest for the sample command.

It also includes code for the intermediate attempts, a wait cursor, and debug trace statements to show whereabouts we are in the process in the Visual Studio debug oputput window.

**Response:** It works fine, and here is the
[final result](https://sites.google.com/a/poli.ufrj.br/dc_menegotto/programacao-autolisp/ArqCin) for
both AutoCAD and Revit.