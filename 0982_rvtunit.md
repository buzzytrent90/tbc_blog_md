---
post_number: "0982"
title: "Revit Add-in Unit Testing"
slug: "rvtunit"
author: "Jeremy Tammik"
tags: ['csharp', 'parameters', 'references', 'revit-api', 'views']
source_file: "0982_rvtunit.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0982_rvtunit.html"
---

### Revit Add-in Unit Testing

Here is a topic I have been flirting with for years, and never really got into seriously yet, except point to the efforts for
[unit testing in Revit 2011](http://thebuildingcoder.typepad.com/blog/2010/11/unit-testing-in-revit.html) by
Rod Howarth and Daren Thomas back in 2010.

Finally, the cavalry is galloping in to our rescue in the shape of Steven Downing, Ali Talebi and Yamin Tengono of
[Arup](http://www.arup.com).

They implemented a very complete and advanced unit testing system for Revit add-ins enabling
[NUnit](http://www.nunit.org) as
well as other test helpers like
[Moq](http://code.google.com/p/moq) to
be used and including support for real-time addition of new tests, recompilation and reloading of add-ins, and re-running updated code and tests without restarting Revit.

The entire
[RvtUnit](https://github.com/ArupAus/RvtUnit) project
including a sample project demonstrating its use is hosted on GitHub.

It includes a
[getting started document](https://github.com/ArupAus/RvtUnit/blob/master/Getting%20Started.docx) and a
[demonstration video](https://github.com/ArupAus/RvtUnit/blob/master/rvtUnit_Demo.mp4) showing the steps I just described:

- Launching the TestRunner external command
- Selecting a directory containing .NET assemblies defining tests to run
- The assembly DLL is loaded via a byte-code stream, so the actual file is not locked
- The tests to run are dynamically detected using custom attributes and .NET reflection
- Running the tests and reporting the unit test run results
- Switching to Visual Studio, adding a new test, recompiling
- Dynamically reloading, reparsing and retesting without leaving or restarting Revit
- Switching to Visual Studio again and adding a detailed test failure message, recompiling
- Reloading and retesting once again with the new message displayed

I would suggest looking at the 2.5 minute demonstration video right away, first thing, to see the system in action and understand exactly what can be achieved and how:

Here is the detailed content of 'Getting Started.docx':

- [Introduction](#2)
- [RvtUnit project](#3)
- [SampleTool project](#4)

#### Introduction

RvtUnit is an example of how to achieve Unit Testing within Revit. By running the NUnit runner inside Revit, we can unit test code which relies on the Revit API, without having to wrap the entire Revit API.

The solution contains three projects:

- The RvtUnit project is the main project that runs the unit tests.
- The SampleTool project shows an example of a Revit IExternalCommand that includes unit tests.
- The Helpers project contains some code used by both the other projects.
  Notably, the RvtUnit project will store a reference to the ActiveUIDocument here, so the unit tests can pick it up.

![RvtUnit solution projects](img/sd_rvtunit.png)

#### RvtUnit Project

- Designed using the [MVVM](http://en.wikipedia.org/wiki/Model_View_ViewModel) pattern.
- References a custom build of NUnit which has been modified to load DLL's from a byte array, rather than from disk
- Allows user to select a directory of DLL's, and it will then load them and present a dialog which lets the user run some or all tests.
- Contains an 'AssemblyResolve' event handler, so that any dependencies of the DLL to be tested can also be loaded. Since the assembly to be tested is loaded dynamically from a byte-array, the normal .NET Framework mechanism will not work.
- The Test runner will not execute the IExternalCommand of any DLL's. It will simply load the DLL, look for any unit tests in the DLL and then run them.

#### SampleTool Project

- Contains an example of an IExternalCommand that includes some classes which are unit tested.
- The 'production code' and the 'test code' are compiled into the same DLL when the project is compiled in 'Debug' mode.
- The test code is omitted when the project is compiled in 'Release' mode.
  This is achieved by putting test code and references inside an ItemGroup that only compiles for Debug:
```csharp
<ItemGroup Condition="'$(Configuration)|$(Platform)' == 'Debug|AnyCPU'">
  <Reference Include="Moq">
    <HintPath>..\rvtUnit\Lib\Moq.dll</HintPath>
  </Reference>
  . . .
  <Compile Include="Tests\UnitTests\HasParameter\_Tests.cs" />
  <None Include="Tests\Features\SetParameter.feature">
    <Generator>SpecFlowSingleFileGenerator</Generator>
    <LastGenOutput>SetParameter.feature.cs</LastGenOutput>
  </None>
</ItemGroup>
```- Contains standard unit tests, and Specflow tests.
- The Specflow test also use Moq, although a custom build of both Moq and Castle.Core are required, as the objects must be properly disposed at the end of the test.
- If you run the SampleTool from the Addins menu, it will be loaded from disk and the file locked as per normal. You will then be unable to change, rebuild and reload it.

#### Conclusion

Jeremy adds: Wow. I have nothing left to add. Almost.

The RvtUnit sample demonstrates a number of other important and exciting other features as well, such as support for multiple versions of Revit, ranging all the way from 2010 through 2014, so a thorough exploration of the source code provided is highly recommended.

To run and test this, you need to start up Revit properly, stand-alone, not in the debugger from within the Visual Studio IDE.

A huge thank-you to Steven, Ali and Yamin for this truly impressive framework!