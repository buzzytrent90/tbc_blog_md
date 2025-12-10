---
post_number: "1200"
title: "How to Remove a Shared Parameter Definition"
slug: "del_shared_param_def"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'parameters', 'python', 'revit-api', 'rooms', 'transactions', 'views']
source_file: "1200_del_shared_param_def.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1200_del_shared_param_def.html"
---

### How to Remove a Shared Parameter Definition

Here is yet another example of the importance of the RevitLookup database exploration tool.

In this case, Trevor Taylor of [ZGF](http://www.zgf.com), Zimmer Gunsul Frasca Architects LLP, shows his use of it to discover how to remove a shared parameter definition.

#### Task at Hand

When I remove a shared parameter definition, it disappears from the UI.
However, the parameter definition isn't truly removed from the model.
Its definition persists.

Steps to repeat:

1. In a shared parameter definition file, create a parameter named 'MyTest' of data type TEXT.
2. Add that shared parameter to a Revit project associated with Rooms (or whatever).
3. Check a room to verify that the 'MyTest' parameter exists, then remove it using Manage > Project Parameters.
4. Edit the parameter definition in your shared parameters text file using a text editor.
   Change the data type to YESNO.
5. Re-add the edited shared parameter file to your project.

Revit will report that the shared parameter GUID exists and cannot be added.

So, it appears that once a parameter GUID is created, it exists eternally in the model file.

This makes it impossible to change the definition of the original parameter once it has been created.
Therefore, if a user creates a parameter with a certain GUID and needs to go back and change the properties (perhaps it was mistakenly created as the wrong data type), it can never be corrected.

So, my question is how to either delete the parameter definition from a project file so that it really goes away or, how to modify the parameter definition properties after creation?

#### Solution

I found a way around this problem using the ADN RevitLookup add-in.
I used Snoop DB to explore an element bound to the parameter and looked up the parameter element id.
Then I use Modify > Select by ID and by jingo, it selects it.
Press delete and to my amazement, it actually deletes the parameter definition.

I guess I just need to do some more digging in the API to see how the parameter's id is accessed.

#### Conclusion

Congratulations on solving the issue!

Many thanks to Trevor for his exploration and sharing!

RevitLookup is miraculously useful, isn't it?
In fact, any tool that enables interactive API access to the Revit database is extremely powerful, and the more interactive it is, the more flexibility and power it will provide.

E.g., the Revit Python and Ruby shells provide an even higher level of interaction, exploration and even modification possibilities, as explained in this demonstration of
[intimate Revit database exploration](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html).

RevitLookup is read-only, whereas the Ruby and Python shells can set up and commit transactions and write to the database as well.

As you also discovered, another thing that is more powerful than you might expect is the simple Document.Delete method.
Just as you say, feed it the right element id, or collection of element ids, and it will remove just about anything.

Actually, this approach reminds me of one of the very early posts on The Building Coder, from 2009, on
[deleting a shared parameter](http://thebuildingcoder.typepad.com/blog/2009/08/deleting-a-shared-parameter.html).

Also, by the way, before closing, as an additional reading item, I just happened across this nice overview by Paul Aubin of the
[four different types of parameters managed by Revit](http://paulaubin.com/blog/revit-parameters).