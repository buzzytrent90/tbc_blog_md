---
post_number: "1837"
title: "Unit Types"
slug: "unit_types"
author: "Jeremy Tammik"
tags: ['family', 'parameters', 'python', 'references', 'revit-api', 'sheets', 'views']
source_file: "1837_unit_types.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1837_unit_types.html"
---

### Revit 2021 Unit Types in Family Type Catalogues
Unfortunately, the new unit type name functionality can cause a problem loading a family with a type catalogue:
- [Unit type update affects family type catalogue loading](#2)
- [New FreeCodeCamp courses](#3)
- [Padlocking The Building Coder](#4)
#### Unit Type Update Affects Family Type Catalogue Loading
\*\*Question:\*\* We have some families that fail to load in Revit 2021.
We have narrowed it down to a Revit change.
When you export PIPING__FLOW (GPM) data to a type 'Family Types' in previous versions of Revit, you get something like this:

```
Cold Water Flow##PIPING_FLOW##GALLONS_US_PER_MINUTE
```

However, when you do the same in Revit 2021, you get:

```
Cold Water Flow##PIPING_FLOW##US_GALLONS_PER_MINUTE
```

The bad news for us being that Revit 2021 does not accept GALLONS_US_PER_MINUTE anymore.
Instead, it expects (and does not ‘upconvert’) the new US_GALLONS_PER_MINUTE.
This is a breaking change for our existing type catalogues.
Is there a published list of changes for parameters that we can review?
\*\*Answer:\*\* Yes, indeed, this is an intentional change.
Sorry that it is affecting you so hard.
Just as you say, no automatic upgrade from the previous version’s DB strings has been implemented.
We documented these changes in the [developer guide](https://help.autodesk.com/view/RVT/2021/ENU/?guid=Revit_API_Revit_API_Developers_Guide_html).
The table of changes to database identifiers can be found at the bottom of the page
on [Introduction > Application and Document > Document Functions > Units](http://help.autodesk.com/view/RVT/2021/ENU/?guid=Revit_API_Revit_API_Developers_Guide_Introduction_Application_and_Document_Units_html).
#### New FreeCodeCamp Courses
I always enjoy browsing through the FreeCodeCamp courses recommended in Quincy Larson's newsletter.
Last week's bunch looked especially useful to me, for instance these:
- [The Ultimate Python Beginner's Handbook](https://www.freecodecamp.org/news/the-python-guide-for-beginners/)
- [Learn Data Analysis with Python – A Free 4-Hour Course](https://www.freecodecamp.org/news/learn-data-analysis-with-python-course/)
- [LockdownConf – A Free Online Conference to Help You Prepare for a Post-Pandemic World](https://www.freecodecamp.org/news/lockdownconf-free-developer-conference/)
- [How to Style Your React Apps with Less Code Using Tailwind CSS and Styled Components](https://www.freecodecamp.org/news/how-to-style-your-react-apps-with-less-code-using-tailwind-css-and-styled-components/)
#### Padlocking The Building Coder
Last week, colleagues pointed out that some of the Autodesk developer blogs were displaying a message saying 'Not secure' in the browser address bar:
![Address bar warning 'not secure'](img/why_no_padlock_blog_url_not_secure.png "Address bar warning 'not secure'")
The problem is caused by a mixed content error, where some references from the blog are not secure.
You can check the site either in Firefox, or using an external page, e.g., [whynopadlock.com](https://www.whynopadlock.com), which will conveniently show the references causing the mixed content error:
![Test results](img/why_no_padlock_test_results.png "Test results")

```
TEST RESULTS
Test Information
Tested URL https://thebuildingcoder.typepad.com/
Mixed Content - Errors
Soft Failure An image with an insecure url of "http://thebuildingcoder.typepad.com/tbc_banner6_1200_200.png" was loaded on line: 4572 of https://thebuildingcoder.typepad.com/.
This URL will need to be updated to use a secure URL for your padlock to return.
```

In The Building Coder, it was only the banner image, and it can be simply fixed by adding `https` to the image reference in the CSS.
![Adding https to banner image in CSS](img/why_no_padlock_banner_lacks_https.png "Adding https to banner image in CSS")
Now all is well and the site is padlocked again:
![Padlocked URL](img/why_no_padlock_fixed.png "Padlocked URL")