---
post_number: "1137"
title: "Migrating RoomEditorApp to Revit 2015"
slug: "roomeditorapp_2015"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'python', 'references', 'revit-api', 'rooms', 'selection', 'sheets', 'views', 'windows']
source_file: "1137_roomeditorapp_2015.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1137_roomeditorapp_2015.html"
---

### Migrating RoomEditorApp to Revit 2015

Yesterday I discussed the typical steps you would need to go through to
[set up a Revit 2015 add-in development environment](http://thebuildingcoder.typepad.com/blog/2014/04/compiling-the-revit-2015-sdk-and-migrating-bc-samples.html).

My top priority right now is getting my Tech Summit talk prepared, which involves adding some functionality to the room editor application add-in.

I decided to migrate it to Revit 2015 as well.

Here are the steps I took:

- [Flat RoomEditorApp Migration](#2)
- [Selection.Elements is Obsolete](#3)
- [ViewSheet.Views is Obsolete](#4)
- [Enabling CopySourceAsHtml for Visual Studio 2012](#5)
- [Download](#6)

#### Flat RoomEditorApp Migration

Just like
[The Building Coder sample migration](http://thebuildingcoder.typepad.com/blog/2014/04/compiling-the-revit-2015-sdk-and-migrating-bc-samples.html#5) yesterday,
the flat migration proved very easy.

I repeated the same steps:

- Update the Revit API assembly references.
- Change the .NET framework from 4.0 to 4.5.
- Updated the version number.
- Rebuild all.

That was it.

The flat migration compiles with zero errors.

The resulting add-in works fine.

This probably actually means that the Revit 2014 add-in would work fine in Revit 2015 as well.

The compilation does produce
[five warnings](zip/roomeditorapp_migr_2015_a.txt) about deprecated API usage; five occurrences, actually, but only two distinct warnings, to be exact:

- [Autodesk.Revit.UI.Selection.Selection.Elements is obsolete](#3): This property is deprecated in Revit 2015. Use GetElementIds and SetElementIds instead.
- [Autodesk.Revit.DB.ViewSheet.Views is obsolete](#4): This property is obsolete in Revit 2015. Use GetAllPlacedViews instead.

Let's take a look at those and work towards a completely clean build.

#### Selection.Elements is Obsolete

The external command causing the first warning is CmdUploadRooms, where the following code is used to check for pre-selected rooms:

```csharp
  List<ElementId> ids = null;

  Selection sel = uidoc.Selection;

  if( 0 < sel.Elements.Size )
  {
    foreach( Element e in sel.Elements )
    {
      if( !( e is Room ) )
      {
        Util.ErrorMsg( "Please pre-select only room"
          + " elements before running this command." );
        return Result.Failed;
      }

      if( null == ids )
      {
        ids = new List<ElementId>( 1 );
      }

      ids.Add( e.Id );
    }
  }
```

Removing that warning is simple, and I can even shorten the code, like this:

```csharp
  Selection sel = uidoc.Selection;

  ICollection<ElementId> ids = sel.GetElementIds();

  if( 0 < ids.Count )
  {
    foreach( ElementId id in ids )
    {
      if( !( doc.GetElement( id ) is Room ) )
      {
        Util.ErrorMsg( "Please pre-select only room"
          + " elements before running this command." );
        return Result.Failed;
      }
    }
  }
```

#### ViewSheet.Views is Obsolete

The second warning is also trivial to fix.

One of the occurrences is this simple loop:

```csharp
  foreach( View v in sheet.Views )
  {
    GetViewTransform( v );
  }
```

The obsolete Views property returns an obsolete ViewSet class instance.

The replacement method GetAllPlacedViews returns a set of element ids instead, in the shape of a generic set `ISet<ElementId>`.

You can use the generic LINQ Select method to convert the element ids straight to View object instances like this, if you like:

```csharp
  foreach( View v in sheet.GetAllPlacedViews()
    .Select<ElementId, View>( id =>
      doc.GetElement( id ) as View ) )
  {
    GetViewTransform( v );
  }
```

Similar conversions resolve the other two warnings concerning this property.

#### Enabling CopySourceAsHtml for Visual Studio 2012

As you see, I presented some syntax coloured C# source code above, copied and pasted from Visual Studio 2012.

I am still using the J.T. Leigh
[CopySourceAsHtml](http://copysourceashtml.codeplex.com) utility
as described in
[The Building Coder source code colour coder](http://thebuildingcoder.typepad.com/blog/2011/04/updated-sdk-2012-products-and-source-code-colourisation.html#4)
and my
[pycolorize.py Python script](zip/pycolorize2014.py) to
strip some extraneous baggage from the HTML code before inserting it into the blog text.

Luckily, the description of how to
[use CopySourceAsHtml for Visual Studio 2010](http://blogs.microsoft.co.il/applisec/2010/02/25/copyashtml-in-visual-studio-2010) also
applies to Visual Studio 2012, more or less, as I found out by trial and error.

I now have three versions of the CopySourceAsHtml add-in file located in the following folders for Visual Studio 2008, 2010 and 2012:

- C:\Users\tammikj\Documents\Visual Studio 2008\Addins\CopySourceAsHtml.AddIn
- C:\Users\tammikj\Documents\Visual Studio 2010\Addins\CopySourceAsHtml.AddIn
- C:\Users\tammikj\Documents\Visual Studio 2012\Addins\CopySourceAsHtml.AddIn

The add-in file that I am using for Visual Studio 2012 looks like this:

```csharp
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<Extensibility xmlns=
  "http://schemas.microsoft.com/AutomationExtensibility">
  <HostApplication>
    <Name>Microsoft Visual Studio Macros</Name>
    <Version>11.0</Version>
  </HostApplication>
  <HostApplication>
    <Name>Microsoft Visual Studio</Name>
    <Version>11.0</Version>
  </HostApplication>
  <Addin>
    <FriendlyName>CopySourceAsHtml</FriendlyName>
    <Description>
      Adds support to Microsoft Visual Studio 2012
      for copying source code, syntax highlighting,
      and line numbers as HTML.
    </Description>
    <Assembly>
      JTLeigh.Tools.Development.CopySourceAsHtml,
      Version=3.0.3215.1, Culture=neutral,
      PublicKeyToken=bb2a58bdc03d2e14,
      processorArchitecture=MSIL
    </Assembly>
    <FullClassName>
      JTLeigh.Tools.Development.CopySourceAsHtml.Connect
    </FullClassName>
    <LoadBehavior>1</LoadBehavior>
    <CommandPreload>0</CommandPreload>
    <CommandLineSafe>0</CommandLineSafe>
  </Addin>
</Extensibility>
```

For the sake of completeness, the current version of my Python clean-up script looks like this:

```
#!/usr/bin/python
# -*- coding: iso-8859-15 -*-
#
# pycolorize.py - massage colorised HTML source code copied from Visual Studio
#
# jeremy tammik, autodesk inc, 2009-02-05
#
# History:
#
# 2009-02-05 initial version
# 2009-05-22 updated to support Visual Studio 2008
# 2011-04-19 updated to support Visual Studio 2010 and CopySourceAsHtml 3.0
# 2013-05-21 migrated to mac os x unix, cf. http://coffeeghost.net/src/pyperclip.py
#
# read a block of text from a file or the windows clipboard
# replace cb[12345] by the appropriate colour
# remove the style and pre tags
#
import os, re

color_map = { '#2b91af' : 'teal', '#a31515' : 'maroon' }

def getTextMac():
  outf = os.popen('pbpaste', 'r')
  content = outf.read()
  outf.close()
  return content

def setTextMac(text):
  outf = os.popen('pbcopy', 'w')
  outf.write(text)
  outf.close()

def getTextWin():
  w.OpenClipboard()
  d = w.GetClipboardData( win32con.CF_TEXT )
  w.CloseClipboard()
  return d

def setTextWin( aType, aString ):
  w.OpenClipboard()
  w.EmptyClipboard()
  w.SetClipboardData( aType, aString )
  w.CloseClipboard()

_regexColor = re.compile( '\.(cb[1-9]) \{ color\: ([#0-9a-z]+); \}' )
_regexStyle = re.compile( '(<style type="text/css">.*</style>\s*<div class="cf">\s*)', re.DOTALL )
_regexEnd = re.compile( '(</pre>\s*</div>)', re.DOTALL )
#_regexPreEnd = re.compile( '(</pre>$)' )

def replace_cb_by_color( s ):
  "Search for '.cb1 { color: blue; }' and globally replace cb[1-9] by the appropriate colour."
  m = _regexColor.search( s )
  if m:
    #print 'match found'
    a = m.groups()
    #print a
    if 2 == len( a ):
      color = a[1]
      #print color
      if color_map.has_key( color ): color = color_map[color]
      #print color
      return True, s.replace( a[0], color )
  #else:
  #  print 'no match found'

  return False, s

def main():
  'Convert Visual Studio CopySourceAsHtml colour styles to a more compact form.'

  s = getTextMac()
  #s = '''<style type="text/css"> ... '''
  #print s

  go = True
  while go: go, s = replace_cb_by_color( s )
  #print s

  m = _regexStyle.match( s )
  #print m

  if m:
    s = s.replace( m.group( 1 ), '' )
    #print s
    s = s.replace( '<pre class="cl">', '' )
    #print s
    m = _regexEnd.search( s )
    #print m

  if m:
    s = s.replace( m.group( 1 ), '' )
    #print s
    s = s.strip().replace( '</pre>', '' )
    #print s

  setTextMac( s )
  #print s

if __name__ == '__main__':
  main()
```

#### Download

The RoomEditorApp source code, Visual Studio solution and add-in manifest live in the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp).

The versions discussed above are
[release 2015.0.2.6](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2015.0.2.6) for
the flat migration and
[release 2015.0.2.7](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2015.0.2.7) with
the deprecated API usage cleaned up.

Next step: finally get going with the Tech Summit preparation!