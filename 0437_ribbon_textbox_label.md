---
post_number: "0437"
title: "Simulating a Ribbon Textbox Label"
slug: "ribbon_textbox_label"
author: "Jeremy Tammik"
tags: ['csharp', 'levels', 'revit-api']
source_file: "0437_ribbon_textbox_label.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0437_ribbon_textbox_label.html"
---

### Simulating a Ribbon Textbox Label

On Wednesday evening I arrived in Stavanger on the coast of Norway, found the sailship
[Pantagruel](http://www.classicsailing.de) and met the crew that I will be spending the coming week with.

Thursday we went on a hike rather than sailing, because we happened to be so close to the spectacular
[Preikestolen or Pulpit Stone](http://en.wikipedia.org/wiki/Preikestolen) over
the Lysefjord.
We took our boat over from Stavanger to Jørpeland by motor, since the wind was exactly opposite our direction, and then spent all afternoon hiking through the incredibly beautiful moors and rocks.

On Friday we started the real sailing tour, motoring out of the Stavanger bay in sunshine and very little wind, and catching more wind outside the coast to sail down towards Hidra.
The wind grew and the waves as well, and all of us newbies got a thorough dose of seasickness.
We arrived in a beautiful bay late in the evening and spent most of Saturday recuperating in tranquillity and sunshine.
I also spend a couple of hours trying my hand at sailing a miniature boat, a jolle, before continuing down to Egersund for shopping.

Tonight we are planning to sail further south during the night.
The final goal is to cross over to Denmark and from there cross the Skagerrak to Sweden, then sail on the west coast until we get to Göteborg and head back home.

We have not yet talked at all about the new ribbon capabilities in the Revit 2011 API.
There are quite a few, as the following excerpt from the What's New section of the Revit API help file RevitAPI.chm shows:

#### Additional options for Ribbon customization

There are new Ribbon components supported via the Ribbon API:

- SplitButton – a pulldown button with a default pushbutton attached- RadioGroup – a set of ToggleButtons, where only one of the set can be active at a time- ComboBox – a pulldown containing a set of selectable items, which can be grouped optionally- TextBox – an input field for users to enter text- SlideOut panel – an extra panel can optionally slide down from a Ribbon panel; this panel can contain any of the standard Ribbon components

For ComboBox and TextBox, events are included; these events call your API code when the component is changed by the user.

The new property

- RibbonItem.Visible

provides control over whether a particular item is visible.

The new properties:

- RibbonItem.LongDescription- RibbonItem.ToolTipImage

allow you to set up an extended tooltip for the Ribbon item. This tooltip can display a longer set of text, and/or a single image.

The new property:

- PushButton.AvailabilityClassName

allows assignment of an availability class to controlled whether or not the button is available, similar to the option provided for ExternalCommands registered by manifest.

There is also a new option supported for PulldownButton – a separator can now be added between buttons this component.

There is also a new option to add custom panels to the Analyze tab in Revit as well as the Add-Ins tab, via a new overload of Application.CreateRibbonPanel().

As a result of these enhancements, some pre-existing APIs have changed:

- RibbonPanel.AddButton() has been replaced with RibbonPanel.AddItem()- RibbonPanel.AddStackedButtons() overloads have been replaced with RibbonPanel.AddStackedItems() overloads- Property RibbonPanel.Items has been replaced with method RibbonPanel.GetItems()- Property PulldownButton.Items has been replaced with method PulldownButton.GetItems()- Methods RibbonPanel.AddPushButton() and RibbonPanel.AddPulldownButton() have been removed. Use RibbonPanel.AddItem() for this operation.- RibbonPanel.AddToPulldown() has been removed. Use PulldownButton.AddItem() for this operation.- PulldownButton.AddPushButton() has been removed. Use PulldownButton.AddItem() for this operation.

Some of the new functionality is demonstrated by the Ribbon SDK sample, but not all.

Here is a question on how to simulate a missing widget, a label to describe the use of a text box:

**Question:** How can I add a label to a ribbon text box like in these two examples from the Revit user interface?

Here is a label for the active workset:

![Active workset label](img/ribbon_textbox_label_active_workset.jpg)

Here is a different label in the Autodesk Seek panel:

![Autodesk Seek panel](img/ribbon_textbox_label_seek.png)

Could I possible use some stacked items to achieve this?

This is what my attempts so far end up looking like, displaying a textbox lacking a label:
![Ribbon textbox lacking label](img/ribbon_textbox_label_attempt.png)

The code I am using to produce it looks like this:
```vbnet
Sub AddTextBox(ByVal panel As RibbonPanel)

  ' fill the text gox information
  Dim txtBoxData As New TextBoxData("TextBox")

  txtBoxData.Image = New BitmapImage( \_
    New Uri(m\_imageFolder + "Basics.ico"))

  txtBoxData.Name = "Text Box"
  txtBoxData.ToolTip = "Enter text here"
  txtBoxData.LongDescription \_
    = "<p>This is Revit UI Labs.</p><p>Ribbon Lab</p>"

  txtBoxData.ToolTipImage = New BitmapImage( \_
    New Uri(m\_imageFolder + "ImgHelloWorld.png"))

  ' create the text box item on the panel
  Dim txtBox As TextBox = panel.AddItem(txtBoxData)
  txtBox.PromptText = "Enter a comment"
  txtBox.ShowImageAsButton = True
  txtBox.Width = 180
  'txtBox.ItemText = "my text box"
  'txtBox.Name ' this is read only.

  ' p51. we'll talk about events in Lab4
  AddHandler txtBox.EnterPressed, New EventHandler( \_
    Of TextBoxEnterPressedEventArgs)( \_
    AddressOf txtBox\_EnterPressed)

End Sub

' event hander for the text box above.
Sub txtBox\_EnterPressed( \_
  ByVal sender As Object, \_
  ByVal e As TextBoxEnterPressedEventArgs)

  ' cast sender as TextBox to retrieve text value
  Dim txtBox As TextBox = sender

  TaskDialog.Show("TextBox Input", \_
    "This is what you typed in: " \_
    + txtBox.Value.ToString())

End Sub
```

**Answer:** The ribbon API does not contain a text label item, so the best alternative might be a disabled pushbutton stacked above the other control.
Here is an example of what this might look like:

![Simulate textbox label with disabled pushbutton](img/ribbon_textbox_label_disabled_pushbutton.jpg)

Here is the tweaked excerpt from the Ribbon SDK sample that produces this result:
```csharp
  PushButtonData pushButtonData
    = new PushButtonData(
      "Active Workset",
      "Active Workset",
      AddInPath,
      className );

  ComboBoxData comboBoxDataLevel
    = new ComboBoxData( "LevelsSelector" );

  IList<RibbonItem> ribbonItemsStacked
    = ribbonSamplePanel.AddStackedItems(
      pushButtonData, comboBoxDataLevel );

  ( (PushButton) (ribbonItemsStacked[0]) ).Enabled
    = false;
```

Many thanks to Mikako Harada and Harry Mattison for this suggestion!