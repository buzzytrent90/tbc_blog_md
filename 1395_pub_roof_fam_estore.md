---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: news
optimization_date: '2025-12-11T11:44:15.930504'
original_url: https://thebuildingcoder.typepad.com/blog/1395_pub_roof_fam_estore.html
post_number: '1395'
reading_time_minutes: 4
series: general
slug: pub_roof_fam_estore
source_file: 1395_pub_roof_fam_estore.md
tags:
- csharp
- elements
- family
- parameters
- python
- revit-api
- selection
- sheets
- views
- walls
title: Pub Roof Fam Estore
word_count: 834
---

### Era of Connection, Families and Data Handling
Let's end this week by taking a look at two new AEC and BIM related publications: a new Autodesk eBook, Era of Connection, and a Dutch architectural design student project exploring Revit families and data handling:
- [Era of Connection](#2)
- [Revit Families and Data Handling](#3)
#### Era of Connection
Check out the new Autodesk AEC and BIM eBook [eraofconnection.com](http://eraofconnection.com) on connecting teams, insight, outcomes and delivery:
> The pace of change is unrelenting: Technology is radically disrupting the way buildings and infrastructure are designed, built, and used. We’re now entering a new age of opportunity for builders, designers and engineers. The power of the cloud + Big Data means we can connect people, processes, ideas, and things like never before. As physical and digital technology come together to create endless possibilities for designing, building, and operating buildings and infrastructure, it’s never been so important to embrace BIM. To help you harness the opportunities, Autodesk is connecting to the future with tightly connected BIM tools and processes for the lifecycle for buildings and infrastructure. This is the future of making things.
- [Connecting teams](http://eraofconnection.com/connecting-teams): In the era of connection, we’re transitioning from applications and files to putting the project at the centre from the start. Using the cloud to connect data and systems, projects and teams are always up to date – in the office and on the job site – sharing information and collaborating across the project lifecycle in real time, without barriers.
- [Connecting insight](http://eraofconnection.com/connecting-insight): Connected technology puts 'big data' to work to help deliver better designs and more predictable performance. Beginning with conceptual design through project delivery and asset management, project teams will be able to tap into huge amounts of data to put their ideas into context and deliver 'best possible' building and infrastructure performance.
- [Connecting outcomes](http://eraofconnection.com/connecting-outcomes): Start with your desired outcome, then explore a near infinite range of possibilities and produce optimal designs in a fraction of the time – the power of the cloud helps make this a reality today. Designers and engineers can generate multiple options, ideas and scenarios more rapidly than ever before – exploring forms, simulating performance, and impressing clients with cinematic quality renders and optimal designs.
- [Connecting delivery](http://eraofconnection.com/connecting-delivery): As the lines between digital processing and physical systems blur, the design and build phases of projects are moving closer together. Seamless integration of these processes using pre-fabrication, modular construction and 3D printing saves time and money.
[eraofconnection.com](http://eraofconnection.com)

#### Revit Families and Data Handling
This is a final project by Martin Plasschaert, HBO ACE Architectural Designer, at the TEC/CAD College in Nijmegen, Holland, on the topic of Revit Families and Data Handling.
The main topics are the analysis, design and implementation of custom roof families and extensible storage data management tools.
![Roof family](img/mp_roof_family.png)
Here is the table of contents:

```
1. Introduction
	1.1 Used software and programming languages
2. Problem
	2.1 Problem family wall plate
	2.2 Problem family roof element
	2.3 Problem Data Tool
3. Purpose
	3.1 Purpose family wall plate
	3.2 Purpose family roof  element
	3.3 Purpose Data Tool
4. Approach
5. Elaboration family wall plate
	5.1 Variable dimensions
	5.2 Variable roof pitch
	5.3 Optional nested families
6. Elaboration family roof element
	6.1 Base family roof element
	6.2 Variable dimensions
		6.2.1 Variable lath distance
	6.3 Variable slope
	6.4 Parameters nested families
	6.5 Visibility family components
		6.5.1 Visibility control by yes/no parameter
		6.5.2 Visibility control by object styles subcategory
7. Elaboration Data Tool
	7.1 Integrated Revit menu structure
		7.1.1 Tab
		7.1.2 Ribbon panels
		7.1.3 Pushbuttons
		7.1.4 Images
		7.1.5 Tooltips
		7.1.6 Quick access toolbar
	7.2 Data handling
		7.2.1 Assigning data
		7.2.2 Reading  data
		7.2.3 Deleting  data
		7.3 Selection based on data
			7.3.1 Highlight based on data
			7.3.2 Hide based on data
		7.4 Database evaluation
			7.4.1 Export to database
			7.4.2 Show data from database
				7.4.2.1 Show data extern application
				7.4.2.2 Show data in browser
				7.4.2.3 Show data in Revit UI
			7.4.3 Edit data in database
			7.4.4 Update model data from database
		7. 5 Display production progress in virtual model
		7. 6 Add parameters to family via API
		7. 7 Provide parameters with extensible storage data
```

Martin's original paper is in Dutch. He very kindly also created a rough English translation for publication here:
- [Revit Families en Data Handling (Dutch)](zip/revit_families_en_data_handling_verslag.pdf)
- [Revit Families and Data Handling (English)](zip/revit_families_en_data_handling_verslag_en.pdf)
Congratulations to Martin on his impressive work, and many thanks for translating and sharing it here with us!