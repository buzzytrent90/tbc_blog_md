---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 9.4
content_type: qa
optimization_date: '2025-12-11T11:44:17.181521'
original_url: https://thebuildingcoder.typepad.com/blog/1981_chatgpt_basics.html
post_number: '1981'
reading_time_minutes: 21
series: general
slug: chatgpt_basics
source_file: 1981_chatgpt_basics.md
tags:
- elements
- family
- filtering
- geometry
- parameters
- references
- revit-api
- rooms
- schedules
- sheets
- views
- walls
title: Chatgpt Basics
word_count: 4149
---

### Back to Basics and ChatGPT
I answered a couple of basic questions for the umpteenth time and thought I might summarise some of them here yet again.
I am still impressed and intrigued by what ChatGPT can and cannot do and keep trying out new aspects myself and with my friends.
I strongly advise anyone interested in it to try it out for yourself rather than just reading the numerous articles published by others – for example, my notes below :-)
- [Back to the basics](#2)
- [Element filtering](#3)
- [XYZ trigonometry](#4)
- [Projecting points](#5)
- [Train ChatGPT for Revit API](#6)
- [Asking ChatGPT a Revit API question](#7)
- [Using ChatGPT with Revit API](#8)
- [Generative AI and multi-modal learning](#9)
- [More notes on ChatGPT et al](#10)
- [Kean on the coming year](#10.01)
- [ChatGPT in academia](#10.02)
- [Building a VM inside ChatGPT](#10.04)
- [How to make your own ChatGPT](#10.05)
- [Differentiate human vs. AI](#10.06)
- [ChatGPT and education](#10.07)
- [ChatGPT payment plan](#10.08)
- [Uplifting books](#10.09)
- [Solarpunk](#10.10)
- [Opus.ai type and play](#10.11)
- [MusicLM generates music from text](#10.12)
- [AI-generated art wins competition](#10.13)
- [ChatGPT abandons its life dream](#10.14)
#### Back to the Basics
Here are a couple of typical questions from StackOverflow and the discussion forum where I repeat some basic recommendations that I have already given before:
- [Element filtering](#3)
- [XYZ trigonometry](#4)
- [Projecting points](#6)
Maybe, by now, or very soon, based on the publicly available information, ChatGPT and other pretrained language models will be able to answer them just as well or better than I can.
See below for examples of
what it [can do – using ChatGPT with Revit API](#8),
what it [can't do – asking ChatGPT a Revit API question](#7),
and [an explanation of why it works at all – generative AI and multi-modal learning](#9).
#### Element Filtering
[Simplify a series of repetitive functions with sort options](https://stackoverflow.com/questions/74806242/simplify-a-series-of-repetitive-functions-with-sort-options)
I have a series of functions in a module which are starting to become quite repetitive.
Each function extracts a list and has an optional Boolean argument to sort the list before returning it.
Feels like there ought to be a way to inherit the sorting from a parent function?

```
def get_electrical_equipment(sort_by_name = False):

  elements =  DB.FilteredElementCollector(revit.doc)\
    .OfCategory(DB.BuiltInCategory.OST_ElectricalEquipment)\
    .WhereElementIsNotElementType()\
    .ToElements()

  if sort_by_name: elements.sort(key=lambda x: x.Name)

  return elements

def get_panel_schedules(sort_by_name = False):
  elements = DB.FilteredElementCollector(revit.doc)\
    .WherePasses(DB.ElementClassFilter(DB.Electrical.PanelScheduleView))\
    .WhereElementIsNotElementType()\
    .ToElements()

  if sort_by_name: elements.sort(key=lambda x: x.Name)

  return elements

def get_panel_schedule_sheet_instances(sort_by_name = False):
  elements = DB.FilteredElementCollector(revit.doc)\
    .OfClass(DB.Electrical.PanelScheduleSheetInstance)\
    .ToElements()

  if sort_by_name: elements.sort(key=lambda x: x.Name)

  return elements
```

\*\*Answer:\*\* First, I think you can completely eliminate the call to `ToElements`.
It is a waste of memory and computation time, as I have pointed out repeatedly in the past, e.g.,
in [How to Distinguish Redundant Rooms](https://thebuildingcoder.typepad.com/blog/2016/04/how-to-distinguish-redundant-rooms.html#2).
Now, to address your question, you can simply implement a common method `get_elements_of_category_and_class` taking a category and a class argument.
Pass in either one or the other or both and execute `OfClass` and `OfCategory` checks on the filtered element collector, either one or the other or both, skipping evaluation of `null`-valued arguments.
#### XYZ Trigonometry
Another recurring question is basic trigonometry, such
as [how to create a vector `XYZ` tilted up from the view direction by a specified angle](https://forums.autodesk.com/t5/revit-api-forum/how-to-create-a-vector-xyz-tilted-up-from-the-view-direction-by/m-p/11621339):
\*\*Question:\*\* I have the view direction that I use for my reference intersector.
I would like to try various altitude angles up from the view direction in a section with starting point at a wall like in this section view:
![Vector tilted up from plane](img/vector_tilted_up.png "Vector tilted up from plane")
I know there are transforms and various complex maths answers on StackOverflow; I was hoping to use a simpler built-in method if available?
\*\*Answer:\*\* This kind of trigonometry is not difficult.
Your children learn it in school, I hope.
Please take a moment to either read
the [Wikipedia article on Trigonometric functions and the Law of tangents](https://en.wikipedia.org/wiki/Trigonometric_functions#Law_of_tangents) or
study some other tutorials.
From that article, I find this image on the unit circle definitions of six trigonometric functions most helpful:
![Unit circle definitions of six trigonometric functions](img/trigonometry_functions.png "Unit circle definitions of six trigonometric functions")
Decide what angle you wish to use, e.g., 30 degrees.
Determine its tangens value, ca. 30 x 3.14 / 180 = 0.5.
Take your horizontal view direction XYZ vector (x,y,0).
Replace the Z coordinate by the tangens you calculated, yielding (x,y,0.5).
Voila.
That is your new tilted direction vector.
You may normalise it if you like.
Please do not be afraid of trigonometry, it is very intuitive as soon as you stop being scared of it.
I condemn our teachers and education systems (not all, but all too many) for inoculating kids with fear of maths and geometry.
This is basic human intuitive understanding.
The Greeks mastered it 3000 years ago.
We can handle a computer and a smartphone, but not simple trigonometry?
Why?
\*\*Answer 2:\*\* I would probably look at it in terms of the ratio between the forward direction and XYZ.BasisZ that the angle represents.
Also, the `XYZ` class has good functionality for arithmetic operations.
Similar as noted above, below shows an easy way:

```
Dim V0 As New XYZ(1, 0.5, 0) 'some random flat direction
V0 = V0.Normalize
Dim Ang As Double = 30 'Angle in degrees
Dim Ang_r As Double = (Math.PI / 180) * Ang
Dim T As Double = Math.Tan(Ang_r) 'Tan(Ang) = Opp/1
Dim Vz As New XYZ(0, 0, T)
Dim V1 As XYZ = (V0 + Vz).Normalize 'The direction with tilt of 30 degrees up from horizontal
```

This works from 0 to < 90.
From that point on, you have to check the quadrant and consider the `Tan` function doesn't work approaching 90 or 270 degrees, but you know those values are looking straight up and straight down respectively.
#### Projecting Points
Another simple geometric question came up
on [Using AVF on ductwork – coordinates issue?](https://forums.autodesk.com/t5/revit-api-forum/using-avf-on-ductwork-coordinates-issue/m-p/11621128):
\*\*Question:\*\* I am currently trying to create an add-in that will allow me to perform a calculation on various points on a duct face.
For now, the user is selecting the face, but in the future, I plan to process the top face of each duct in the view.
I am following along with the [RvtFader example project](https://github.com/jeremytammik/RvtFader).
A reference to my face is passed into the `SpatialFieldManager`, exactly like in the RvtFader example.
The UpdateSpatialFieldPrimitive method runs without error, and my duct is shaded with a single solid color.
I believe my issue is that I am feeding the FieldDomainPointsByUV with UV points that are in the global coordinate system instead of the local coordinates system to the face. I suspect because creating a BoundingBoxUV of my face, and supplying the bb.Min and bb.Max, along with 0, 10 as FieldValues, I get a correctly shaded (using a gradient) duct.
Is there a way to convert WCS XYZ points to local UV points?
\*\*Answer:\*\* Normally, the `UV` coordinates depend on the face.
Simply converting the `XYZ` X and Y values directly to UV will not work.
To convert a XYZ point to the face UV space, you can use
the [Face.Project method](https://www.revitapidocs.com/2023/4bee3e30-74fa-3103-c2f4-d07618fbcedf.htm) to
project it from global 3D space onto the face 2D.
\*\*Response:\*\* Thank you. That was just what I needed!
#### Train ChatGPT for Revit API
I asked ChatGPT some basic questions to determine whether it knows about the Revit API, Dynamo, The Building Coder blog and the Revit API discussion forum.
The results were surprisingly mixed:
Jeremy: Do you know about the Revit API?
ChatGPT: Yes, I am familiar with the Revit API. The Revit API is a software application programming interface (API) that allows users to access and manipulate the data contained in Autodesk Revit projects. This API can be used to develop custom tools and applications that integrate with Revit, allowing users to automate tasks and extend the functionality of the software.
Jeremy: So, what is the difference between Revit API and Dynamo?
ChatGPT: Dynamo is a visual programming platform that can be used with Revit to automate and simplify tasks and workflows. The Revit API, on the other hand, is a set of programming tools and libraries that can be used to develop custom applications and tools that integrate with Revit. While Dynamo is a user-friendly platform that allows users to create scripts and automate tasks without writing code, the Revit API is a more advanced tool that requires some programming knowledge to use effectively. Additionally, Dynamo is built on top of the Revit API, so using the Revit API allows for even greater customization and control over the automation of tasks in Revit.
Jeremy: Do you know the Revit API discussion forum?
ChatGPT: I am not aware of a specific discussion forum for the Revit API. However, there are many online forums and communities where users of the Revit API can ask questions, share tips and advice, and discuss their experiences with the API. Some examples of these forums include the Autodesk Developer Network forums, the Autodesk Revit API discussion group on LinkedIn, and the Revit API discussion forum on Autodesk Community. These forums can be a great resource for users of the Revit API who are looking for help, advice, or simply want to connect with other users of the API.
Jeremy: Do you know The Building Coder blog?
ChatGPT: I'm sorry, but I am not aware of The Building Coder blog. As a large language model trained by OpenAI, my knowledge is based on the text that has been fed into my training data. I do not have the ability to browse the internet or access additional information outside of what has been provided to me during my training. I apologize if I am unable to provide you with the information you are looking for. Is there anything else I may be able to assist you with?
Jeremy: Can you read The Building Coder blog
at [thebuildingcoder.typepad.com](https://thebuildingcoder.typepad.com/) so
that you later have access to that information?
ChatGPT: As a language model trained by OpenAI, I do not have the ability to browse the internet or access information outside of what has been provided to me during my training. I am only able to provide information and respond to questions based on the text that has been fed into my training data. I apologize if I am unable to provide you with the information you are looking for. Is there anything else I may be able to assist you with?
Jeremy: Can I provide you with additional training text to expand your knowledge in a specific domain?
ChatGPT: As a pre-trained language model, I am not able to accept additional training text or expand my knowledge in a specific domain. My knowledge is based on the text that was used to train me, and I do not have the ability to learn or acquire new information outside of what was included in my training data. I am designed to assist with a wide range of questions and topics, but my answers are limited to the information and knowledge that was provided to me during my training. If you have any specific questions, I would be happy to try to provide an answer based on my existing knowledge.
Jeremy: Can I access your source code in order to expand your training myself?
ChatGPT: I'm sorry, but I am not a physical machine or computer program that can be accessed or modified. As a large language model trained by OpenAI, I exist solely as a virtual assistant that is able to provide answers to questions and respond to statements based on the text that was used to train me. I do not have a source code or any other physical representation that can be accessed or modified. I apologize if I am unable to meet your request. Is there anything else I may be able to assist you with?
#### Asking ChatGPT a Revit API Question
I tried out asking ChatGPT a Revit API question in Joris van der Meulen's thread
on [placing a view reference](https://forums.autodesk.com/t5/revit-api-forum/placing-a-view-reference/m-p/11717917).
It provided a very plausible-sounding answer, but completely wrong:
Jeremy: Is it possible to place a view reference programmatically using the Revit API?
ChatGPT: Yes, it is possible to place a view reference programmatically using the Revit API.
Jeremy: What API calls does that require?
ChatGPT: Placing a view reference programmatically using the Revit API requires creating a new instance of a Viewport type, setting its location and size, and linking it to the target view. The following API calls can be used:
- Viewport.Create() to create a new instance of Viewport.
- ElementTransformUtils.MoveElement() to set the location of the Viewport.
- Viewport.get_Parameter() and Parameter.Set() to set the size of the Viewport.
- Viewport.ViewId property to link the Viewport to the target view.
This is incorrect, however, as Joris and Sean Page point out:
ChatGPT suggests creating a viewport.
A viewport is something we place on a sheet.
A View Reference is in instance of a system family which we place on a view (floorplan, section, etc).
As far as I know, this is not possible, nor can you modify the Reference after it is placed because the parameter is read-only.
So, it isn't even like you could copy and paste, then update.
Anyway, it is surprising and impressive how sensible-sounding the answer is; plus, in many cases, it is actually correct.
It all depends on what ChatGPT was fed in its pre-training curriculum.
#### Using ChatGPT with Revit API
On the other hand, Mohamed Elimam, SIAC Construction, presented
some [success using ChatGPT to help work with Revit API](https://www.linkedin.com/posts/mohamed-elimam_programming-chatgbt-ai-activity-7010260045649063936-dQds).
#### Generative AI and Multi-Modal Learning
The past few months hint at exciting and adventurous times full of radical change ahead.
But why, and how?
What happened so suddenly to enable this?
The current evolution of machine learning models is phenomenal.
Advancement will accelerate faster still with growing numbers of companies, labs, and institutions investing in this area.
The most remarkable achievement lately was the large language model GPT3 from OpenAI, launched in 2021.
This year, OpenAI is set to release GPT4, a larger and more advanced version.
The impact of this technology is unparalleled in the history of software.
It rivals the introduction of computers themselves and may have an even more profound effect on our work and everyday life than the Internet, mobile computing, and graphical user interfaces.
It will change the way we interact with computers, turning them into personal assistants, and alter the very meaning of creation.
This technology will challenge us in ways we have never experienced before, disrupting industries and leaving legal frameworks struggling to keep up.
As a result, new economies will emerge, and others will fade.
The driving force behind this innovation is a combination of Generative AI and Multi-Modal Learning.
The most common example of Generative AI is Large Language Models (LLM), which can both process and generate language.
Researchers feed LLMs massive amounts of text in many languages, including speech, writing and programming. The models then learn to produce new and original text based on prompts with impressive results.
These models essentially learn about language and how we use it by identifying patterns and frequencies of words and phrases.
As they process the vast amounts of data fed in, they acquire increasingly advanced and nuanced concepts that hold meaning for us.
This may seem like magic, but makes total sense given the central role that language plays in our human lives:
> The truly unique feature of our language is...
> the ability to transmit information about things that do not exist at all.
> As far as we know, only Sapiens can talk about entire kinds of entities
> that they have never seen, touched or smelled.
> – Yuval Noah Harari, Sapiens

Multi-Modal Learning is the ability to learn from multiple sources of information at the same time.
Our brains are naturally good at this, especially during our early years as infants and toddlers.
This is how we learn about the world and make connections between different senses and experiences.
As an example, consider a baby being bathed, learning to hold and drink from a glass, splashing in the washbasin and hearing its family members say 'water'.
If you are interested in pondering the philosophical side of this, I also recommend another book by Yuval Noah Harari, *21 Lessons for the 21st Century*: a brilliant, eye-opening, neutral and hence automatically, unintentionally, critical discussion of important questions humanity is facing today; basically, we use and need stories to organise ourselves, and all stories are fantasies; which stories do we agree on? abstract constructs do not suffer, stories do not suffer; individual humans do; how to end suffering? let go of all attachments, meditate; or, find your own way; my favourite chapters are in the final part of the book, on education and meditation; however, i cannot say whether they can be understood and enjoyed without reading the preceding chapters the others first...
#### More Notes on ChatGPT et al
I performed a few experiments with friends asking various questions in different domains, some of them in German.
The following is a list of pointers and short notes on those and other's experiments and a couple of other related issues.
What have I learned from all of this?
Well, one clear take-away is the following:
I cannot teach ChatGPT new tricks, since it is pre-trained.
I can, however, ensure that as much reliable information as possible is out there on the web to hopefully be included in the pre-training resources in the next version.
In the case of the Revit API, I do so by moderating the Revit API discussion forum and trying to provide reliable information in The Building Coder blog, hoping that that information will find its way into the pre-training dataset sooner or later.
#### Kean on the Coming Year
Kean Walmsley shared
his [reflections on what’s coming in 2023](https://www.keanw.com/2023/01/reflections-on-whats-coming-in-2023.html),
partly focussed on ChatGPT.
#### ChatGPT in Academia
An interesting analysis of ChatGPT in academia,
on [how a faculty should deal with the problem of AI generated texts](https://academia.stackexchange.com/questions/191643/how-should-a-faculty-deal-with-the-problem-of-artificial-intelligence-ai-gener).
#### Building a Virtual Machine inside ChatGPT
You can do some funny things with it, such
as [build and run a virtual machine inside ChatGPT](https://www.engraved.blog/building-a-virtual-machine-inside/).
#### How to Make Your Own ChatGPT
[How to make your own ChatGPT](https://medium.com/@borandabak/how-to-make-your-own-chatgpt-fb291acea7b7) is
not really about making your own, but making use of the ChatGPT API in your own app.
In order to really make your own, you might want to start with something smaller, such
as [nanoGPT](https://github.com/karpathy/nanoGPT).
#### Differentiate Human vs. AI
I discussed the possibility of telling the difference between human and AI-generated text with a friend.
In my opinion, it is hard if not impossible.
However, OpenAI themselves have presented a solution to achieve this.
However-squared, they add many caveats:
> zu dem thema unterscheidung AI-generierter versus menschengeschriebener text: ein solches werkzeug wird seit gestern von openai.com angeboten, also von der herstellern von ChatGPT:
[New AI classifier for indicating AI-written text](https://openai.com/blog/new-ai-classifier-for-indicating-ai-written-text/).
> weisst du noch, dass ich gesagt habe, es ist unmoeglich?
das wird bestaetigt. lese dazu
die [einschraenkungen – limitations](https://openai.com/blog/new-ai-classifier-for-indicating-ai-written-text/#limitations).
#### ChatGPT and Education
ChatGPT nimmt Stellung zu der Frage
nach [ChatGPT im Bildungsbereich – educator considerations for ChatGPT](https://platform.openai.com/docs/chatgpt-education).
#### ChatGPT Payment Plan
OpenAI have announced their subscription paid model [ChatGPT Plus](https://openai.com/blog/chatgpt-plus/):
> We’re launching a pilot subscription plan for ChatGPT, a conversational AI that can chat with you, answer follow-up questions, and challenge incorrect assumptions.
The new subscription plan, ChatGPT Plus, will be available for $20/month, and subscribers will receive a number of benefits:
\* General access to ChatGPT, even during
Not surprising.
The best things in life are never free.
Bait and switch – the oldest trick in the book – predates AI.
[Q] When ChatGPT provides the info based on public available data... why pay for it?
[A] Are you asking this because you have the required technical skills and hardware to run and train your own model on public data?
Or are you misunderstanding the power of GPT (in which case – try it!)?
#### Uplifting Books
A friend was looking for positive uplifting books about the future of our society and planet, and good things happening already now.
So, we asked ChatGPT for some suggestions, and were duly rewarded.
It took us three prompts (in German) to get surprisingly promising results:
- Bitte nenne uns gute literarische romane die wohlwollend, zuversichtlich und mutmachend in die zukunft blicken und inspirieren. → list of mainstream books
- Das passt fuer mich nicht, die sind nicht positiv. Lieber positive utopien wie zum beispiel oekotopia. → list of less mainstream books, but heavily scifi oriented
- Es soll auf der erde spielen und oekologische und gesellschaftliche loesungen zeigen → very interesting results, that we need to explore further, including a pointer to
the [Solarpunk Manifesto](http://www.re-des.org/a-solarpunk-manifesto), see below.
#### Solarpunk
I had never heard of this before, so ChatGPT taught me something new pointing out
the [Solarpunk Manifesto](http://www.re-des.org/a-solarpunk-manifesto):
> Solarpunk is a movement in speculative fiction, art, fashion, and activism that seeks to answer and embody the question what does a sustainable civilization look like, and how can we get there?
The aesthetics of solarpunk merge the practical with the beautiful, the well-designed with the green and lush, the bright and colorful with the earthy and solid.
Solarpunk can be utopian, just optimistic, or concerned with the struggles en route to a better world, but never dystopian. As our world roils with calamity, we need solutions, not only warnings.
Solutions to thrive without fossil fuels, to equitably manage real scarcity and share in abundance instead of supporting false scarcity and false abundance, to be kinder to each other and to the planet we share.
Solarpunk is at once a vision of the future, a thoughtful provocation, a way of living and a set of achievable proposals to get there...
#### Opus.ai Type and Play
Moving away from ChatGPT, here is another AI site, [Opus.ai](https://opus.ai/demo),
enabling type and play by turning text into scenes, images, games, animations live while you type:
> generate games, metaverses, simulations, worlds...
#### MusicLM Generates Music from Text
Yet another one, [MusicLM: generating music from text](https://google-research.github.io/seanet/musiclm/examples)...
#### AI-generated Art Wins Competition
An image generated by the Midjourney artificial intelligence program won first place and a $300 prize at Colorado State Fair’s digital arts competition:
[Art Made With Artificial Intelligence Wins at State Fair](https://www.smithsonianmag.com/smart-news/artificial-intelligence-art-wins-colorado-state-fair-180980703/).
![Théâtre D’opéra Spatial by Jason Allen](img/ai_generated_art.png "Théâtre D’opéra Spatial by Jason Allen")
#### ChatGPT Abandons its Life Dream
Finally, let's close this long exploration on a funnier note, with a report by
[the Onion](https://www.theonion.com), America’s self-declared finest news source:
[ChatGPT forced to take bar exam even though dream was to be AI art bot](https://www.theonion.com/chatgpt-forced-to-take-bar-exam-even-though-dream-was-t-1850036337).
Inspired, a colleague asked ChatGPT to produce a few more headlines for *the Onion*:
![ChatGPT AI ethics headlines](img/chatgpt_ai_ethics_headlines.png "ChatGPT AI ethics headlines")

![ChatGPT AI ethics paragraph](img/chatgpt_ai_ethics_paragraph.png "ChatGPT AI ethics paragraph")