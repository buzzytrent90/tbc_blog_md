---
post_number: "1040"
title: "Programmatic Custom Add-In External Command Launch"
slug: "postcommand"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'parameters', 'revit-api', 'rooms', 'schedules', 'sheets', 'views']
source_file: "1040_postcommand.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1040_postcommand.html"
---

### Programmatic Custom Add-In External Command Launch

One of the
[highlights](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html#2) of
the
[Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) is
the new PostCommand functionality enabling programmatic launching of built-in Revit commands.

The Revit SDK includes a new sample demonstrating its use, PostCommandWorkflow.

However, that programmatically launches an existing built-in Revit command, not an external command added by a custom application.

Furthermore, there was a problem launching custom external commands in the initial release of Revit 2014.
Happily, it was fixed by the update release 1, UR1, a fact listed among the
[Revit 2014 update release 1 API enhancements](http://thebuildingcoder.typepad.com/blog/2013/08/revit-2014-update-release-1.html):

- Allows UIApplication.PostCommand() to work consistently for Add-in created commands.

So how do we make use of this functionality?

Here is today's question from – and my exploration of – a recent developer query, with separate yet similar answers for [external tool](#1) and [custom ribbon button](#2) commands.

First, however, let me mention that we had a wonderful sunny weekend here, and my friend Karin told a nice joke that I would like to share:

Two planets happen to meet somewhere, wandering around through the milky way.
Says one: "Hi. Be careful, I have homo sapiens."
Replies the other: "Hi. Don't worry, it will pass."

#### Programmatically Launching an External Tool Add-In Command

**Question:** The PostCommand Revit API call seems to expect a RevitCommandId argument:

```
  commandData.Application.PostCommand(revitCmdID)
```

I tried to obtain a command id using the LookupCommandId method, passing in the name of my external command class, but it returns nothing:

```
  Dim revitCmdID As Autodesk.Revit.UI.RevitCommandId _
    = RevitCommandId.LookupCommandId("DoorCommandLogger")

  Dim revitCmdID As Autodesk.Revit.UI.RevitCommandId _
    = RevitCommandId.LookupCommandId("SwingTool.DoorCommandLogger")
```

I also looked at the journal file after running the command from the user interface and tried passing in some variations of the different names I discovered there, such as:

- "Execute external command:CustomCtrl\_%CustomCtrl\_%NTItools%Parameters%Door Tool:SwingTool.DoorCommandLogger"
- "CustomCtrl\_%CustomCtrl\_%NTItools%Parameters%Door Tool:SwingTool.DoorCommandLogger"

Still I get nothing.

How can I post a custom command, please?

**Answer:** Let's start out by taking a more detailed look at the PostCommandWorkflow SDK sample and the classes involved in this scenario.

Here is the code used there to actually launch the command:

```csharp
  /// <summary>
  /// Prompts to edit the revision and resave.
  /// </summary>
  /// <param name="application"></param>
  private void PromptToEditRevisionsAndResave(
    UIApplication application )
  {
    // Setup external event to be notified
    // when activity is done

    externalEvent = ExternalEvent.Create(
      new PostCommandRevisionMonitorEvent(
        this ) );

    // Setup event to be notified when revisions
    // command starts (this is a good place to
    // raise this external event)

    RevitCommandId id
      = RevitCommandId.LookupPostableCommandId(
        PostableCommand.SheetIssuesOrRevisions );

    if( binding == null )
    {
      binding = application
        .CreateAddInCommandBinding( id );
    }

    binding.BeforeExecuted
      += ReactToRevisionsAndSchedulesCommand;

    // Post the revision editing command

    application.PostCommand( id );
  }
```

There is lots of stuff in there that is of no concern to us now and here.

Actually, the only two lines of interest are the calls to LookupPostableCommandId and PostCommand.

LookupPostableCommandId takes a PostableCommand enumeration value as an argument and returns a RevitCommandId instance, which is the argument required by the call to PostCommand.

The RevitCommandId class internally manages a command name and id.

In the case of the SheetIssuesOrRevisions command retrieved by PostCommandWorkflow, these are

- Id = 3153
- Name = "ID\_SETTINGS\_REVISIONS"

You can easily see this in the Visual Studio debugger by implementing an external command executing this single statement:

```csharp
  RevitCommandId id\_built\_in
    = RevitCommandId.LookupPostableCommandId(
      PostableCommand.SheetIssuesOrRevisions );
```

OK, so built-in Revit commands that can be posted are listed in the PostableCommand enumeration, and we can use LookupPostableCommandId to retrieve the appropriate RevitCommandId for posting.

However, a custom add-in defining its own external command will obviously not be included there, so what can we do?

I studied the Revit API help file for these classes and found the following:

The RevitCommandId class represents a command id in Autodesk Revit.
Each Revit command is assigned a command id and non-localised name.
This class allows you to look up a command by its name, and represents any Revit command in the use of an AddInCommandBinding.

OK, but where to obtain it?

LookupPostableCommandId takes an enumeration value, and we do not have any to offer.

However, the RevitCommandId class provides exactly two static methods, LookupPostableCommandId and LookupCommandId.

The former takes a PostableCommand enumeration value, the latter a simple string, and its documentation states:

You can use the RevitCommandId.LookupCommandId method to retrieve a corresponding Revit command id for a given id string.
You can refer to the entries in the Revit journal to find the string to use for a particular command.

So I did just what you already tried and described in your query:

- Implement a dummy command named CmdDummy, for example.
- Launch it manually in the user interface.
- Search for all occurrences of the string "Dummy" in the resulting journal file.

Here is the result (copy and paste somewhere or view source to see truncated lines in full):

```
C:\Users\tammikj\AppData\Local\Autodesk\Revit\Autodesk Revit 2014\Journals > grep -i dummy journal.0242.txt

' 0:< Added new API pushbutton 35024, name PostAddinCommand Dummy Command, class PostAddinCommand.CmdDummy, assembly PostAddinCommand.dll, vendorId TBC_, vendor description The Building Coder, http://thebuildingcoder.typepad.com.

' 4:< MasterLocks 0x0000000014BB1250 DummyStorage stole m_oDataStorage 0x0000000014C52950 but left m_pDataStorage 0x0000000014C52950

Jrn.RibbonEvent "Execute external command:64b3d907-37cf-4cab-8bbc-3de9b66a3efa:PostAddinCommand.CmdDummy"

' 0:< DummyStorage destroying DataStorageInterface 0x0000000014C52950
```

Eliminating the DummyStorage entries, which are probably not caused by my application, leaves only the following candidate entry for me to try to extract the appropriate string from:

- Jrn.RibbonEvent "Execute external command:64b3d907-37cf-4cab-8bbc-3de9b66a3efa:PostAddinCommand.CmdDummy"

The GUID prefixed to the command name happens to be the external command ClientId from my add-in manifest:

```
  <AddIn Type="Command">
    <Text>PostAddinCommand Dummy Command</Text>
    <Description>Test command for PostAddinCommand.</Description>
    <Assembly>PostAddinCommand.dll</Assembly>
    <FullClassName>PostAddinCommand.CmdDummy</FullClassName>
    <ClientId>64b3d907-37cf-4cab-8bbc-3de9b66a3efa</ClientId>
    <VendorId>TBC_</VendorId>
    <VendorDescription>The Building Coder, http://thebuildingcoder.typepad.com</VendorDescription>
  </AddIn>
```

My first attempt was therefore to use the following string as an input to LookupCommandId:

- "64b3d907-37cf-4cab-8bbc-3de9b66a3efa:PostAddinCommand.CmdDummy"

That returned null, just as in your attempts.

In my next attempt, I just used the client id all on its own.

Lo and behold, that works perfectly fine.

Here is the resulting external command demonstrating both methods, and launching my test command successfully:

```csharp
  UIApplication uiapp = commandData.Application;

  // Built-in Revit commands are listed in the
  // PostableCommand enumeration

  RevitCommandId id\_built\_in
    = RevitCommandId.LookupPostableCommandId(
      PostableCommand.SheetIssuesOrRevisions );

  // External commands defined by add-ins are
  // identified by the client id specified in
  // the add-in manifest

  string name
    = "64b3d907-37cf-4cab-8bbc-3de9b66a3efa";

  RevitCommandId id\_addin
    = RevitCommandId.LookupCommandId(
      name );

  uiapp.PostCommand( id\_addin );
```

The resulting numerical id is 35024, and the name is simply the client id GUID.

This numerical id is also listed in the journal file when the pushbutton is created, as you can see above.

The entire source code, Visual Studio solution and add-in manifest for the whole test is available from my
[PostAddinCommand GitHub repository](https://github.com/jeremytammik/PostAddinCommand), and
the version described up until now is
[2014.0.0.1](https://github.com/jeremytammik/PostAddinCommand/releases/tag/2014.0.0.1).

#### Launching a Custom Add-In Ribbon Button Command

We have now learned how to programmatically launch an external tool command defined in the add-in manifest and therefore equipped with its own client id.

Most serious applications do not list individual commands explicitly in the add-in manifest, though, because it is not very user friendly to have to navigate to the Add-Ins tab External Tools menu each time you want to launch them.

Actually, there are two completely different types of external commands, both implementing the IExternalCommand interface:

- External tool commands, listed in the add-in manifest and launched by selecting the entry added by Revit in the Add-Ins > External Tools menu.
- Custom button commands launched by clicking a custom button created by an external application.

The latter have no client id.

Still, the procedure of analysing the journal file to discover the corresponding command name string to pass into the LookupCommandId method remains exactly the same, even thought the result differs.

I implemented a new external command CmdDummy2 to test the latter case, and an external application to create a custom ribbon panel and custom button to launch it.

Repeating the steps described above to manually launch the command and examine the journal file to discover its command name string produced the following lines (copy and paste somewhere or view source to see truncated lines in full):

```
c:\Users\tammikj\AppData\Local\Autodesk\Revit\Autodesk Revit 2014\Journals>grep Dummy2 journal.0249.txt

' 0:< Added new API pushbutton 6417 name  text Dummy2 class PostAddinCommand.CmdDummy2 assembly C:\Users\tammikj\AppData\Roaming\Autodesk\Revit\Addins\2014\PostAddinCommand.dll

 Jrn.RibbonEvent "Execute external command:CustomCtrl_%CustomCtrl_%Add-Ins%Post Add-in Command%Dummy2:PostAddinCommand.CmdDummy2"

' 1:< TaskDialog "Hello from CmdDummy2!"
```

The command name string is thus

- "CustomCtrl\_%CustomCtrl\_%Add-Ins%Post Add-in Command%Dummy2"

This is concatenation of a hierarchical list of entries separated by percentage signs '%'.

The first two entries appear to be fixed constants, "CustomCtrl\_".

The last three are the sequence of controls a user needs to navigate through to manually launch the corresponding command, in this case the standard Revit Add-Ins ribbon tab, the custom "Post Add-in Command" ribbon panel, and the text of the ribbon button item, "Dummy2".

Although this string may seem a bit unwieldy, it does have the advantage that it can be controlled and generated by the add-in itself according to these rules, since each component is based on an add-in-defined string constant.

The following code successfully generates a RevitCommandId for this button and programmatically launches the command:

```csharp
  // External tool commands defined by add-ins are
  // identified by the string listed in the
  // journal file when the command is launched
  // manually.

  string name\_addin\_button\_cmd
    = "CustomCtrl\_%CustomCtrl\_%"
      + "Add-Ins%Post Add-in Command%Dummy2";

  RevitCommandId id\_addin\_button\_cmd
    = RevitCommandId.LookupCommandId(
      name\_addin\_button\_cmd );

  uiapp.PostCommand( id\_addin\_button\_cmd );
```

Stepping through the execution of this code in the Visual Studio debugger and examining the resulting RevitCommandId instance shows that its internal numerical id field value equals 6417.

This is the same number listed in the journal file during the creation of the ribbon pushbutton.

In the case above of the external tool command, the RevitCommandId numerical id was also equal to the pushbutton number.

If you look at the RevitCommandId numerical id produced by instantiation from a built-in Revit command PostableCommand enumeration value, you will note that they end up equal.

This led me to wonder whether it might be possible to cast the pushbutton number to a PostableCommand enumeration value and use LookupPostableCommandId to look it up instead of going through this rigmarole of confusing command name strings.

#### The Pushbutton Number Cannot be Cast to PostableCommand

Well, in short, I tested it and it does not work.

I cast the pushbutton number of both custom external commands listed in the journal file to PostableCommand enumeration values and passed them in to the LookupPostableCommandId.

Both attempts threw the same Revit ArgumentException saying "Invalid PostableCommand".

#### Summary and Download

Well, enough time spent on that, I think, and now all should be clear.

The updated version of the PostAddinCommand test add-in defines quite a number of classes:

- CmdDummy – dummy external tool command.
- CmdPost – external command to programmatically launch CmdDummy.
- CmdDummy2 – dummy custom ribbon button command.
- App – external application defining a ribbon panel and button to launch CmdDummy2.
- CmdPost2 – external command to programmatically launch CmdDummy2.
- CmdPostId – external command showing that the cast from pushbutton number to PostableCommand fails for CmdDummy.
- CmdPostId2 – ditto for CmdDummy2.

The entire source code, Visual Studio solution and add-in manifest for the updated test suite is available from my
[PostAddinCommand GitHub repository](https://github.com/jeremytammik/PostAddinCommand), and
the version described here is
[2014.0.0.2](https://github.com/jeremytammik/PostAddinCommand/releases/tag/2014.0.0.2).

This GitHub thingy is pretty handy, actually.

Oh, and now I finally performed the final step of setting up my git access on the Mac, to cache my credentials as described in
[password caching](https://help.github.com/articles/set-up-git):

```
  git config --global credential.helper osxkeychain
```

```csharp
```

####