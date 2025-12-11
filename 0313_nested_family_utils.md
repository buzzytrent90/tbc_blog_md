---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.729067'
original_url: https://thebuildingcoder.typepad.com/blog/0313_nested_family_utils.html
post_number: '0313'
reading_time_minutes: 8
series: family
slug: nested_family_utils
source_file: 0313_nested_family_utils.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- references
- revit-api
- views
title: Nested Family Utility Methods
word_count: 1649
---

### Nested Family Utility Methods

Nested families seem to be a recurring topic right now.
We just recently discussed how to test whether a family instance in a project file is
[nested inside another](http://thebuildingcoder.typepad.com/blog/2010/02/nested-family-instance.html),
and before that we addressed
[nested instance geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html) and
[creating a nested family](http://thebuildingcoder.typepad.com/blog/2009/11/nested-family.html).
The first place to look for information on using the Revit API in the context of families is the
[overview of the family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html)
with the associated
[labs](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html#4) and the
[materials and recording](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html#5) of the webcast we presented on the topic.

I received a couple of useful helper methods for exploring and working in the family context from an anonymous contributor whom we can call SOA-boy, since he already contributed some hints on using
[Service-Oriented Architecture](http://thebuildingcoder.typepad.com/blog/2009/08/serviceoriented-architecture.html). Here is his description:

Because the normal filtering mechanism apparently doesn't work for detecting nested family definitions or nested family instances within a family document, I wrote some code for iterating over all the elements in the host family document as quickly as possible.
The code is intended to be highly reusable.

Another issue addressed is to get to a nested family's parameter, because you have to call different methods depending on whether or not that parameter is a shared parameter or a family one.
The code to do that turns out to be simple, just non-intuitive, until you think about it some more.
Maybe that's a more familiar approach to programmers of the project editor API, but if you work more in the family editor API, it wasn't intuitive at all.
Methods are provided for getting a reference to a nested family's parameter, and to allow you to link a nested family's parameter to a FamilyParameter on the host family.

I implemented a new Building Coder sample module CmdNestedFamilies.cs which includes these methods in the class NestedFamilyFunctions.
The methods provided include:

- GetFilteredNestedFamilyDefinitions:
  Returns a list of the nested family files in the given family document whose name
  matches the given family file name filter. Useful for checking to see if a family
  desired for nesting into the host family document is already nested in.- GetFilteredNestedFamilyInstances:
    Returns a list of family instances found in the given family document whose family file
    name matches the given familyFileNameFilter and whose type name matches the given
    typeNameFilter. If no filter values are provided (or they evaluate to the empty string
    when trimmed) then all instances will be evaluated.- GetFamilyParameter:
      Returns a reference to the FAMILY parameter (as a simple Parameter data type) on the given instance
      for the parameter with the given name. Will return the parameter
      whether it is an instance or type parameter.- LinkNestedFamilyParameterToHostFamilyParameter:
        This method takes an instance of a nested family and links a parameter on it to
        a parameter on the given host family instance. This allows a change at the host
        level to automatically be sent down and applied to the nested family instance.

The motivation for the first method is that standard Revit filtering techniques fail when searching for nested families in a family document, so we have no choice but to iterate over all elements in the family.
While there usually aren't that many elements at the family level, nonetheless this method has been built for speed.
It returns collection of family file definitions nested into the given family document.

The methods are packaged in a utility class NestedFamilyFunctions.
It also includes two helper methods FilterMatches and ValidateFamilyDocument:

- FilterMatches:
  Returns whether or not the nameToCheck matches the given filter.
  This is done with a simple Contains check, so wildcards won't work.- ValidateFamilyDocument:
    This method will validate the provided Revit Document to make sure the reference
    exists and is for a FAMILY document. It will throw an ArgumentNullException
    if nothing is sent, and will throw an ArgumentOutOfRangeException if the document
    provided isn't a family document (e.g. is a project document)

Here is the NestedFamilyFunctions source code, including some comments describing what is going on and why.
Following good SOA practices, the methods verify that the incoming data can be worked with.
Since some lines are too long to display here in this post, you may have to either copy and paste to an editor or download the full source code to see them in full length:
```csharp
#region Public Methods
public static List<Family>
  GetFilteredNestedFamilyDefinitions(
    string familyFileNameFilter,
    Document familyDocument,
    bool caseSensitiveFiltering )
{
  ValidateFamilyDocument( familyDocument );

  // The filter can be null, the filter matching function checks for that.

  List<Family> oResult = new List<Family>();

  ElementIterator it = familyDocument.Elements;

  while( it.MoveNext() )
  {
    Element oElement = it.Current as Element;

    if( ( oElement is Family )
      && FilterMatches( oElement.Name,
        familyFileNameFilter, caseSensitiveFiltering ) )
    {
      oResult.Add( oElement as Family );
    }
  }
  return oResult;
}

public static List<FamilyInstance>
  GetFilteredNestedFamilyInstances(
    string familyFileNameFilter,
    string typeNameFilter,
    Document familyDocument,
    bool caseSensitiveFiltering )
{
  ValidateFamilyDocument( familyDocument );

  // The filters can be null

  List<FamilyInstance> oResult
    = new List<FamilyInstance>();

  Family oNestedFamilyFileCandidate;
  FamilyInstance oFamilyInstanceCandidate;
  FamilySymbol oFamilySymbolCandidate;

  List<Family> oMatchingNestedFamilies
    = new List<Family>();

  List<FamilyInstance> oAllFamilyInstances
    = new List<FamilyInstance>();

  bool bFamilyFileNameFilterExists = true;
  bool bTypeNameFilterExists = true;

  // Set up some fast-to-test boolean values, which will be
  // used for short-circuit Boolean evaluation later.

  if( string.IsNullOrEmpty( familyFileNameFilter ) )
  {
    bFamilyFileNameFilterExists = false;
  }

  if( string.IsNullOrEmpty( typeNameFilter ) )
  {
    bTypeNameFilterExists = false;
  }

  // Unfortunately detecting nested families in a family document requires iterating
  // over all the elements in the document, because the built-in filtering mechanism
  // doesn't work for this case.  However, families typically don't have nearly as many
  // elements as a whole project, so the performance hit shouldn't be too bad.
  //
  // Still, the fastest performance should come by iterating over all elements in the given
  // family document exactly once, keeping subsets of the family instances found for
  // later testing against the nested family file matches found.

  ElementIterator oElementIterator
    = familyDocument.Elements;

  while( oElementIterator.MoveNext() )
  {
    // See if this is a family file nested into the current family document.
    oNestedFamilyFileCandidate
      = oElementIterator.Current as Family;

    if( oNestedFamilyFileCandidate != null )
    {
      // Must ask the "Element" version for it's name, because the Family object's
      // name is always the empty string.
      if( !bFamilyFileNameFilterExists
        || FilterMatches( oNestedFamilyFileCandidate.Name,
          familyFileNameFilter, caseSensitiveFiltering ) )
      {
        // This is a nested family file, and either no valid family file name filter was
        // given, or the name of this family file matches the filter.

        oMatchingNestedFamilies.Add(
          oNestedFamilyFileCandidate );
      }
    }
    else
    {
      // This element is not a nested family file definition, see if it's a
      // nested family instance.

      oFamilyInstanceCandidate
        = oElementIterator.Current as FamilyInstance;

      if( oFamilyInstanceCandidate != null )
      {
        // Just add the family instance to our "all" collection for later testing
        // because we may not have yet found all the matching nested family file
        // definitions.
        oAllFamilyInstances.Add(
          oFamilyInstanceCandidate );
      }
    }

  } // End iterating over all the elements in the family document exactly once

  // See if any matching nested family file definitions were found.  Only do any
  // more work if at least one was found.
  foreach( Family oMatchingNestedFamilyFile
    in oMatchingNestedFamilies )
  {
    // Count backwards through the all family instances list.  As we find
    // matches on this iteration through the matching nested families, we can
    // delete them from the candidates list to reduce the number of family
    // instance candidates to test for later matching nested family files to be tested
    for( int iCounter = oAllFamilyInstances.Count - 1;
      iCounter >= 0; iCounter-- )
    {
      oFamilyInstanceCandidate
        = oAllFamilyInstances[iCounter];

      oFamilySymbolCandidate
        = oFamilyInstanceCandidate.ObjectType
          as FamilySymbol;

      if( oFamilySymbolCandidate.Family.UniqueId
        == oMatchingNestedFamilyFile.UniqueId )
      {
        // Only add this family instance to the results if there was no type name
        // filter, or this family instance's type matches the given filter.

        if( !bTypeNameFilterExists
          || FilterMatches( oFamilyInstanceCandidate.Name,
            typeNameFilter, caseSensitiveFiltering ) )
        {
          oResult.Add( oFamilyInstanceCandidate );
        }

        // No point in testing this one again,
        // since we know its family definition
        // has already been processed.

        oAllFamilyInstances.RemoveAt( iCounter );
      }

    } // Next family instance candidate

  } // End of for each matching nested family file definition found

  return oResult;
}

public static Parameter GetFamilyParameter(
  FamilyInstance nestedFamilyInstance,
  string parameterName )
{
  if( nestedFamilyInstance == null )
  {
    throw new ArgumentNullException(
      "nestedFamilyInstance" );
  }

  if( string.IsNullOrEmpty( parameterName ) )
  {
    throw new ArgumentNullException(
      "parameterName" );
  }

  Parameter oResult = null;

  //See if the parameter is an Instance parameter
  oResult = nestedFamilyInstance.get\_Parameter(
    parameterName );

  // No?  See if it's a Type parameter
  if( oResult == null )
  {
    oResult = nestedFamilyInstance.Symbol.get\_Parameter(
      parameterName );
  }
  return oResult;
}

public static void
  LinkNestedFamilyParameterToHostFamilyParameter(
    Document hostFamilyDocument,
    FamilyInstance nestedFamilyInstance,
    string nestedFamilyParameterName,
    string hostFamilyParameterNameToLink )
{
  ValidateFamilyDocument( hostFamilyDocument );

  if( nestedFamilyInstance == null )
  {
    throw new ArgumentNullException(
      "nestedFamilyInstance" );
  }

  if( string.IsNullOrEmpty( nestedFamilyParameterName ) )
  {
    throw new ArgumentNullException(
      "nestedFamilyParameterName" );
  }

  if( string.IsNullOrEmpty( hostFamilyParameterNameToLink ) )
  {
    throw new ArgumentNullException(
      "hostFamilyParameterNameToLink" );
  }

  Parameter oNestedFamilyParameter
    = GetFamilyParameter( nestedFamilyInstance,
      nestedFamilyParameterName );

  if( oNestedFamilyParameter == null )
  {
    throw new Exception( "Parameter '"
      + nestedFamilyParameterName
      + "' was not found on the nested family '"
      + nestedFamilyInstance.Symbol.Name + "'" );
  }

  FamilyParameter oHostFamilyParameter
    = hostFamilyDocument.FamilyManager.get\_Parameter(
      hostFamilyParameterNameToLink );

  if( oHostFamilyParameter == null )
  {
    throw new Exception( "Parameter '"
      + hostFamilyParameterNameToLink
      + "' was not found on the host family." );
  }

  hostFamilyDocument.FamilyManager
    .AssociateElementParameterToFamilyParameter(
      oNestedFamilyParameter, oHostFamilyParameter );
}
#endregion Public Methods

#region Private / Helper Methods
private static bool FilterMatches(
  string nameToCheck,
  string filter,
  bool caseSensitiveComparison )
{
  bool bResult = false;

  if( string.IsNullOrEmpty( nameToCheck ) )
  {
    // No name given, so the call must fail.
    return false;
  }

  if( string.IsNullOrEmpty( filter ) )
  {
    // No filter given, so the given name passes the test
    return true;
  }

  if( !caseSensitiveComparison )
  {
    // Since the String.Contains function only does case-sensitive checks,
    // cheat with our copies of the values which we'll use for the comparison.
    nameToCheck = nameToCheck.ToUpper();
    filter = filter.ToUpper();
  }

  bResult = nameToCheck.Contains( filter );

  return bResult;
}

private static void ValidateFamilyDocument(
  Document document )
{
  if( null == document )
  {
    throw new ArgumentNullException( "document" );
  }

  if( !document.IsFamilyDocument )
  {
    throw new ArgumentOutOfRangeException(
      "The document provided is not a Family Document." );
  }
}
#endregion Private / Helper Methods
```

Here is
[version 1.1.0.63](zip/bc11063.zip)
of the complete Building Coder source code and Visual Studio solution including the new external command CmdNestedFamilies and the NestedFamilyFunctions utility class.

Many thanks to SOA-boy for this contribution!