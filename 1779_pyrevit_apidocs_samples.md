---
post_number: "1779"
title: "Pyrevit Apidocs Samples"
slug: "pyrevit_apidocs_samples"
author: "Jeremy Tammik"
tags: ['python', 'revit-api', 'sheets']
source_file: "1779_pyrevit_apidocs_samples.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1779_pyrevit_apidocs_samples.html"
---

### pyRevit Home and ApiDocs Code Sample Collection
Two exciting Revit API related news announcements from Ehsan Iran-Nejad and Gui Talarico:
- [New comprehensive pyRevit home page](#2)
- [ApiDocs.co code search sample collection](#3)
- [How does it work?](#3.1)
- [Limitations](#3.2)
- [How to contribute](#3.3)
- [RevitApiDocs and ApiDocs comparison](#3.4)
#### New Comprehensive pyRevit Home Page
Ehsan [@eirannejad](https://twitter.com/eirannejad) Iran-Nejad
[unified the pyRevit online experience](https://twitter.com/eirannejad/status/1170576981538172928?ref_src=twsrc%5Etfw),
creating a new [pyRevit home page](http://wiki.pyrevitlabs.io) that
takes you through all project related aspects:
![pyRevit home page](img/pyrevit_home_page_2.png)
#### ApiDocs.co Code Search Sample Collection
Gui [@gtalarico](https://twitter.com/gtalarico) Talarico, the author of both
the [online Revit API documentation revitapidocs.com](https://www.revitapidocs.com) and the more
general [Apidocs.co](https://apidocs.co) covering Grasshopper, Navisworks and Rhino as well,
[announced an expanded search functionality](https://twitter.com/gtalarico/status/1170473246275145729?ref_src=twsrc%5Etfw) in
the latter that provides code samples directly within its pages by searching a whole collection of samples hosted in the
new [ApiDocs.co code search sample repository](https://github.com/gtalarico/apidocs.samples).
![ApiDocs.co](img/apidocs.co.png)
##### How Does it Work?
[Apidocs.co](https://apidocs.co) uses the GitHub Code Search API against this repo to provide Code Samples directly within pages.
Because the GitHub Code Search API is limited to a single user or repo, this repository aggregates multiple relevant repos so they can all be searchable in a single request.
##### Limitations
- It's plain text search – some generic names like `Application` can trigger many false positives
- It's limited to certain entity types (e.g., `Class`, `Method`, `Property`, etc.)
##### How to Contribute
- Fork this repo
- Add a relevant repo to `repos.json`
- Run `python update.py`
- Send a [Pull Request](https://github.com/gtalarico/apidocs.samples/pulls)
##### RevitApiDocs and ApiDocs Comparison
\*\*Question:\*\* Could the new code sample search functionality be added to
both [apidocs.co](http://apidocs.co)
and [revitapidocs](https://www.revitapidocs.com)?
It is tricky to know when to use which...
\*\*Answer:\*\* Users can use whichever they prefer.
While [revitapidocs](https://www.revitapidocs.com) will likely continue to get new API versions, it will not get new features – the code base has not aged well and adding new features to it is no fun.