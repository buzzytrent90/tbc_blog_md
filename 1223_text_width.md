---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.3
content_type: code_example
optimization_date: '2025-12-11T11:44:15.544708'
original_url: https://thebuildingcoder.typepad.com/blog/1223_text_width.html
post_number: '1223'
reading_time_minutes: 18
series: general
slug: text_width
source_file: 1223_text_width.htm
tags:
- csharp
- elements
- filtering
- parameters
- python
- references
- revit-api
- rooms
- selection
- sheets
- transactions
- views
- windows
title: New Text Note and Text Width Calculation
word_count: 3614
---

### New Text Note and Text Width Calculation

How can I determine the exact width of a Revit text note?

This is the topic of the Revit API discussion thread on
[textnote width calculate minimum](http://forums.autodesk.com/t5/revit-api/textnote-width-calculate-minimum/m-p/5322447).

We worked through a whole series of implementation attempts, mostly suggested by Scott Wilson, who also finally suggested the one that currently seems to be optimal and near perfect:

**Question:** When creating a new TextNote I'm given the option to give the TextNote a LineWidth Property.
I understand this is the 1:1 scale of the width and the real Width would be that number \* the scale factor of the View.
So here's the code I'm using to create the TextNote:

```csharp
  TextNoteType Bold = doc.GetElement(
    new ElementId( 1212838 ) ) as TextNoteType;

  using( Transaction t = new Transaction( doc ) )
  {
    t.Start( "Create TextNotes" );
    TextNote txNote = doc.Create.NewTextNote(
      doc.ActiveView, XYZ.Zero, XYZ.BasisX,
      XYZ.BasisY, 0.06, TextAlignFlags.TEF\_ALIGN\_LEFT
      | TextAlignFlags.TEF\_ALIGN\_BOTTOM, "TEST BOLD" );

    txNote.TextNoteType = Bold;
    t.Commit();
  }
```

This works fine so long as the string doesn't get longer.
If I need to change the length of the string (which I am) I'd need to give it a new LineWidth, or set the Width Property after I change the TextNoteType. This is important because I need to create a new TextNote after this bolded String and I want to make sure it's not too far from the last letter of the bolded string I just created.

So, is there a way to calculate the minimum Width of a TextNote without it wrapping?

**Answer:** When I used to do text rendering in game engines, I made use of Windows GDI to measure the size of text strings for a particular font. Maybe you could combine this kind of technique with other known information of the text note and view scale to get what you need?

I did a quick google and found what looks like
[the .NET way of doing this](http://msdn.microsoft.com/en-us/library/9bt8ty58(v=vs.110).aspx).

**Response:** Thanks for the suggestion, that worked out perfect in the following specific case!

I had to do some manipulation and calculations to try and figure out how tall my font in pixels and how to convert the width from pixels to inches, but I got it close enough to do a sentence (and I only need a few words) so this code works pretty good:

```csharp
  TextNoteType Bold = doc.GetElement(
    new ElementId( 1212838 ) )
      as TextNoteType; // Arial 3/32" Bold

  Font ArialBoldFont = new Font( "Arial", 9,
    FontStyle.Bold );

  using( Transaction t = new Transaction( doc ) )
  {
    t.Start( "Create TextNote" );

    string s = "TEST BOLD";

    Size txtBox = TextRenderer.MeasureText( s,
      ArialBoldFont );

    double newWidth = ( (double) txtBox.Width / 86 )
      / 12;

    TextNote txNote = doc.Create.NewTextNote(
      doc.ActiveView, XYZ.Zero, XYZ.BasisX,
      XYZ.BasisY, 0.1, TextAlignFlags.TEF\_ALIGN\_LEFT
      | TextAlignFlags.TEF\_ALIGN\_BOTTOM, s );

    txNote.TextNoteType = Bold;
    txNote.Width = newWidth;
    t.Commit();
  }
```

**Response 2:** I do not understand how you get from the 3/32" font size defined in the Revit text type to the number 9 that you specify as the em-size in points of the new .NET font.

Also, vice versa, MeasureText returns the size in pixels. Why do you divide this by 86? Would that mean 86 pixels per inch? Does that depend on the Revit scale? If it is pixels, it also depends on the device, doesn't it? And the division by 12 transforms that to feet, of course.

Can you explain, please?

Actually, trying to make general use of the solution above does not seem to work for me.

I looked at ways to improve it, making it work in my context, and would like to hear your opinion on the following expanded experiment:

It performs the following steps:

- Pick a text insertion point.
- Grab the first text note type encountered.
- Convert the text note type nominal size to points, i.e. 1/72".
- Create the appropriate .NET font.
- Measure the text size in pixels in this font.
- Determine the current display horizontal dots per inch resolution.
- Use that to convert the size in pixels to inches.
- Determine the Revit view scale.
- Scale the width in inches according to that.
- Add a factor to account for imprecision, currently the experimentally determined factor 1.4.

That seems to work for me and produces the following result:

![Text note width](img/text_width_1_1.4.png)

Here is the current version of the external command source code producing that result:

```python
  [Transaction( TransactionMode.Manual )]
  class CmdNewTextNote : IExternalCommand
  {
    [DllImport( "user32.dll" )]
    private static extern IntPtr GetDC( IntPtr hwnd );

    [DllImport( "user32.dll" )]
    private static extern Int32 ReleaseDC( IntPtr hwnd );

    /// <summary>
    /// Determine the current display
    /// horizontal dots per inch.
    /// </summary>
    static float DpiX
    {
      get
      {
        Single xDpi, yDpi;

        IntPtr dc = GetDC( IntPtr.Zero );

        using( Graphics g = Graphics.FromHdc( dc ) )
        {
          xDpi = g.DpiX;
          yDpi = g.DpiY;
        }

        if( ReleaseDC( IntPtr.Zero ) != 0 )
        {
          // GetLastError and handle...
        }
        return xDpi;
      }
    }

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIApplication uiapp = commandData.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Document doc = uidoc.Document;
      View view = doc.ActiveView;

      XYZ p;

      try
      {
        p = uidoc.Selection.PickPoint(
          "Please pick text insertion point" );
      }
      catch( Autodesk.Revit.Exceptions.OperationCanceledException )
      {
        return Result.Cancelled;
      }

      //TextNoteType boldTextType = doc.GetElement(
      //  new ElementId( 1212838 ) ) as TextNoteType; // Arial 3/32" Bold

      // 1 inch = 72 points
      // 3/32" = 72\*3/32 points =

      TextNoteType textType
        = new FilteredElementCollector( doc )
          .OfClass( typeof( TextNoteType ) )
          .FirstElement() as TextNoteType;

      Debug.Print( textType.Name );

      // 6 mm Arial happens to be the first text type found
      // 6 mm = 6 / 25.4 inch = 72 \* 6 / 25.4 points = 17 pt

      Font font = new Font( "Arial", 17, FontStyle.Bold );

      using( Transaction t = new Transaction( doc ) )
      {
        t.Start( "Create TextNote" );

        string s = "The quick brown fox jumped over the lazy dog";

        Size txtBox = System.Windows.Forms.TextRenderer
          .MeasureText( s, font );

        double w\_inch = txtBox.Width / DpiX;
        double v\_scale = view.Scale; // ratio of true model size to paper size

        Debug.Print(
          "Text box width in pixels {0} = {1} inch, scale {2}",
          txtBox.Width, w\_inch, v\_scale );

        //double newWidth
        //  = ( (double) txtBox.Width / 86 ) / 12;

        double newWidth = w\_inch / 12;

        newWidth = newWidth \* v\_scale;

        newWidth \*= 1.4;

        TextNote txNote = doc.Create.NewTextNote(
          doc.ActiveView, p, XYZ.BasisX, XYZ.BasisY,
          0.1, TextAlignFlags.TEF\_ALIGN\_LEFT
          | TextAlignFlags.TEF\_ALIGN\_BOTTOM, s );

        txNote.TextNoteType = textType;
        txNote.Width = newWidth;

        t.Commit();
      }
      return Result.Succeeded;
    }
  }
```

I would love to hear your comments on this.

I am sure it includes at least lots of room for improvement, and potentially serious errors.

I do wonder how your simple solution presented above could work as well as you say it does.

Does it really?

Thank you!

Actually, the factor 1.4 is a bit large for the lower case test sentence:

![Text note width using factor 1.4](img/text_width_1_1.4.png)

Further experiments prove that a factor of 1.3 is too small for upper case text.

It works for the lower case test sentence:

![Text note width using factor 1.3](img/text_width_2_1.3.png)

The width calculated using a factor of 1.3 for upper case words is insufficient, however, and they are wrapped:

![Upper case text note width using factor 1.3](img/text_width_3_1.3.png)

**Answer:** Fortunately, I have the luxury of only having to deal with one specific font (Arial), one specific height (3/32"), one view scale (1:1), and very short phrases, so my code does work with the crazy math. I used 9 for the TextRenderer because I was going to use 1/8" text (which, at 72DPI, is 9em), but then changed to 3/32", I just forgot to change the 9 to 6.75, which might be why I had to use 86 for my DPI. Once I got it to work I was just like, "got it, don't change anything".

I did update my code to 6.75 em and changed the division to be by 72.27 DPI, and it still gets pretty close to accurate, see attached PNG.

If you can improve the code it would be great, because I do think it would be a great help (at least until we can override characters in a TextBox to be bold like we can with the GUI).

I realize the DPI could be different on each user's screen, but wouldn't you be able to always use the same 72 DPI to calculate the width? The width of the TextNote doesn't rely on the DPI of the user's screen, so I feel like the DPI division should be a constant. Also, the Width of the TextNote will include the BuiltInParameter.LEADER\_OFFSET\_SHEET (times 2), but the TextRenderer wouldn't. You'll have to take that into account for the Width too.

I had a bit of a play with the enhanced code to see what I could discover.
I generalised it further to read in the size of the text style, width factor and border offset and then calculate the rest on the fly without hard-coding any values. I think the magic scaling factor of 1.4 is actually due to Revit treating fonts as 96 DPI (which I believe is the standard for Windows these days) instead of 72 (1.3333 factor). If I calculate the point size at 96 dpi I am able to discard the final scaling step and still get similar results.

I also played around a little with other common typographic units such as the
[pixel to millimeter converter](http://www.translatorscafe.com/cafe/EN/units-converter/typography/7-4/pixel_(X)-millimeter).

However, nothing else could explain the ~1.4 figure (I also considered that 1.4 is quite close to sqrt(2) which is a scaling factor used for ISO paper sizes, but that didn't really make sense).

The text measure method from Windows.Forms returns the size in whole pixels and therefore will be rounded. I have noticed that the variance between actual shortest line length and the calculated value is inconsistent between text sizes (sometimes the calculated value is larger, sometimes it is smaller and sometimes almost spot on), this is probably related to this integer rounding. The Graphics.MeasureString method that I originally suggested returns the width in fractional pixels without rounding, so is more consistent, but the value returned is always too large and still fluctuates a little depending on text size.

This is I believe caused by non-standard font sizes causing the text rendering system to perform its own rounding while scaling the font for drawing (and calculating string widths). I've found that if I round my calculated point size up to the nearest 8th of a point (0.125) then multiply that by 10 (then of course dividing the resulting text width by 10) to gain more resolution the end result is much more consistent.

I tested several sizes ranging from 1 mm up to 12 mm and found that a few sizes just don't behave very well (3 mm and 5 mm for example) and will calculate a length just a little too short while the rest are fine. To compensate for this I then added 2.5 points to my scaled up font size and it then worked for all sizes.

It's a pretty good length approximation that is usually just a little larger than required. There are still a few sizes that don't behave though, so there's more analysis required to pin down a proper generalised solution. I think the inconsistency comes from Revit dealing with font size point rounding and scaling in a different way to the methods we are using, sometimes the rounding is in synch, other times it will diverge.
It would be handy to know the actual method Revit uses so that it could be mimicked.

Also, be sure to apply the text type's Width Factor and then add twice the border offset defined by the BuiltInParameter.LEADER\_OFFSET\_SHEET.

**Response:** Wow, thank you very much indeed for your valuable additional research.

That is more than I could have hoped for.

It sounds like this provides the basis for a pretty complete solution.

I implemented this partially as follows:

```python
  [Transaction( TransactionMode.Manual )]
  class CmdNewTextNote : IExternalCommand
  {
    [DllImport( "user32.dll" )]
    private static extern IntPtr GetDC( IntPtr hwnd );

    [DllImport( "user32.dll" )]
    private static extern Int32 ReleaseDC( IntPtr hwnd );

    /// <summary>
    /// Determine the current display
    /// horizontal dots per inch.
    /// </summary>
    static float DpiX
    {
      get
      {
        Single xDpi, yDpi;

        IntPtr dc = GetDC( IntPtr.Zero );

        using( Graphics g = Graphics.FromHdc( dc ) )
        {
          xDpi = g.DpiX;
          yDpi = g.DpiY;
        }

        if( ReleaseDC( IntPtr.Zero ) != 0 )
        {
          // GetLastError and handle...
        }
        return xDpi;
      }
    }

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIApplication uiapp = commandData.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Document doc = uidoc.Document;
      View view = doc.ActiveView;

      XYZ p;

      try
      {
        p = uidoc.Selection.PickPoint(
          "Please pick text insertion point" );
      }
      catch( Autodesk.Revit.Exceptions.OperationCanceledException )
      {
        return Result.Cancelled;
      }

      TextNoteType textType
        = new FilteredElementCollector( doc )
          .OfClass( typeof( TextNoteType ) )
          .FirstElement() as TextNoteType;

      Debug.Print( "TextNoteType.Name = " + textType.Name );

      // 6 mm Arial happens to be the first text type found
      // 6 mm = 6 / 25.4 inch = 72 \* 6 / 25.4 points = 17 pt.
      // Nowadays, Windows does not assume that a point is
      // 1/72", but moved to 1/96" instead.

      float text\_type\_height\_mm = 6;

      float mm\_per\_inch = 25.4f;

      float points\_per\_inch = 96; // not 72

      float em\_size = points\_per\_inch
        \* ( text\_type\_height\_mm / mm\_per\_inch );

      em\_size += 2.5f;

      Font font = new Font( "Arial", em\_size,
        FontStyle.Regular );

      using( Transaction t = new Transaction( doc ) )
      {
        t.Start( "Create TextNote" );

        string s = "The quick brown fox jumps over the lazy dog";

        Size txtBox = System.Windows.Forms.TextRenderer
          .MeasureText( s, font );

        double w\_inch = txtBox.Width / DpiX;
        double v\_scale = view.Scale; // ratio of true model size to paper size

        Debug.Print(
          "Text box width in pixels {0} = {1} inch, "
          + "view scale = {2}",
          txtBox.Width, w\_inch, v\_scale );

        double newWidth = w\_inch / 12;

        TextNote txNote = doc.Create.NewTextNote(
          doc.ActiveView, p, XYZ.BasisX, XYZ.BasisY,
          newWidth, TextAlignFlags.TEF\_ALIGN\_LEFT
          | TextAlignFlags.TEF\_ALIGN\_BOTTOM, s );

        txNote.TextNoteType = textType;

        Debug.Print(
          "NewTextNote lineWidth {0} times view scale "
          + "{1} = {2} generated TextNote.Width {3}",
          Util.RealString( newWidth ),
          Util.RealString( v\_scale ),
          Util.RealString( newWidth \* v\_scale ),
          Util.RealString( txNote.Width ) );

        // This fails.

        //Debug.Assert(
        //  Util.IsEqual( newWidth \* v\_scale, txNote.Width ),
        //  "expected the NewTextNote lineWidth "
        //  + "argument to determine the resulting "
        //  + "text note width" );

        txNote.Width = newWidth \* v\_scale;

        t.Commit();
      }
      return Result.Succeeded;
    }
  }
```

Does that match your intentions and explanation, Scott?

I do not see where to obtain the "text type's Width Factor" that you mention. Where is that stored?

Furthermore, what element hosts the BuiltInParameter.LEADER\_OFFSET\_SHEET?

It sounds like you tested this for various different fonts and text string samples.

Have you written some kind of test framework iterating over the available text note types, matching them with .NET fonts and trying them out with different strings?

In order to use this approach in a serious production environment, I would definitely set up and use such a system and a corresponding unit test to verify that the chosen adjustments really do work correctly in all cases encountered.

**Answer:** That looks pretty close Jeremy.
One extra thing my solution does is scale the font point size up by 10x then divides the resulting string width by 10 afterwards so that we can get sub-pixel accuracy out of the MeasureText method. I do this upscaling before adding the 2.5 points (magic number unfortunately...).

The command I developed is based upon your code, but iterates through all available textnote types calculating the font metrics and placing a note for each just below the preceding one. It works fine for each note type in my testing template (ranging from 1 through to 12mm), none of them wrap, but some sizes do have a little more line length than I would like. I'll clean it up a little to remove the references to my framework and post it up.

BuiltInParameter.LEADER\_OFFSET\_SHEET is the Leader/Border offset parameter found in the TextNoteType element. The font height and width factor can also be found there. One thing my code doesn't do just yet is handle font styles such as bold and italics; it'll only take 5 min to throw that in before I post it up though.

Ok...

After a little more testing I've decided that using TextRenderer.MeasureText() is not the way to go.
The results are too inconsistent and require too much hand-waving to normalise.
I believe that TextRenderer.MeasureText() uses standard GDI to calculate the text dimensions while Graphics.MeasureString() makes use of GDI+. I would think that Revit would itself be using GDI+, so it would make sense to use the same system for our calculations.

I stripped away all of the scaling and magic numbers and used Graphics.MeasureString().
I found that although the line lengths do end up a little larger than you would like, the results are much more stable (although it isn't perfect, I was able to cause a line wrap with an Arial bold, italic, underlined 16.5mm font). I also stripped out the User32 Interop stuff as it is quite easy to get at the same data using managed code.

Anyway here is the full external command class for you to play with, I won't go as far as to say it is a complete solution as there are still a few instances where it will cause a line wrap. I just think that Revit is doing some odd things internally with regard to calculation of line wrapping and we'll never know for sure until it is one day (hopefully) exposed to the API.

```csharp
  static float GetDpiX()
  {
    float xDpi, yDpi;

    using( Graphics g = Graphics.FromHwnd( IntPtr.Zero ) )
    {
      xDpi = g.DpiX;
      yDpi = g.DpiY;
    }
    return xDpi;
  }

  static double GetStringWidth( string text, Font font )
  {
    double textWidth = 0.0;

    using( Graphics g = Graphics.FromHwnd( IntPtr.Zero ) )
    {
      textWidth = g.MeasureString( text, font ).Width;
    }
    return textWidth;
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    Result commandResult = Result.Succeeded;

    try
    {
      UIApplication uiApp = commandData.Application;
      UIDocument uiDoc = uiApp.ActiveUIDocument;
      Document dbDoc = uiDoc.Document;
      View view = uiDoc.ActiveGraphicalView;

      XYZ pLoc = XYZ.Zero;

      try
      {
        pLoc = uiDoc.Selection.PickPoint(
          "Please pick text insertion point" );
      }
      catch( Autodesk.Revit.Exceptions.OperationCanceledException )
      {
        Debug.WriteLine( "Operation cancelled." );
        message = "Operation cancelled.";

        return Result.Succeeded;
      }

      List<TextNoteType> noteTypeList
        = new FilteredElementCollector( dbDoc )
          .OfClass( typeof( TextNoteType ) )
          .Cast<TextNoteType>()
          .ToList();

      // Sort note types into ascending text size

      BuiltInParameter bipTextSize
        = BuiltInParameter.TEXT\_SIZE;

      noteTypeList.Sort( ( a, b )
        => a.get\_Parameter( bipTextSize ).AsDouble()
          .CompareTo(
            b.get\_Parameter( bipTextSize ).AsDouble() ) );

      foreach( TextNoteType textType in noteTypeList )
      {
        Debug.WriteLine( textType.Name );

        Parameter paramTextFont
          = textType.get\_Parameter(
            BuiltInParameter.TEXT\_FONT );

        Parameter paramTextSize
          = textType.get\_Parameter(
            BuiltInParameter.TEXT\_SIZE );

        Parameter paramBorderSize
          = textType.get\_Parameter(
            BuiltInParameter.LEADER\_OFFSET\_SHEET );

        Parameter paramTextBold
          = textType.get\_Parameter(
            BuiltInParameter.TEXT\_STYLE\_BOLD );

        Parameter paramTextItalic
          = textType.get\_Parameter(
            BuiltInParameter.TEXT\_STYLE\_ITALIC );

        Parameter paramTextUnderline
          = textType.get\_Parameter(
            BuiltInParameter.TEXT\_STYLE\_UNDERLINE );

        Parameter paramTextWidthScale
          = textType.get\_Parameter(
            BuiltInParameter.TEXT\_WIDTH\_SCALE );

        string fontName = paramTextFont.AsString();

        double textHeight = paramTextSize.AsDouble();

        bool textBold = paramTextBold.AsInteger() == 1
          ? true : false;

        bool textItalic = paramTextItalic.AsInteger() == 1
          ? true : false;

        bool textUnderline = paramTextUnderline.AsInteger() == 1
          ? true : false;

        double textBorder = paramBorderSize.AsDouble();

        double textWidthScale = paramTextWidthScale.AsDouble();

        FontStyle textStyle = FontStyle.Regular;

        if( textBold )
        {
          textStyle |= FontStyle.Bold;
        }

        if( textItalic )
        {
          textStyle |= FontStyle.Italic;
        }

        if( textUnderline )
        {
          textStyle |= FontStyle.Underline;
        }

        float fontHeightInch = (float) textHeight \* 12.0f;
        float displayDpiX = GetDpiX();

        float fontDpi = 96.0f;
        float pointSize = (float) ( textHeight \* 12.0 \* fontDpi );

        Font font = new Font( fontName, pointSize, textStyle );

        int viewScale = view.Scale;

        using( Transaction t = new Transaction( dbDoc ) )
        {
          t.Start( "Test TextNote lineWidth calculation" );

          string textString = textType.Name
            + " (" + fontName + " "
            + ( textHeight \* 304.8 ).ToString( "0.##" ) + "mm, "
            + textStyle.ToString() + ", "
            + ( textWidthScale \* 100.0 ).ToString( "0.##" )
            + "%): The quick brown fox jumps over the lazy dog.";

          double stringWidthPx = GetStringWidth( textString, font );

          double stringWidthIn = stringWidthPx / displayDpiX;

          Debug.WriteLine( "String Width in pixels: "
            + stringWidthPx.ToString( "F3" ) );
          Debug.WriteLine( ( stringWidthIn \* 25.4 \* viewScale ).ToString( "F3" )
            + " mm at 1:" + viewScale.ToString() );

          double stringWidthFt = stringWidthIn / 12.0;

          double lineWidth = ( ( stringWidthFt \* textWidthScale )
            + ( textBorder \* 2.0 ) ) \* viewScale;

          TextNote textNote = dbDoc.Create.NewTextNote(
            view, pLoc, XYZ.BasisX, XYZ.BasisY, 0.001,
            TextAlignFlags.TEF\_ALIGN\_LEFT
            | TextAlignFlags.TEF\_ALIGN\_TOP, textString );

          textNote.TextNoteType = textType;
          textNote.Width = lineWidth;

          t.Commit();
        }

        // Place next text note below this one with 5 mm gap

        pLoc += view.UpDirection.Multiply(
          ( textHeight + ( 5.0 / 304.8 ) )
            \* viewScale ).Negate();
      }
    }
    catch( Autodesk.Revit.Exceptions.ExternalApplicationException e )
    {
      message = e.Message;
      Debug.WriteLine( "Exception Encountered (Application)\n"
        + e.Message + "\nStack Trace: " + e.StackTrace );

      commandResult = Result.Failed;
    }
    catch( Exception e )
    {
      message = e.Message;
      Debug.WriteLine( "Exception Encountered (General)\n"
        + e.Message + "\nStack Trace: " + e.StackTrace );

      commandResult = Result.Failed;
    }
    return commandResult;
  }
```

This provides good results.
Please test it out with various combinations of textnote styles with width factors like 0.75 and you'll see that it does sometimes fail at larger text heights, e.g. > 16mm.
Overall I'm happy with it, though, and will be adding it to my framework as a basis for several commands I've been meaning to add for a while now.

**Response:** Your new implementation based on Graphics.MeasureString instead of TextRenderer.MeasureText works perfectly for me in all the simple standard cases:

![Text note width using Graphics.MeasureString](img/text_width_4_iterate.png)

I added this as a new external command CmdNewTextNote to
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) published
on GitHub.

Your current code is tagged as
[release 2015.0.114.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.114.2).

Thank you!