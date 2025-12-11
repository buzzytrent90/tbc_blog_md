---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:17.213672'
original_url: https://thebuildingcoder.typepad.com/blog/1990_rvt_2024_update.html
post_number: '1990'
reading_time_minutes: 5
series: general
slug: rvt_2024_update
source_file: 1990_rvt_2024_update.md
tags:
- elements
- parameters
- references
- revit-api
- selection
- sheets
- views
- windows
title: Rvt 2024 Update
word_count: 918
---

### News Reel, Roadmap and RevitLookup Updates
More news and updates related to Revit 2024 and some little titbits on AI and literature:
- [News reel and AEC roadmaps](#2)
- [Revit API training](#3)
- [RevitLookup 2024 updates](#4)
- [Free Dolly open-source instruction-tuned LLM](#5)
- [Websites powered by AI](#6)
- [Walkaway by Cory Doctorow](#7)
#### News Reel and AEC Roadmaps
To get a quick overview of what's new in the Revit 2024 product, you can check out the news reel:
- 00:00 [Introduction](https://youtu.be/qA74NHN8lh0)
- 00:30 [For Everyone](https://youtu.be/qA74NHN8lh0?t=30)
- 01:29 [Architects](https://youtu.be/qA74NHN8lh0?t=89)
- 02:37 [MEP](https://youtu.be/qA74NHN8lh0?t=157)
- 03:46 [Structure](https://youtu.be/qA74NHN8lh0?t=226)
- 05:00 [Model Coordination](https://youtu.be/qA74NHN8lh0?t=300)
- 05:24 [Documentation](https://youtu.be/qA74NHN8lh0?t=324)
To find out where it is going from here forward, explore
the updated [Autodesk AEC Public Roadmaps](https://blogs.autodesk.com/revit/roadmap/) and
take the opportunity to contribute your own requirements and ideas.
#### Revit API Training
I was asked once again about Revit API training:
\*\*Question:\*\* Do you teach or have you ever taught Revit in an online mode? I could use that.
\*\*Answer:\*\* We used to teach face-to-face courses before concentrating harder on making all material publicly available to larger audiences world-wide.
Now, the material we used back then has been published and maintained on GitHub:
- [ADN RevitTrainingMaterial](https://github.com/ADN-DevTech/RevitTrainingMaterial)
- [AdnRevitApiLabsXtra](https://github.com/jeremytammik/AdnRevitApiLabsXtra)
I never did online courses myself.
I know [Harry Mattison](https://www.youtube.com/user/BoostYourBIM) did, though.
#### RevitLookup 2024 Updates
We already have two RevitLookup updates to share, 2024.0.1 and 2024.0.2:
- [RevitLookup 2024.0.2](https://github.com/jeremytammik/RevitLookup/releases/edit/2024.0.2)
– Fixed Fatal Error on Windows 10 [#153](https://github.com/jeremytammik/RevitLookup/issues/153)
– Accent colour sync with OS now only available in Windows 11 and above
– Many thanks to [Aleksey Negus](https://t.me/a_negus) for testing builds.
- [RevitLookup 2024.0.1](https://github.com/jeremytammik/RevitLookup/releases/edit/2024.0.1) – see below:
Breaking changes:
- Added option to enable hardware acceleration (experimental)

The user interface is now more responsive. Revit uses software acceleration by default. Contact us if you encounter problems with your graphics cards

Known issue: rendering performance drops on selection. This is especially evident on roofs,
cf. [Revit 2024 rendering performance drops on selection](https://forums.autodesk.com/t5/revit-api-forum/revit-2024-rendering-performance-drops-on-selection/td-p/11878396)
- Added button to enable RevitLookup panel on Modify tab by @ricaun in [#152](https://github.com/jeremytammik/RevitLookup/pull/152)

Disabled by default. Thanks for voting! [#151](https://github.com/jeremytammik/RevitLookup/discussions/151)
- Opening RevitLookup window only when the Revit runtime context is available [#155](https://github.com/jeremytammik/RevitLookup/issues/155)
Improvements:
- Added shortcuts support for the Modify tab [#150](https://github.com/jeremytammik/RevitLookup/issues/150)
- Added EvaluatedParameter support
- Added Category.get_Visible support
- Added Category.get_AllowsVisibilityControl support
- Added Category.GetLineWeight support
- Added Category.GetLinePatternId support
- Added Category.GetElements extension
- Added Reference.ConvertToStableRepresentation support
Bugs:
- Fixed rare crashes in EventMonitor on large models
- Fixed Curve.Evaluate resolver using EndParameter as values
Other:
- Added installers for [previous RevitLookup versions](https://github.com/jeremytammik/RevitLookup/wiki/Versions)
Many thanks to Roman [Nice3point](https://github.com/Nice3point) for his tremendous work maintaining and improving RevitLookup!
#### Free Dolly Open-Source Instruction-Tuned LLM
In spite of its name, [OpenAI](https://en.wikipedia.org/wiki/OpenAI) is not open.
Hence, this is exciting, welcome and positive news for the open community:
- [Free Dolly: Introducing the World's First Truly Open Instruction-Tuned LLM](https://www.databricks.com/blog/2023/04/12/dolly-first-open-commercially-viable-instruction-tuned-llm)
Coming up soon is a webinar on how
to [build your own large language model like Dolly](https://www.databricks.com/resources/webinar/build-your-own-large-language-model-dolly) detailing
how to fine-tune and deploy your custom LLM.
#### Websites Powered by AI
To experience some helpful AI in action yourself, you can check
out [12 secret websites powered by AI to finish hours of work in minutes](https://twitter.com/heyBarsee/status/1646161514682884099?s=20).
#### Walkaway by Cory Doctorow
I discovered a new author and a new favourite book,
[Walkaway](https://en.wikipedia.org/wiki/Walkaway_(Doctorow_novel))
by [Cory Doctorow](https://en.wikipedia.org/wiki/Cory_Doctorow).
For me, it is a brilliant philosophical scifi reminiscent
of [William Gibson's Agency](https://thebuildingcoder.typepad.com/blog/2021/10/sci-fi-languages-and-pipe-insulation-retrieval.html#4) that I was so thrilled by in 2021.
It is also focused on community, cooperation, communication, appreciation, relationships, exploitation, rich exploiters fighting to maintain a world of inequality, walkaways proving that we live in a world of surplus, not shortage, realising a post-scarcity gift economy, on-site fabrication of everything you need, and finally moving towards real visionary scifi ideas like eternal life using digital simulations of the human mind.
It also taught me some new vocabulary,
such as pwn,
as in [owned and pwned](https://en.wikipedia.org/wiki/Leet#Owned_and_pwned).
That in turn led me to discover [leet](https://en.wikipedia.org/wiki/Leet) and
[leetspeak translators](https://duckduckgo.com/?q=leetspeak+translator).
Not awfully useful in everyday life, but cool stuff to be aware of.
![Walkaway by Cory Doctorow](img/walkaway_cory_doctorow.jpg "Walkaway by Cory Doctorow")