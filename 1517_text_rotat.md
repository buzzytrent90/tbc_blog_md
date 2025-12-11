---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.2
content_type: code_example
optimization_date: '2025-12-11T11:44:16.200513'
original_url: https://thebuildingcoder.typepad.com/blog/1517_text_rotat.html
post_number: '1517'
reading_time_minutes: 8
series: general
slug: text_rotat
source_file: 1517_text_rotat.md
tags:
- elements
- python
- references
- revit-api
- rooms
- sheets
- views
title: Text Rotat
word_count: 1564
---

### TextNote Rotation, Forge DevCon, TensorFlow and Keras
The Forge DevCon developer conference has been happily united with Autodesk University, text note rotation is easy, and I continued my deep learning exploration for implementing a Revit API question answering system:
- [Forge DevCon at AU](#2)
- [Setting `TextNote` rotation](#3)
- [TensorFlow and Keras](#4)
- [Updating restricted Python packages](#5)
- [Rules of machine learning](#6)
#### Forge DevCon at AU
A match made in heaven:
the [Forge DevCon](https://forge.autodesk.com/DevCon) developer
conference has been happily united
with [Autodesk University](http://au.autodesk.com/las-vegas)
A lot of synergy is gained connecting the two.
![Forge DevCon at AU](img/forge_devcon_at_au.png)
The last DevCon in 2016 was held in San Francisco and was a great success.
We plan to make it bigger and better still this year, and moving it to Las Vegas for 2017 will help.
It will take place on November 13-14,
i.e., [Monday and Tuesday of the Autodesk University week](http://adndevblog.typepad.com/cloud_and_mobile/2017/01/forge-devcon-change-of-date-and-venue.html).
In previous years, these were the days on which we hosted the DevDay and DevLab conference and open house support hackathon for application developers.
This is a good thing.
Somewhat selfishly, it means less travel for those of us coming across from Europe.
It will also make AU a more compelling, developer-centric event.
Now nobody has to choose between DevCon and DevDays + AU. It’s all together in one place.
#### Setting TextNote Rotation
Rotating a text note is very easy, both during creation and when modifying existing elements.
One little complication for existing add-ins is introduced by fact that the approach changed with
the [`TextNote` API overhaul in Revit 2016](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html#4.02):
\*\*Question:\*\* In Revit 2015, I used the method:
```csharp
TextNote text = doc.Create.NewTextNote(
view, origin, baseVec, upVec, lineWidth,
TextAlignFlags.TEF_ALIGN_CENTER
| TextAlignFlags.TEF_ALIGN_MIDDLE,
strText );
```
for creating a text note.
As this method has been replaced with `TextNote.Create`, I can create a text note using:
```csharp
TextNote text = TextNote.Create( doc,
view.Id, origin, strText, texttypeID );
```
But how do I set its rotation when creating it?
Also, how do I change the rotation of existing text notes?
\*\*Answer:\*\* To rotate the `TextNote` as it is placed, one of the overloads for `TextNote.Create` uses `TextNoteOptions` and its `Rotation` option:
```csharp
TextNoteOptions tno = new TextNoteOptions();
tno.Rotation = 0.5 \* Math.PI; // in Radians
tno.TypeId = texttypeID; // need to include text type
TextNote text = TextNote.Create( doc,
view.Id, origin, strText, tno );
```
\*\*Response:\*\* This is how I managed to create the rotated text aligned with existing curve based elements:
```csharp
// get the curve of the element
Curve curve = ( (LocationCurve) elem.Location )
.Curve;
// get start and end point of pipe
XYZ PipeEnd = curve.GetEndPoint( 1 );
XYZ PipeStart = curve.GetEndPoint( 0 );
// get vector representing the pipe
XYZ v = PipeEnd - PipeStart;
double angle = Math.Atan2( v.Y, v.X );
var testOptions = new TextNoteOptions()
{
HorizontalAlignment = HorizontalTextAlignment.Center,
Rotation = angle,
TypeId = texttypeID
};
var text = TextNote.Create( doc, view.Id,
origin, strText, testOptions );
```
![Rotate text](img/rotate_text.png)
#### TensorFlow and Keras
Continuing my superficial browsing of various open source possibilities on which to base a question answering system for the Revit API add-in programming, I now bumped
into [TensorFlow](https://en.wikipedia.org/wiki/TensorFlow)
([web site](https://www.tensorflow.org/),
[GitHub repo](https://github.com/tensorflow/tensorflow)).
It converts the learning matter to vectors and computes similarities, learning best matches by assigning weight using various algorithms.
Here are a bunch of interesting TensorFlow tutorial and other related texts that I would all like to work through in greater depth anon:
- [MNIST tutorial for beginners](https://www.tensorflow.org/tutorials/mnist/beginners/index) and [advanced](https://www.tensorflow.org/tutorials/mnist/pros/index)
- [Vector Representations of Words](https://www.tensorflow.org/tutorials/word2vec/)
- [Recurrent Neural Networks](https://www.tensorflow.org/tutorials/recurrent/)
- [Sequence-to-Sequence Models](https://www.tensorflow.org/tutorials/seq2seq/)
- [TensorKart: self-driving MarioKart with TensorFlow](http://kevinhughes.ca/blog/tensor-kart)
They also led to further papers and the [Keras](https://keras.io) system built on top of it:
- [SyntaxNet](https://www.tensorflow.org/tutorials/syntaxnet/) is a neural-network Natural Language Processing framework for TensorFlow.
- [Question Answering Using Deep Learning](https://cs224d.stanford.edu/reports/StrohMathur.pdf) – neural network variants are becoming the dominant architecture for many NLP tasks. This project applies several deep learning approaches to question answering, with a focus on the bAbI dataset, including TensorFlow.
- [Keras: Deep Learning library for Theano and TensorFlow](https://keras.io/) – paper on [Deep Language Modelling for Question Answering using Keras](http://benjaminbolte.com/blog/2016/keras-language-modeling.html) with [GitHub repo](https://github.com/fchollet/keras/blob/master/examples/babi_rnn.py)
- [Question answering on the Facebook bAbi dataset using recurrent neural networks and 175 lines of Python + Keras](http://smerity.com/articles/2015/keras_qa.html)
- [The Facebook bAbI tasks](https://research.fb.com/downloads/babi/)
I decided to install these packages, which led to an interesting problem that took a while to sort out and seems worthwhile documenting for future reference.
#### Updating Restricted Python Packages
TensorFlow and Keras are Python based.
Installation is very straightforward and easy.
TensorFlow requires several other base packages: `six`, `funcsigs`, `pbr`, `mock`, `numpy`, `protobuf`.
Unfortunately, on my system, some of these were out of date and the system refused to allow me to update them.
For instance, calling `$ pip install tensorflow` led to a conflict due to the existence of prior versions of `six` and `numpy`.
After some research, I determined that I could resolve the issues by simply deleting the associated files.
Unfortunately, this is not permitted by the system, as described in the question
on [why `chown` reports `operation not permitted` on OS X](http://superuser.com/questions/279235/why-does-chown-report-operation-not-permitted-on-os-x).
The short description of the solution for this is given there:
> System Integrity Protection (rootless) ... boot into Recovery Mode (Cmd-R), run `csrutil disable`, then restart; to reenable, use `csrutil enable`.
It is explained in greater detail in the StackOverflow question
on [restricted folder and files in OS X El Capitan](http://stackoverflow.com/questions/30768087/restricted-folder-files-in-os-x-el-capitan):
To temporarily disable SIP (System Integrity Protection):
- Reboot
- As soon as you hear the "Mac sound" on the grey screen, press Cmd+R to enter Recovery mode
- Open Utilities > Terminal
- Run the command `csrutil disable`
- Reboot, you'll land in the normal OS with SIP disabled
- Do all the changes you'd like to do
- Reboot again
- As soon as you hear the "Mac sound" on the grey screen, press Cmd+R to enter Recovery mode
- Enable SIP with `csrutil enable`
- Reboot again
- Done
On my system, the problematic packages live in the folder
- `System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python`
You can examine their restricted status using `ls`:

```
$ ls -lhdO num*
drwxr-xr-x  36 root  wheel  restricted             1.2K May 25  2016 numpy
-rw-r--r--   1 root  wheel  restricted,compressed  1.6K Aug  1  2015 numpy-1.8.0rc1-py2.7.egg-info
```

By temporarily disabling SIP, I was finally able to move them out of the way and continue normal installation:

```
$ sudo -H pip install tensorflow
```

With TensorFlow in place, I successfully ran the standard Python install in the `keras` folder and immediately executed one of its interesting examples:

```
/a/src/deep_learning $ cd kears
/a/src/deep_learning/keras $ sudo python setup.py install
/a/src/deep_learning/keras $ cd examples
/a/src/deep_learning/keras/examples $ python babi_rnn.py
```

Keras is up and running now.
What shall I do next?
Here are two descriptions of successful experiments implementing minimal question answering systems with it that I am taking a deeper look at:
- [Deep Language Modelling for Question Answering using Keras](http://benjaminbolte.com/blog/2016/keras-language-modeling.html)
([^](/a/doc/deep_learning/keras/Deep_Language_Modeling_for_Question_Answering_using_Keras.pdf)),
[GitHub repo](https://github.com/codekansas/keras-language-modeling)
- [Question Answering Using Deep Learning](https://cs224d.stanford.edu/reports/StrohMathur.pdf)
([^](/a/doc/deep_learning/keras/Question_Answering_Using_Deep_Learning_by_Stroh_Mathur.pdf)),
[GitHub repo](https://github.com/priyank87/memn2n)
Time for me to get going putting together some Revit API related tests?
#### Rules of Machine Learning
By the way, before doing anything else, I and everybody else interested in machine learning ought to read and ponder
Martin Zinkevich's 43 [Rules of Machine Learning](http://martin.zinkevich.org/rules_of_ml/rules_of_ml.pdf),
split up into useful consecutive phases:
- Before Machine Learning
- Phase I: Your First Pipeline
- Monitoring
- Your First Objective
- Phase II: Feature Engineering
- Human Analysis of the System
- Training-­Serving Skew
- Phase III: Slowed Growth, Optimization Refinement, and Complex Models
To quote Martin:
The document is arranged in four parts:
- The first part should help you understand whether the time is right for building a machine learning system.
- The second part is about deploying your first pipeline.
- The third part is about launching and iterating while adding new features to your pipeline, how to evaluate models and training-­serving skew.
- The final part is about what to do when you reach a plateau.
- Afterwards, there is a list of related work and an appendix with some background on the systems commonly used as examples in this document.
It is strongly geared towards ranking.