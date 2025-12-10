---
post_number: "0870"
title: "BIM 360 Glue REST API Authentication Using Python"
slug: "py_rest_glue"
author: "Jeremy Tammik"
tags: ['parameters', 'python', 'revit-api', 'views']
source_file: "0870_py_rest_glue.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0870_py_rest_glue.html"
---

### BIM 360 Glue REST API Authentication Using Python

I provided an overview of the
[BIM 360 Glue REST API and SDK](http://thebuildingcoder.typepad.com/blog/2012/12/the-bim-360-glue-viewer-and-rest-api.html) last
Friday and hinted at upcoming further exploration.
Well, here it is already.

Due to Autodesk University and the world-wide developer conferences, I had to skip my last education day, but this stuff was too exciting to wait any longer :-)

So, unwilling to go for any length of time without trying out something new, I played a bit with the Glue API anyway.

For fun, I will describe here stepping through the exploration of the Glue authentication process completely manually, making use of the Python programming language and a handy library which probably provides an easier access to the REST API than you imagined possible.
Here are the steps:

1. [Python and requests](#1)
2. [Get the Google page](#2)
3. [Access BIM 360 Glue](#3)
4. [Adding authentication](#4)
5. [Timestamp and MD5 digest](#5)
6. [More login credentials](#6)
7. [Successful authentication](#7)

#### Python and Requests

Looking for an easy way to manually interact with REST, I immediately turned to Python and found the
[requests](http://pypi.python.org/pypi/requests/0.4.1) library,
which describes itself as an 'awesome Python HTTP library that's actually usable'.
I would agree that is a fair assessment.

#### Get the Google Page

Here is an example showing how simple it is to issue an HTTP request from scratch, including launching the Python interpreter from the command line; basically, it uses one single line of code, calling the method requests.get with the desired URL:

```
$ python
Python 2.7.2 (default, Jun 20 2012, 16:23:33)
[GCC 4.2.1 Compatible Apple Clang 4.0 (tags/Apple/clang-418.0.60)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import requests
>>> r = requests.get('http://google.com')
>>> print r
<Response [200=""]>
>>> r.headers['content-type']
'text/html; charset=ISO-8859-1'
>>> r.content
'<!doctype html>
<html itemscope="itemscope" itemtype="http://schema.org/WebPage">
<head>
<meta content="Search the world\'s information, including webpages, images, videos and more. Google has many special features..."
```

The REST response 200 is a typical
[HTTP status code](http://restpatterns.org/HTTP_Status_Codes) and
means OK.

#### Accessing BIM 360 Glue

Ok, so requesting the Google home page is simple.
Let's try accessing BIM 360 in a similar manner.

Trying to access something, e.g. query the model services, immediately reacts:

```
>>> u1='https://bim360.autodesk.com/api/model/v1/info.json'
>>> r = requests.get(u1)
>>> print r
<Response [400=""]>
```

Oops.
Response codes in the 400 range indicate client errors.
400 itself stands for
[bad request](http://restpatterns.org/HTTP_Status_Codes/400_-_Bad_Request).

Yes, of course, we need some authentication!

Time to start looking at the documentation.
First, find out where it can be found at all.
The starting point
[bim360.autodesk.com/api](https://bim360.autodesk.com/api) redirects
us to
[bim360.autodesk.com/api/doc/index.shtml](https://bim360.autodesk.com/api/doc/index.shtml),
providing human readable documentation and a link to the
[web services API documentation](https://bim360.autodesk.com/api/doc/doc_api.shtml).

#### Adding Authentication

So, let's authenticate ourselves:

Looking at the Glue
[web services API documentation](https://bim360.autodesk.com/api/doc/doc_api.shtml) on
[creating a signed request](https://bim360.autodesk.com/api/doc/doc_api.shtml#createsigned),
this requires some interesting bits and pieces besides the basic information, which consists of

- Company id- API key- API secret

The API key and secret need to be requested from Autodesk.
Currently, there is no official developer program running for Glue.
You can however buy a normal user account and ask for additional developer access based on that.

As we can see from the documentation, in addition to the API key and secret, plus the normal user account login credentials, the authentication requires a timestamp, more precisely a Unix epoch timestamp using GMT time, the number of seconds since the Unix epoch, January 1 1970 00:00:00 GMT.

The API key and secret are concatenated with the timestamp and encoded using an MD5 cryptographic hash to create a signature, which also has to be sent with the request.

#### Timestamp and MD5 Digest

Luckily, Python can easily support us in providing the timestamp and signature components.

The timestamp can be generated like this using the time module:

```
import time
def expires():
  '''return a UNIX style timestamp representing 5 minutes from now'''
  return int(time.time()+300)
```

The Python Standard Library Cryptographic Services includes the MD5 message digest algorithm 'md5', so that is also easily taken care of.

Following the example given in the Glue API documentation, I created the concatenation and digest of the following items:

- API Key: ddbf3f51b3824ecbb824ae4e65d31be4- API Secret: 12345678901234567890123456789012- UNIX Timestamp: 1305568169 - (5/16/2011 5:50:36 PM)

Here is the code doing that by hand, interacting with the interpreted environment:

```
>>> key='ddbf3f51b3824ecbb824ae4e65d31be4'
>>> secret='12345678901234567890123456789012'
>>> timestamp='1305568169'
>>> s=key+secret+timestamp
>>> s
'ddbf3f51b3824ecbb824ae4e65d31be4123456789012345678901234567890121305568169'
>>> import md5
>>> signature=md5.new(s)
>>> print signature
<md5 HASH="" object="" @="" 0x10c0b5d30="">
>>> print signature.hexdigest()
b3298cf0b4dc88450d00773b4449ba51
```

The hexadecimal digest exactly matches the signature string listed in the Glue documentation example, so we seem to be on the right track so far.

#### More Login Credentials

Studying the documentation further, we end up at the nitty-gritty internals of the
[Security Service: Login](https://bim360.autodesk.com/api/security/v1/login/doc) request,
specifying the following full list of required parameters:

- format- login\_name- password- company\_id- api\_key- api\_secret- timestamp- sig

Actually, I intuitively fixed an error or two when transferring this list; e.g. the secret was not mentioned at this point, and the timestamp has a wrong description associated with it.
So do what every programmer always has to do: ignore the documentation (but only some of it!), trust your own insight, take everything with a grain of salt, and use your brains, intuition and good taste.

By the way, the user name and password required here are the Autodesk id single sign-on credentials, also known as SSO, formerly Autodesk unique login or AUL.

I initially tried to use a GET request and was kindly informed by a suitable error message that I should be using POST instead.

#### Successful Authentication

I ran into a couple of other not unexpected issues as well, and finally ended up with this method to construct the authentication POST request:

```
url = 'https://bim360.autodesk.com:443/api/security/v1/login.json'

def bim_360_glue_authenticate( login_name, password, company_id, api_key, api_secret ):
  timestamp = str(int(time.time()))
  sig=md5.new(api_key + api_secret + timestamp).hexdigest()
  data={
    'login_name' : login_name,
    'password' : password,
    'company_id' : company_id,
    'api_key' : api_key,
    'api_secret' : api_secret,
    'timestamp' : timestamp,
    'sig' : sig
  }
  r = requests.post(url, data=data)
  print r.status_code
  print r.headers['content-type']
  print r.content
```

This call succeeds and prints:

```
>>> bim_360_glue_authenticate( ... )
200
application/json; charset=UTF-8;
{"auth_token":"b61d3ec10a7042cf884806e4e5a55601","user_id":"b2409a28-08b4-4bd4-a935-a6e33d5b030d"}
```

Again, 200 means OK, i.e. success.
Hooray!

Cool, huh?

There may be easier ways to achieve this, but hardly more instructive :-)

And as you can see, the interactive Python environment and rich library support really help a lot!

The resulting code is also pretty succinct, considering we are starting from absolute zero here.

How long would you have needed to explore and implement this in a compiled environment?

# Cloud and Mobile

### BIM 360 Glue SDK, REST API, and Authentication

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

Hi.
I just published two articles related to BIM 360 Glue:

- [An overview of the SDK components and the REST API.](http://thebuildingcoder.typepad.com/blog/2012/12/the-bim-360-glue-viewer-and-rest-api.html)- [An example of manually authenticating in the REST API using Python.](http://thebuildingcoder.typepad.com/blog/2012/12/bim-360-glue-rest-api-authentication-using-python.html)

I hope these prove useful not only for BIM 360 Glue specifically, but also to understand how to use REST in general, and what a powerful tool Python and its huge range of libraries can provide, especially for manual experimentation in this area.