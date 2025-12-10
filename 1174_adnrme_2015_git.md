---
post_number: "1174"
title: "The ADN Sample AdnRme Migrated to Revit MEP 2015 and on GitHub"
slug: "adnrme_2015_git"
author: "Jeremy Tammik"
tags: ['elements', 'parameters', 'revit-api', 'views']
source_file: "1174_adnrme_2015_git.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1174_adnrme_2015_git.html"
---

### The ADN Sample AdnRme Migrated to Revit MEP 2015 and on GitHub

John asked a very relevant
[question](http://thebuildingcoder.typepad.com/blog/2014/06/revit-2015-update-release-3.html?cid=6a00e553e16897883301a73de0a111970d#comment-6a00e553e16897883301a73de0a111970d
) in
a comment on the
[Revit 2015 Update Release 3](http://thebuildingcoder.typepad.com/blog/2014/06/revit-2015-update-release-3.html) that
prompted me to complete the rather overdue migration of the ADN Revit MEP sample add-in AdnRme to Revit MEP 2015.

I also took this opportunity to create a GitHub repository for it:

- [AdnRme GitHub repository](https://github.com/jeremytammik/AdnRme)
- [Revit MEP 2014 version 2014.0.0.1](https://github.com/jeremytammik/AdnRme/releases/tag/2014.0.0.1)
- [Revit MEP 2015 version 2015.0.0.1](https://github.com/jeremytammik/AdnRme/releases/tag/2015.0.0.1)

Much easier to share and collaborate on.

The enhanced
[documentation](https://github.com/jeremytammik/AdnRme/blob/master/README.md) now
explains all the basics and points to more detailed discussions on everything else there is to know about it:

#### AdnRme

Revit MEP Sample Application for Revit MEP HVAC and electrical – Demonstrate use of the Revit API for MEP specific tasks.

#### HVAC

Use of the generic Revit API for HVAC specific tasks, using only standard Revit element properties and parameters:

- Determine air terminals for each space.
- Assign flow to the air terminals depending on the space's calculated supply air flow.
- Change size of diffuser based on flow.
- Populate the value of the 'CFM per SF' variable on all spaces.
- Determine unhosted elements.
- Reset demo.

#### Electrical

Use of the MEP specific API to traverse an electrical system and display its hierarchy in a tree view.

#### Documentation

- [AdnRme for Revit MEP 2013](http://thebuildingcoder.typepad.com/blog/2012/05/the-adn-mep-sample-adnrme-for-revit-mep-2013.html) – extensive description.
- [AdnRme for Revit MEP 2014](http://thebuildingcoder.typepad.com/blog/2013/06/the-adn-sample-adnrme-for-revit-mep-2014.html) – previous migration.
- [AdnRme for Revit MEP 2014 on GitHub](http://thebuildingcoder.typepad.com/blog/2014/06/adnrme-migrated-to-revit-mep-2015-on-github.html) – this migration.