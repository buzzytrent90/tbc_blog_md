---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: code_example
optimization_date: '2025-12-11T11:44:14.592950'
original_url: https://thebuildingcoder.typepad.com/blog/0788_command_ids.html
post_number: 0788
reading_time_minutes: 3
series: general
slug: command_ids
source_file: 0788_command_ids.htm
tags:
- csharp
- family
- python
- revit-api
- views
title: Replacing Built-In Revit Commands and Their Ids
word_count: 682
---

﻿

### Replacing Built-In Revit Commands and Their Ids

One main focus of the
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html) features
is better support for
[add-in integration](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2).
This includes the possibility for an add-in to replace an existing Revit command with its own implementation.
Note that you cannot call an existing command, just replace it entirely.

Such a command replacement is demonstrated in a very simple form by the
[DisableCommand SDK sample](http://thebuildingcoder.typepad.com/blog/2012/04/developer-center-and-sdk-update.html#21),
which disables a command by replacing its implementation with a simple popup message.

A slightly more complex example is given by the
[UIAPI SDK sample](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4),
which implements and installs an alternative command binding for the Design Model command to start up a new family freshly created from the conceptual mass template and display its 3D view.

These are the steps to replace a built-in Revit command:

- Determine the RevitCommandId to replace by examining the journal file and searching for the name of the original command.- Create an AddInCommandBinding for this command id.- Provide an alternate implementation for the command binding.

#### List of all Revit Command Ids

To simplify the first step, Victor Chekalin aka Виктор Чекалин now presents a list of all Revit command ids,

In Victor's own words:

I'm learning the ability to replace a Revit command with my own add-in implementation.
To replace a command, I must know CommandId of the command which I want to replace.
The help tells me I can find a CommandId in the journal file.
So the Revit SDK help doesn't contains full list of CommandIds.
I have decided to correct this mistake and create that list.

I thought this information will be useful for you and other developers and send you the list of all command ids in both
[text](zip/CommandIds.txt) and
[Excel](zip/CommandIds.xslx)
format.

**Question:** Wow!
How did you create this list?

**Answer:** It is my little secret how I've got it. Joke :-)

At first I thought the commands must be described somewhere.
I searched for the text 'ID\_EXPORT\_IFC' (the command name from the Dev Days Online – Revit 2013 API) in all files at the Revit Program folder.
So I found the UIFramework.dll file.
I opened this file in the text editor and saw XML data containing command descriptions.

Next steps were very easy.
I saved this part of the UIFramework.dll file to the XML file and read the XML data:
```python
private IEnumerable<InternalCommandDef>
  ReadCommandsFromFile()
{
  using( var streamReader
    = File.OpenText( @"C:\Users\ChekalinVV"
      + "\Documents\Revit\Revit2013SDK"
      + "\RevitUICommands.xml" ) )
  {
    using( var reader = XmlReader.Create(
      streamReader ) )
    {
      while( reader.Read() )
      {
        if( reader.Name.Equals( "Command" ) )
        {
          InternalCommandDef commandDef
            = new InternalCommandDef();

          if( reader.MoveToAttribute( "Path" ) )
            commandDef.Path = reader.Value;

          if( reader.MoveToAttribute( "CommandId" ) )
            commandDef.CommandId = reader.Value;

          yield return commandDef;
        }
      }
    }
  }
}
```

Retrieve get CommandId info for each command and write it to the text file:
```csharp
  using( var textFile = File.CreateText(
    @"C:\Users\ChekalinVV\Documents\Revit"
    + "\Revit2013SDK\RevitUICommands.txt" ) )
  {
    foreach( var command in commands )
    {
      RevitCommandId commandId = RevitCommandId
        .LookupCommandId( command.CommandId );

      if( commandId == null ) continue;

      command.CanHaveBinding
        = commandId.CanHaveBinding;

      command.Id = commandId.Id;

      textFile.WriteLine( "{0}\t{1}\t{2}\t{3}",
          command.CommandId,
          command.Path,
          command.Id,
          command.CanHaveBinding );

      /\* It is just a joke

      if( !commandId.HasBinding
        && commandId.CanHaveBinding )
      {
        var commandBinding =
          App.ControlledApplication
            .CreateAddInCommandBinding(commandId);

        commandBinding.Executed += OnCommandExecute;
      }
      \*/
    }
  }
}
```

It is not necessary to get full CommandId properties, but I wanted to retrieve the CanHaveBinding property for all commands.
As I can see only two commands cannot have binding: Undo and Redo commands.
Although I try to bind ID\_APP\_EXIT command (just for test because it was the first command I found in the journal files).
It binds without any errors but doesn't work.

Very many thanks to Victor for this interesting research and the useful comprehensive list!