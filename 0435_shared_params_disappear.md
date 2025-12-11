---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.942079'
original_url: https://thebuildingcoder.typepad.com/blog/0435_shared_params_disappear.html
post_number: '0435'
reading_time_minutes: 7
series: general
slug: shared_params_disappear
source_file: 0435_shared_params_disappear.htm
tags:
- csharp
- elements
- parameters
- revit-api
- selection
- windows
title: Modeless Form and Shared Parameter Disappearance
word_count: 1311
---

### Modeless Form and Shared Parameter Disappearance

I am still acclimatising to Europe after four weeks in Thailand and Cambodia.
So many differences: time zone, temperature, food, health, people...

Yesterday I was able to join some colleagues in Neuchâtel for a game of hockey, which is generally the most exhausting sportive event I participate in on a regular basis.
I managed to keep up in spite of feeling a bit weak, and was saved by the rain which stopped us from going for the full customary hour and a half.
Great fun!

Here is a question from before my holidays which starts out discussing a shared parameter visibility issue seemingly arising from a port of an application from a previous API version to Revit 2011.
However, it ends up as a completely different issue, showing the importance of proper handling of asynchronous access to the Revit API, for instance the special care that needs to be taken when trying to interact with Revit from a modeless dialogue.

This just goes to show how much confusion an attempt to access the Revit from an invalid context can cause.
It also shows how useful it can be to create a really minimal sample project to reproduce an issue, because that very often narrows down the question so significantly, almost automatically producing the answer.
Finally, it also shows that a completely different approach can vastly reduce complexity and avoid certain errors completey, in this case the use of the new PickObjects method instead of a modeless dialogue.

**Question 1:** We have to add some instance parameters through the API to elements within our Revit model.
I used the code on page 270 of the 2011 developer's guide.
This code works fine until I hit the save button.
After save the newly added parameters are 'lost'.
They are no longer visible within the element properties.
Though, they are still visible in the shared parameter window within Revit.

In 2010 we use the 2010 version of the same code without any problems.
The difference we see is that after the parameters are added, in 2010 they show up as project parameters.
In 2011 they do not show up as project parameters.

Is there a fix or workaround so that my parameters are not getting lost in 2011?

**Answer 1:** I had a look at the sample code you mention, and I see that it uses the line
```csharp
  // create an instance definition in
  // definition group MyParameters
  Definition myDefinition\_ProductDate
    = myGroup.Definitions.Create(
      "Instance\_ProductDate",
      ParameterType.Text );
```

Looking at the Definitions.Create method in the Revit API help file RevitAPI.chm, I see the following two overloads
listed:

- Create(String, ParameterType) Creates a new parameter definition using name, type and visibility.- Create(String, ParameterType, Boolean) Creates a new user visible parameter definition using name and type.

Note that the second version taking three parameters mentions 'user visible', whereas the first overload with only two arguments does not.
The name of the Boolean argument is 'visible'.
I assume that you need to set that to true to ensure that the shared parameter appears in the user interface.

I looked at the Revit 2010 and 2009 API help files as well, and the overloads there are identical and described in a similar fashion, so I do not actually see how that should affect the behaviour of your code. Anyway, I would suggest using the overload taking the 'visible' argument and setting it to true and seeing whether that helps.

Some categories do not allow visible shared parameters to be added to them at all.
This is indicated by the AllowsBoundParameters property on the corresponding category instance.

For an example of checking that property and setting the shared parameter to be visible whenever that is permitted,
please refer to the CmdCreateSharedParams command in The Building Coder sample code that I touched on most recently when discussing the creation of a
[shared type parameter](http://thebuildingcoder.typepad.com/blog/2010/07/shared-type-parameter.html).
It is also described in previous posts to highlight various topics in the area of the creation of shared parameters
that may also be of interest to you.

Here is the code snippet that checks the property and sets the visibility accordingly:
```csharp
  // set visibility of the new parameter:

  // Category.AllowsBoundParameters property
  // indicates if a category can have shared
  // or project parameters. If it is false,
  // it may not be bound to shared parameters
  // using the BindingMap. Please note that
  // non-user-visible parameters can still be
  // bound to these categories.

  bool visible = cat.AllowsBoundParameters;

  // get or create the shared params definition:

  string defname = \_defname + nameSuffix.ToString();

  Definition definition = group.Definitions.get\_Item(
    defname );

  if( null == definition )
  {
    definition = group.Definitions.Create(
      defname, \_deftype, visible );
  }
```

**Question 2:** I created a sample project to illustrate the problem and now I know exactly where the problem is.
The problem seems not to be related to the method of adding parameters after all.

I add parameters from within a Windows form.
When I display this form with MyForm.ShowDialog it all works fine and the parameters do not disappear.
The problem occurs when I want to keep my form open.
For that I use MyForm.Show.

I want to keep the form open while users set different parameter values to different Revit elements.
To close and open the tool each time for every new set of elements is annoying, so I need to keep the form open.

Is there another way to keep the form open so that users can select one or more elements while the form is open?

**Answer 2:** Thank you for your update, and I am very glad that you were able to nail down the exact problem.

That clarifies everything completely for me. Now let's see if I can explain it to you.

The problem you are encountering is the fact that the Revit API is only available and accessible to your add-in as long as you remain within a valid Revit API context.
One such context is during the execution of your external command Execute method. Other valid Revit API contexts are provided by all of the various notification methods called by the Revit API events.

I provided a detailed discussion of this issue in the presentation of my
[modeless loose connectors](http://thebuildingcoder.typepad.com/blog/2010/07/modeless-loose-connectors.html) navigator,
which points to more detailed previous posts on the topic as well.

In short, you can solve the problem by subscribing to the Idling event.
Your modeless form has no valid Revit API context.
When some user action in your modeless form requires access to the Revit API, you can notify you Idling event handler, which does have Revit API access.
The next time it fires, it can pick up the request from the modeless form and execute it.

The Idling event is called so frequently that the user will not necessarily notice that the modeless form is actually running in a separate thread, as demonstrated by my modeless loose connector navigator form.

**Response:** Thank you for your reply.
I understand why it loses the API context and how I can implement a work around through the idle event, but maybe I have another easier solution.
If I remember well, from 2011 onwards the API provides a PickObjects method.
The reason to keep the window open is to let the user select elements.
Maybe I can alter the user interface a little to make use of this method.

**Answer:** Yes, making use of the Selection.PickObjects method instead will definitely be much simpler and more stable.
If you can avoid the modeless interaction, you definitely should.
For samples of using PickObjects, please refer to the Revit SDK Selections sample and my
[pipe to conduit converter](http://thebuildingcoder.typepad.com/blog/2010/05/pipe-to-conduit-converter.html) sample.