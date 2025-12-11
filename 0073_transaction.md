---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.316807'
original_url: https://thebuildingcoder.typepad.com/blog/0073_transaction.html
post_number: '0073'
reading_time_minutes: 4
series: transactions
slug: transaction
source_file: 0073_transaction.htm
tags:
- csharp
- parameters
- revit-api
- rooms
- transactions
- views
title: Transactions
word_count: 784
---

### Transactions

We recently mentioned the need to
[open a transaction in an event handler](http://thebuildingcoder.typepad.com/blog/2009/01/barcelona-questions.html#open_transaction_in_event_handler).
In that case, the transaction was required in a handler for the document closed and saved events on the application object.
Here is another similar case involving the application OnDocumentNewed event.

**Question:**
I am trying to set up some shared parameters on OST\_Rooms in a ControlledApplication.
I can read and bind values from a shared parameter file, and have checked that the binding is set up correctly for the document. However, when I open the properties of a room, I don't see my parameter there.
As a little background, we are creating an application to link to a database which will get and set values in the shared parameters for each room.

**Answer:**
If you create a new document explicitly, you need to start a transaction on it.
Although in this case you are not creating the new document yourself, starting a transaction solves the problem here as well:

```csharp
void application\_OnDocumentNewed( Document doc )
{
  bool rc = doc.BeginTransaction();
  Debug.Assert( rc, "begin transaction failed" );
  if( rc )
  {
    CreateUserDefinedParameters( doc );
    rc = doc.EndTransaction();
    Debug.Assert( rc, "end transaction failed" );
  }
}
```

So, once again, you need to encapsulate the modification in a transaction.
In the context of an external command, the Revit API will
[automatically begin a transaction](http://thebuildingcoder.typepad.com/blog/2008/12/document-ismodified-property.html)
for you.
Depending on the IExternalCommand result value returned from the external command's Execute method, the transaction will be either closed or aborted.
In all other contexts, however, including all event handlers, no such transaction will be automatically opened for you, so you are responsible for doing so yourself.

Many thanks Lindsay Rader at
[Advanced Technologies Group, Inc.](http://www.atginc.com)
for raising this question and to Adam Nagy of Autodesk for the answer!

#### Rome and Cantalupo

I wrote the text above after getting hooked in a small coffee shop in Barcelona providing a wifi connection, on my way to the airport.
At least I should have been on my way, instead of sitting there blogging.
I very narrowly avoided missing my flight to Rome.
I did make it though, checking in as the last passenger before the flight was closed, and am now on a completely uninformed and purely intuitive few-hour tour of some parts of the city.
My first station here has turned out to be Viale Nino Manfredi with a fine view over the entire city.
From there I took a long walk:

- through the Palatino, almost getting lost in some fenced off areas,
- around the Arco di Constantino and Colloseum,
- through a huge protest march against the Israeli warfare on Gaza,
- around the unbelievably huge and impressive Monumento di Vittorio Emmanuele II,
- through the ruins of the Teatro Marcello,
- across Isola and Fiume Tevere,
- through Trastevere, looking for a hotel and some good food,
- before spontaneously and abruptly deciding to leave Rome immediately after all.

After driving off into the unknown I ended up in a quiet and hidden little place called Cantalupo, which may or may not mean song of wolf, next to Casperia, with no idea of where I actually am, which is a very enjoyable state to be in.
There is no hotel in this little town, so I am in a guest house where I first had to spend ten minutes trying to get any one electric light in the room to work.
My landlady is an old lady with no teeth.
She made me late supper, a delicious three-course meal from nothing, toasting some bruscetta over an open fire in the fireplace, spaghetti al'arrabiata, and cheese with some green vegetable fried in oil that she named for me but I was unable to identify or remember.

I am awakened Sunday morning at half past seven by the firing of cannons.
To celebrate the fiesta of the benediction of the animals, I am told.
This place is so fantastic ... in case you are interested, the address is Villa delle Rose, Viale Verdi 42, Cantalupo, Rieti, Italy, tel. 0765/514035.
From here I will visit Casperia, just a few kilometres away, and then maybe continue towards Perugia, Firenze, Bologna, Verona.
In case you are wondering what I am actually up to on this haphazard trip, my purpose is to teach Revit programming in Verona and learn some Italian first, with the ultimate goal of holding the training in Italian.

Well, no internet connection in Casperia, so now I am in the city of Terni, posting from the bonnet of somebody's car, hooked up to somebody's unprotected wifi.