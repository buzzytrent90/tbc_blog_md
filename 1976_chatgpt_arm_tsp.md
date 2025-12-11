---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:17.170863'
original_url: https://thebuildingcoder.typepad.com/blog/1976_chatgpt_arm_tsp.html
post_number: '1976'
reading_time_minutes: 9
series: general
slug: chatgpt_arm_tsp
source_file: 1976_chatgpt_arm_tsp.md
tags:
- levels
- parameters
- revit-api
- sheets
- views
- windows
title: Chatgpt Arm Tsp
word_count: 1813
---

### Exploring ARM, ChatGPT, Nairobi and the TSP
I am still in Nairobi, Kenya, getting to know the lay of the land and team here, as well as setting up my new computer; meanwhile, OpenAI ChatGPT is making global waves and first explorative steps are being made to use it with the Revit API:
- [First outings in Nairobi](#2)
- [ChatGPT in Kiswahili](#3)
- [ChatGPT for music and poetry mimicry](#4)
- [ChatGPT for Revit API I](#5)
- [ChatGPT for Revit API II](#6)
- [ChatGPT for Revit interaction](#7)
- [Teaching ChatGPT about the birds and the bees](#8)
- [ChatGPT limitations and improvement](#9)
- [ChatGPT acrostic poem](#10)
- [VS2022 on MacBook Pro M1 with Parallels](#11)
- [Chasing the travelling salesman](#12)
- [LinkedIn comments](#13)
#### First Outings in Nairobi
My colleagues warn me of the dangers of moving around freely here, so I am being careful.
Not a single bad experience so far, though. All good.
Lots of security all around, such as security checks entering every building.
The people I meet in the street are either neutral or friendly, though, and I feel happy and safe.
So far, besides working, I went for a hike
to [Mount Longonot](https://www.alltrails.com/trail/kenya/rift-valley/mount-longonot) on
Saturday and took a long walk into the centre of town on a Sunday;
however, most things were closed and the city was pretty empty with very little going on that day.
Here are my [pictures in and around Nairobi](https://flic.kr/s/aHBqjAiJv1) so far.
#### ChatGPT in Kiswahili
[ChatGPT](https://chat.openai.com/chat) is currently a very hot topic among techies, so I created an account and checked it out:
Jeremy: can you speak swahili?
> Huna! Naunga mkono nyumbani kwako. Samahani, sijawahi kusoma Swahili. Je, unaweza kusema kwa Kiingereza?
Jeremy: oh dear. i am afraid i cannot.
> Asante sana. Inawezekana kukutumia lugha ya Kiingereza katika mazungumzo yetu. Kwamba ni ajabu kwa sababu Swahili ni lugha nne ya makundi ya Bantu na inatumika sana katika Afrika Mashariki. Kama una nia njema, unaweza kusoma zaidi kuhusu Swahili kupitia tovuti za mtandao.
Jeremy: i wish i could.
> Kama unahitaji msaada zaidi, basi wasiliana nami. Napenda kusaidia katika mambo yanayohusiana na lugha za Afrika.
Jeremy: can you translate all that you said for me, please?
> Sure! Here's my previous response translated into English:
"Hello! I extend my hand to your home.
Sorry, I haven't read Swahili.
Well, can you speak English?
Thank you very much.
It is possible to use the English language in our conversation.
It is strange because Swahili is one of the four languages of the Bantu group and is widely used in East Africa.
If you are interested, you can learn more about Swahili through websites.
If you need more help, please contact me.
I like to help with things related to African languages."
I was surprised how much sense that made.
Since some people I talk with make less sense, I would say the Turing test is passed.
#### ChatGPT for Music and Poetry Mimicry
Another interesting example was brought
by Raphaël Millière [@raphaelmilliere](https://twitter.com/raphaelmilliere)
on [twitter](https://twitter.com/raphaelmilliere/status/1598469100535259136?t=upd),
triggering a lot of response and discussion:
> I asked ChatGPT to rewrite Bohemian Rhapsody to be about the life of a postdoc, and the output was flawless...
Check out the [full thread with comments](https://twitter.com/raphaelmilliere/status/1598469100535259136?t=upd) for details.
#### ChatGPT for Revit API I
The obvious next step for this community is to try out its abilities helping with Revit API questions.
My colleague [Jacob Small](https://twitter.com/JacobWSmall) tried that out
and [documented his steps on twitter](https://twitter.com/JacobWSmall/status/1600477115853737984):
- Ok, so I have an option here... I could 'try again' which might make things fail worse off as in my first go-round that resulted in brand new stuff. Or I could try and get it to improve by telling where the error was. Let's go for the later as design of anything needs iteration.
- Again, trying to guide it like a user would guide me in a text chat (which I happen to have a few years of experience with). Already know that it's going to fail at the Id property of the undefined level variable, but we'll get there when we get the error.
- Good(ish) news, is that the new code returned a new error (new errors are good things as it often means we fixed something).
- Bad news is that it is a 'calling a variable which hasn't been defined'. The chatbot appears to be copy/pasting line by line instead of sections of code.
- In the chatbot's defence, I've done this myself a few times by not ensuring the user had robust enough boilerplate code to start with. I'm going to switch to not bothering with running this, as `__revit__` also hasn't been defined. No longer a user guided chat though.
- Well, it's thinking a lot... Am I not playing by the chatbot's rules?
- Ok... it doesn't take directions as well as a software engineer on the Dynamo or Revit team. Let's try the previous prompt like a user would, and just reply with the errors for another few minutes or until I get bored.
- And yes, I could have written this from scratch by now.
You need to look at
the [original thread including all the screen snapshots](https://twitter.com/JacobWSmall/status/1600477115853737984) to
see the details of what he's talking about.
#### ChatGPT for Revit API II
Bird Tools also tried out and
documented [using ChatGPT for Revit API](https://www.linkedin.com/posts/bird-tools_openai-revit-code-activity-7005369990614798336-vqXP) in
a LinkedIn post, with many more interesting comments.
#### ChatGPT for Revit Interaction
Last but not least concerning the Revit API, our active contributor [Chuong Ho](https://github.com/chuongmep) started exploring the possibility
to [integrate ChaGPT interactively into Revit](https://www.linkedin.com/posts/chuongmep_dynamo-gpt3-openai-activity-7007387062999928832-AyL5).
Here are the first few comments on that effort, with many more following since then:
- Chuong Ho: Do you know, Chat GPT3, OpenAI will be coming soon Dynamo BIM 🤣. I'm still figuring out how to make them good at the moment, it's surprisingly accurate, but sometimes it's as innocent as a toddler learning to speak. I'd love to have comments right now.
- Maycon Freitas: Awesome, Chuong Ho!! I tried to create a chatbot to integrate Revit and WhatsApp using Dynamo some time ago (but a dumb one, not smart as GPT3) – https://www.linkedin.com/feed/update/urn:li:activity:6671175468063973376/ – and also using Forge + Whatsapp – https://www.linkedin.com/feed/update/urn:li:activity:6851904940378480640/...
- Chuong Ho: So, as you might already know, my suggestion is: Chatbots to execute some tasks, extract data e so on. It would be a really good feature to implement.
- Luisa Cypriano: Great one Chuong Ho :) Did you publish openly the pack already?
- Chuong Ho: Still not yet, I working with control parameters input to help respond better for AEC
#### Teaching ChatGPT About the Birds and the Bees
For an example beyond the Revit API where ChatGPT really excels and surpasses all expectations, check out my namesake Jeremy Pinto's 20-minute AI training implementation article
on [teaching ChatGPT about the birds and the bees](https://www.jerpint.io/blog/chatgpt).
#### ChatGPT Limitations and Improvement
Leon Furze takes his analysis one step further and explores aspects of writing essays, disrupting education, ChatGPT limitations and improvement possibilities in his article
on [prompt whispering: getting better results from ChatGPT](https://www.linkedin.com/pulse/prompt-whispering-getting-better-results-from-chatgpt-leon-furze).
#### ChatGPT Acrostic Poem
Finally, to round this off, demand seems to be pretty high now;
I wanted to ask for a suitable French idiom to wish Cyrille happy holidays, and was rewarded with
an [acrostic poem](https://en.wikipedia.org/wiki/Acrostic):
C: ChatGPT is currently down

H: Huge demand has caused the site to crash

A: All users will have to wait

T: Time is needed for the servers to catch up

G: Go grab a coffee and check back soon

P: Patience is key in this situation

T: Trust that the team is working hard to fix it up![Acrostic poem by ChatGPT](img/chatgpt_down.png "Acrostic poem by ChatGPT")
So, enough of ChatGPT for now.
#### VS2022 on MacBook Pro M1 with Parallels
Meanwhile, moving on from this inexhaustible and rapidly evolving topic, I received and started setting up my new PC, a MacBook Pro M1 with
an [Apple M1 Pro ARM chip](https://en.wikipedia.org/wiki/Apple_M1).
So, with no Intel hardware, can I use it for Revit at all?
Well, I gave up trying to install Visual Studio 2019, which I had previously been using.
The final decision was prompted by a thread
on [running Visual Studio 2019 on Windows 11 on M1 Mac](https://forum.parallels.com/threads/running-visual-studio-2019-on-windows-11-on-m1-mac.355149),
since it ends with the following unequivocal advice:
> I genuinely recommend just using VS2022; it's actually ARM-native, so it will run a lot faster and with far less issues.
I installed Parallels Desktop 18 for Mac, Windows 11, Revit 2023 and Visual Studio 2022, in that order, and am now able to open and compile a sample Revit add-in, launch Revit in the Visual Studio debugger and successfully trigger a breakpoint:
![Debug Revit add-in on ARM](img/vs2022_debug_addin.png "Debug Revit add-in on ARM")
No further testing performed yet, though.
#### Chasing the Travelling Salesman
Finally, completely unrelated to ChatGPT and Revit,
[Diego Vicente](https://diego.codes/about/) published a new approach to approximately tackle the well-known hard NP-complete challenge,
successfully [using self-organizing maps to solve the traveling salesman problem](https://diego.codes/post/som-tsp/).
![Self-organizing maps approximating the traveling salesman problem for Italy](img/tsp_som_italy.png "Self-organizing maps approximating the traveling salesman problem for Italy")
#### LinkedIn Comments
I'm only adding the first two of them... can't keep on updating forever...
[Igor KhisamievView asks](https://www.linkedin.com/feed/update/urn:li:activity:7008498173014888449?commentUrn=urn%3Ali%3Acomment%3A%28activity%3A7008498173014888449%2C7008922401031741440%29&dashCommentUrn=urn%3Ali%3Afsd_comment%3A%287008922401031741440%2Curn%3Ali%3Aactivity%3A7008498173014888449%29):
> Hi, Jeremy. How are you testing your addins on MacBook? Revit doesn’t work on MacBook 🤔 or is Windows installed?
> \*\*Answer:\*\* I use [Parallels to run Windows on MacOS](https://www.parallels.com/products/desktop)
[Göker Malik Altuntaş says](https://www.linkedin.com/feed/update/urn:li:activity:7008498173014888449?commentUrn=urn%3Ali%3Acomment%3A%28activity%3A7008498173014888449%2C7008946537527435264%29):
> I am literally amazed by #ChatGPT after seeing it can actually write a functioning Revit plug-in. Thank you for fantastic sharings 👍