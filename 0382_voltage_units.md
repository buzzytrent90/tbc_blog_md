---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.854069'
original_url: https://thebuildingcoder.typepad.com/blog/0382_voltage_units.html
post_number: 0382
reading_time_minutes: 3
series: general
slug: voltage_units
source_file: 0382_voltage_units.htm
tags:
- csharp
- elements
- filtering
- parameters
- revit-api
title: Voltage Units
word_count: 578
---

### Voltage Units

The last day before DevCamp!

I am happy to note that the articles on
[grabbing a webcam image](http://thebuildingcoder.typepad.com/blog/2010/06/grabbing-an-internet-webcam-image.html) and
[displaying it on a Revit element face](http://thebuildingcoder.typepad.com/blog/2010/06/display-webcam-image-on-building-element-face.html) were
so popular ... I will be demonstrating that and other samples using the
[Idling event](http://thebuildingcoder.typepad.com/blog/2010/04/idling-event.html) and
[modeless dialogues](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html) at DevCamp.

Meanwhile, here is an interesting little item that my colleagues Jorgen Dahl and Martin Schmid (who by the way published a
[book on AutoCAD MEP 2010](http://www.amazon.com/exec/obidos/tg/detail/-/1439057664/ref=ord_cart_shr?_encoding=UTF8&m=ATVPDKIKX0DER&v=glance))
dug up, another nugget of information on the topic of Revit database units, an area that we looked at repeatedly in the past:

- [Units](http://thebuildingcoder.typepad.com/blog/2008/09/units.html).
- [Formatting unit strings](http://thebuildingcoder.typepad.com/blog/2008/11/formatting-units-strings.html).
- [Unit types and format options](http://thebuildingcoder.typepad.com/blog/2009/01/unit-types-and-format-options.html).
- [Unit conversion](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html).
- [Unit Suffix and the ProjectUnit SDK sample](http://thebuildingcoder.typepad.com/blog/2009/10/unit-suffix-and-the-projectunit-sdk-sample.html).
- [DisplayUnitType and UnitSymbolType](http://thebuildingcoder.typepad.com/blog/2009/10/dut-versus-ust.html).
- [Parameter filter units](http://thebuildingcoder.typepad.com/blog/2009/12/parameter-filter-units.html).

Martin is an expert in the MEP domain, so one area of interest to him is the following:

**Question:** What is the internal unit used by Revit for voltage?
In the API, the ElectricalSystem.Voltage property returns a value that is about 10.76391 times the expected value in Volts, i.e., I get 1291.669 instead of 120 V.
The same applies to the TrueLoad property.
So I assume Revit is using some non-standard unit for voltage?

**Answer:** Electrical potential can be defined using the following more fundamental units:

(Length2 \* Mass) / (Time3 \* Current).

In Revit, this formula makes use of the following units:

- Length: feet- Mass: kg- Time: s- Current: A

The unit Volt is defined in the same way, of course, but uses the
[SI unit](http://nostalgia.wikipedia.org/wiki/SI_unit) meters
instead of feet, as specified by the International System of Units SI.

As a result, to convert the Revit internal database voltage unit to Volts, you have to multiply it with 0.3048^2.

It would be nice to upgrade the internal Revit database unit to use meters as well, but the problem is that there are a lot of data fields on elements in Revit that dont know if they are just a number or a length.
We are working on it, though.

Every unit in Revit knows what base units it is built of, and data is stored in that format.
The base units used internally by Revit are currently:
```csharp
enum BaseUnit
{
  BU\_Length = 0,         // length, feet (ft)
  BU\_Angle,              // angle, radian (rad)
  BU\_Mass,               // mass, kilogram (kg)
  BU\_Time,               // time, second (s)
  BU\_Electric\_Current,   // electric current, ampere (A)
  BU\_Temperature,        // temperature, kelvin (K)
  BU\_Luminous\_Intensity, // luminous intensity, candela (cd)
  BU\_Solid\_Angle,        // solid angle, steradian (sr)
};
```

Many thanks to Jorgen and Martin for this useful background information!