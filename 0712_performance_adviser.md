---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.9
content_type: code_example
optimization_date: '2025-12-11T11:44:14.445016'
original_url: https://thebuildingcoder.typepad.com/blog/0712_performance_adviser.html
post_number: '0712'
reading_time_minutes: 8
series: general
slug: performance_adviser
source_file: 0712_performance_adviser.htm
tags:
- csharp
- elements
- filtering
- python
- revit-api
- transactions
- views
- walls
title: The Performance Adviser API
word_count: 1633
---

### The Performance Adviser API

Unbelievable as it may seem, there are still a number of new areas of functionality added in the Revit 2012 API that we have not even got around to covering here yet at all.
One of these is the performance adviser.
Here is an article on that by my colleague Saikat Bhattacharya from the ADN AEC newsletter.

The Performance Adviser API added in Revit 2012 allows developers to execute a set of built-in or custom rules against a given model, and to display a warning dialog about the elements that fail to satisfy the rules. You can use this feature to flag the user about possible performance degradations, and to create your own rules that could be run on a model to check for adherence to certain design guidelines. For example, you may want to signal the user where walls may be overlapping.

This article provides an overview and discusses how to
[access performance adviser rules](#1),
[execute rules](#2), and
[define custom rules](#3).

#### Accessing Performance Adviser Rules

The performance adviser rules are a set of application-wide rules. To work with performance adviser rules, we use an application-wide, singleton class called PerformanceAdviser. This class is a registry of all the performance checking rules as well as an engine to execute the rules. You can access the singleton PerformanceAdviser instance using a PerformanceAdviser.GetPerformanceAdviser static method.

Once we have access to the PerformanceAdviser object, we can use the GetAllRuleIds method to obtain a list of rule IDs registered in the application. We can then iterate through each rule ID and get the rule information for the corresponding rule ID, using the methods, such as GetRuleName and GetRuleDescription.

The following command illustrates how to list all the registered performance adviser:
```python
/// <summary>
/// List all registered performance adviser rules
/// </summary>
[Transaction( TransactionMode.Manual )]
public class ListPerformanceAdviserRules
  : IExternalCommand
{
  public Autodesk.Revit.UI.Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    // Get access to Performance Adviser object

    PerformanceAdviser adviser
      = PerformanceAdviser.GetPerformanceAdviser();

    // Access the rules

    IList<PerformanceAdviserRuleId> ruleIds
      = adviser.GetAllRuleIds();

    String ruleInfo = String.Empty;

    // Loop through each rule Id and get rule name and description

    foreach( PerformanceAdviserRuleId ruleId
      in ruleIds )
    {
      ruleInfo += "\n" + adviser.GetRuleName( ruleId )
        + " : " + adviser.GetRuleDescription( ruleId );
    }

    TaskDialog.Show( "Existing Rules Info", ruleInfo );

    return Result.Succeeded;
  }
}
```

#### Executing a Performance Adviser Rule

Once you know the ids of rules that you are interested in, you can use the PerformanceAdviser.ExecuteRules method to execute the given rules.
This method takes a document and a set of rule ids as input arguments.
It returns a list of FailureMessages, which we can iterate through and display, for instance, in a standard Revit warning dialog using Document.PostFailure method.

For an introductory discussion about Failure Posting and Handling, please refer to
[this introduction](http://thebuildingcoder.typepad.com/blog/2010/04/failure-api.html),
[its follow-up](http://thebuildingcoder.typepad.com/blog/2010/11/failure-api-take-two.html),
and the ADN *AEC Customization Newsletter – BIM Edition – Winter 2011*.

The following command shows a simple example of executing the overlapping walls performance adviser rule:
```python
/// <summary>
/// Execute overlapping walls?
/// performance adviser rule.
/// </summary>
[Transaction( TransactionMode.Manual )]
public class ExecutePerformanceRule : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    Document doc = commandData.Application
      .ActiveUIDocument.Document;

    // Access the Performance Adviser object

    PerformanceAdviser adviser
      = PerformanceAdviser.GetPerformanceAdviser();

    // List all the rules

    IList<PerformanceAdviserRuleId> ruleIds = adviser.GetAllRuleIds();

    // Create an empty list for
    // the rules to be executed

    IList<PerformanceAdviserRuleId> rulesIdsToExecute =
      new List<PerformanceAdviserRuleId>();

    // Iterate through each

    foreach( PerformanceAdviserRuleId ruleId in ruleIds )
    {
      if( adviser.GetRuleName( ruleId ).Equals( "Overlapping walls" ) )
      {
        rulesIdsToExecute.Add( ruleId );
        break;
      }
    }

    // Now we need to post the results

    IList<FailureMessage> failureMessages =
      adviser.ExecuteRules( doc, rulesIdsToExecute );

    foreach( FailureMessage fm in failureMessages )
    {
      Transaction trans = new Transaction( doc );
      trans.Start( "Failure Reporting" );
      doc.PostFailure( fm );
      trans.Commit();
    }

    return Result.Succeeded;
  }
}
```

#### Defining a Custom Rule

Now, the most powerful part of this API is that we define custom adviser rules. To create our own, we define a class implementing an interface called IPerformanceAdviserRule. Your custom adviser class then needs to define the following methods:

- GetName – returns the name of this rule.- GetDescription – provides the description of this custom rule.- InitCheck – performs the initialization work before running the rule. For example, it could be a task like clearing a collection of elements that we will be storing elements that fail the rule execution test.- WillCheckElements – sets whether the rule will iterate through elements or not.- GetElementFilter – provides the element filter which can help narrow down the type or category of elements that a certain rule is expected to work with.- ExecuteElementCheck – This method contains most of the implementation steps for a given rule. It examines the element passed to it (which is filtered by the GetElementFilter) and checks for all elements that fail the criteria laid by the rule.- FinalizeCheck – This method gets called by the PerformanceAdviser after all the elements in a document matching the element filter in the GetElementFilter method have been checked by the ExecuteElementCheck method. This method usually will contain the code to check if there are elements which fail the rule test and if there are, the names of the elements are listed along with the standard warning failure message.

The code below shows an example of a custom performance adviser rule, which provides warnings if there are clashes between ducts in a Revit model:
```csharp
// A custom performance adviser rule class.
// This rule provides a warning if the model
// contains ducts which clash each other.

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using Autodesk.Revit.UI;
using Autodesk.Revit.DB;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB.Mechanical;
using System.Diagnostics;

namespace BuildingCoder
{
  public class DuctClashDetection
    : IPerformanceAdviserRule
  {
    private FailureDefinition m\_warning;
    private FailureDefinitionId m\_warningId;
    private List<ElementId> m\_ClashingDucts;

    /// <summary>
    /// DuctClashDetection Constructor
    /// </summary>
    public DuctClashDetection()
    {
      // Create failure message

      m\_warningId = new FailureDefinitionId(
        new Guid( "AACC9C09-7956-4B36-9D27-FF8A64544C31" ) );

      m\_warning = FailureDefinition.CreateFailureDefinition(
        m\_warningId, FailureSeverity.Warning,
        "Some ducts clash with each other" );
    }

    /// <summary>
    /// This method does most of the work of the
    /// IPerformanceAdviserRule implementation.
    /// It is called by PerformanceAdviser and
    /// examines the element passed to it
    /// (which was previously filtered by the filter
    /// returned by GetElementFilter).  This method
    /// is checking for all elements that intersect
    /// with current duct and adds them to the list
    /// of elements.
    /// </summary>
    public void ExecuteElementCheck(
      Document document,
      Element element )
    {
      // For each of the elements, check if there is
      // any intersection with the specific element
      // being passed on via the method signature

      FilteredElementCollector collector
        = new FilteredElementCollector( document );

      // Use the Elements Intersection Filter to get elements
      // that intersect with the specified element

      ElementIntersectsElementFilter elementFilter =
        new ElementIntersectsElementFilter( element, false );

      collector.WherePasses( elementFilter );

      // Exclude the element in the intersection

      List<ElementId> excludes = new List<ElementId>();
      excludes.Add( element.Id );
      collector.Excluding( excludes );

      // Save the intersecting ducts

      foreach( Element elem in collector )
      {
        m\_ClashingDucts.Add( elem.Id );
      }
    }

    // The rule ID for this rule;

    public PerformanceAdviserRuleId Id =
      new PerformanceAdviserRuleId(
        new Guid( "C00CEF6D-2C4B-402C-9AED-160DFA1785BE" ) );

    /// <summary>
    /// This method is called by PerformanceAdviser
    /// after all elements in document matching the
    /// ElementFilter from GetElementFilter() are
    /// checked by ExecuteElementCheck(). It checks to
    /// see if there are any ducts in the
    /// m\_ClashingDucts list. If so, it iterates
    /// through that list and reports the failures.
    /// </summary>
    public void FinalizeCheck( Document document )
    {
      try
      {
        // If no ducts clash

        if( m\_ClashingDucts.Count == 0 )
        {
          Debug.WriteLine(
            "No clashing ducts. Test passed." );
        }
        else
        {
          // Pass the element IDs of the clashing ducts to the revit
          // failure reporting APIs.

          FailureMessage fm =
            new FailureMessage( m\_warningId );

          fm.SetFailingElements( m\_ClashingDucts );

          Transaction failureReportingTransaction =
            new Transaction( document );

          failureReportingTransaction.Start(
            "Failure reporting" );

          PerformanceAdviser.GetPerformanceAdviser()
            .PostWarning( fm );

          failureReportingTransaction.Commit();

          m\_ClashingDucts.Clear();
        }
      }
      catch( System.Exception ex )
      {
        TaskDialog.Show( "Oops!", ex.Message );
      }
    }

    /// <summary>
    /// Return rule description.
    /// </summary>
    public string GetDescription()
    {
      return "Detects clash between ducts";
    }

    /// <summary>
    /// Provide the filter to focus on elements
    /// that this rule should be focused on.
    /// </summary>
    public ElementFilter GetElementFilter(
      Document document )
    {
      return new ElementClassFilter( typeof( Duct ) );
    }

    /// <summary>
    /// Return name of rule.
    /// </summary>
    public string GetName()
    {
      return "Duct Clash Detection";
    }

    /// <summary>
    /// Perform initial/preliminary work
    /// before executing the tests.
    /// </summary>
    public void InitCheck( Document document )
    {
      if( m\_ClashingDucts == null )
        m\_ClashingDucts = new List<ElementId>();
      else
        m\_ClashingDucts.Clear();

      return;
    }

    /// <summary>
    /// Return true if this rule will iterate
    /// through elements and check them,
    /// else return false.
    /// </summary>
    public bool WillCheckElements()
    {
      return true;
    }
  }
}
```

Once we have defined a class for our custom rule, our next step is to register the rule. You can do this by calling PerformanceAdvisor.AddRule.
This is done at the start-up of Revit using the OnStartup event of IExternalApplication.

Similarly, we can unregister the Performance Adviser rule at the OnShutdown event. The following code shows an example of how the custom rule is registered and unregistered:
```python
[Transaction( TransactionMode.Manual )]
public class AddPerformanceRule : IExternalApplication
{
  public DuctClashDetection ductClashDetect;

  /// <summary>
  /// Unregister the custom rule.
  /// </summary>
  public Result OnShutdown(
    UIControlledApplication application )
  {
    PerformanceAdviser adviser
      = PerformanceAdviser.GetPerformanceAdviser();

    adviser.DeleteRule( ductClashDetect.Id );

    return Result.Succeeded;
  }

  /// <summary>
  /// Register the custom rule.
  /// </summary>
  public Result OnStartup(
    UIControlledApplication application )
  {
    PerformanceAdviser adviser
      = PerformanceAdviser.GetPerformanceAdviser();

    ductClashDetect = new DuctClashDetection();

    adviser.AddRule( ductClashDetect.Id, ductClashDetect );

    return Result.Succeeded;
  }
}
```

Once the custom rule has been added, we can execute it in the same way as the predefined rules.
Using the code used in this article against a given set of clashing ducts, this new API can report the clash results as shown below:

![Collision test rule](img/performance_adviser_collision_test.jpg)

In this article, we have shown you that the performance adviser API provides the ability to define custom rules, execute them, and report potentially problematic results via the failure message APIs.
The Revit SDK PerformanceAdviserControl sample shows more comprehensive examples of this API.
It presents a list of rules for the users to select and execute.
Please refer to the sample for more details.

Many thanks to Saikat for this helpful overview!