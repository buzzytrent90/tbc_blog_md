---
post_number: "1273"
title: "Wall Elevation Profiles in The Building Coder Samples"
slug: "wall_elevation_profile"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'transactions', 'walls']
source_file: "1273_wall_elevation_profile.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1273_wall_elevation_profile.html"
---

### Wall Elevation Profiles in The Building Coder Samples

Last week, I presented a stand-alone command by my colleague Katsuaki Takamizawa to
[retrieve wall elevation profiles](http://thebuildingcoder.typepad.com/blog/2015/01/getting-the-wall-elevation-profile.html).

His implementation provides a nice little example of using the
[ExporterIFCUtils.SortCurveLoops method](http://thebuildingcoder.typepad.com/blog/2015/01/exporterifcutils-curve-loop-sort-and-validate.html) and
differentiates between outer and inner loops.

It can be used in almost exactly the same way as the
[first wall elevation profile](http://thebuildingcoder.typepad.com/blog/2008/11/wall-elevation-profile.html) implementation
presented in 2008.

After publication, Katsu-san and I discussed what to do next and unearthed a couple of interesting aspects:

**Response:** Thank you very much for publishing the sample code.

I migrated the original wall profile sample. There were a number of places that needed to be changed for this migration.

I see many other Building Coder samples. It would be ideal to migrate them all, but this would require a good effort.

Maybe we can migrate frequently used samples and have developers migrate the rest by themselves as needed?

**Answer:** The Building Coder samples are already completely migrated, and the most up-to-date version is always available from
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples).

I am not thinking of re-migrating the old code, since that is already done, but of replacing the current migrated version in The Building Coder samples by your new implementation, since you say it is simpler and better.

**Response:** I was not aware that all the old samples have already been migrated and put on GitHub. That is really great!

I migrated the old code from the original blog post. I am afraid that some developers might be doing the same.

Is this link visible and accessible from your blog?

**Answer:** Basically, I keep pointing out that the most up-to-date version is always available from
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples), as above.

I followed your suggestion and integrated the new implementation in parallel with the original one.

The main entry point now looks like this:

```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    // Choose which implementation to use.

    bool use\_execute\_2 = true;

    return use\_execute\_2
      ? Execute2( commandData, ref message, elements )
      : Execute1( commandData, ref message, elements );
  }
```

You can switch between the two by toggling the value of the Boolean variable `use_execute_2` in the debugger.

Here is the result of running the new implementation in the sample model you provided:

![Wall elelvation profiles produced by the new implementation](img/wall_elev_profiles_2.png)

The result of the original implementation looks like this:

![Wall elelvation profiles produced by the original implementation](img/wall_elev_profiles_1.png)

I published the version including the new command implementation as
[release 2015.0.116.7](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.116.7).

The integration of the two required changing the transaction mode of the external command from automatic to manual.

This really ought to be done in all of the external command samples.

Another overdue task is to completely remove all obsolete API usage.
I last took a stab at it in November,
[eliminating more obsolete API usage](http://thebuildingcoder.typepad.com/blog/2014/11/determining-intersecting-elements-and-continued-futureproofing.html#2).

I also want to update the copyright message to reflect the new year 2015.

In fact, I just completed the latter right away, and published the result as
[release 2015.0.116.8](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.116.8).

Many thanks to Katsu-san for this useful discussion and clarification!