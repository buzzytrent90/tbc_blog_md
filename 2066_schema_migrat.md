---
post_number: "2066"
title: "Schema Migrat"
slug: "schema_migrat"
author: "Jeremy Tammik"
tags: ['revit-api', 'sheets', 'views', 'windows']
source_file: "2066_schema_migrat.md"
original_url: "https://thebuildingcoder.typepad.com/blog/2066_schema_migrat.html"
---

### Tools for Extensible Storage and OAuth Auth0
Revit API tools for extensible storage and OAuth Auth0, notes on AI news, a Mac feature and electrical energy storage:
- [SchemaMigrations extensible storage lib](#2)
- [OpenAI ChatGPT deep research](#3)
- [OAuth Auth0 in a Revit add-in](#4)
- [Docling markdown generator](#5)
- [MacOS copy paste sans formatting](#6)
- [DIY open source redox flow battery](#7)
#### SchemaMigrations Extensible Storage Lib
The [atomatiq](https://www.linkedin.com/company/atomatiq/) team including Ilia Ivanov and Sergei Nefedov presents
the [SchemaMigrations library](https://github.com/atomatiq/SchemaMigrations) of comfortable tools for the Revit Extensible Storage API to make its usage similar to
the [.NET Entity Framework](https://en.wikipedia.org/wiki/Entity_Framework):
- Define your models, add them to `SchemaContext`.
- Run `Schema Migration Generator` to create migration.
- Save your models in ES and load them from ES as instances of your `Models` class, instead of only primitive objects.
For further details, check out
the [SchemaMigrations GitHub repository documentation](https://github.com/atomatiq/SchemaMigrations).
#### OpenAI ChatGPT Deep Research
Daily exciting news on AI keeps on coming and continues accelerating.
OpenAI published
a [20-minute YouTube presentation on Deep Research](https://youtu.be/YkCDVn3_wiw), allowing the LLM to use agents, Internet access and other tools for longer-lasting partially unsupervised tasks.
This functionality already launched in ChatGPT pro.
Some of this functionality was previously available for some other LLMs, e.g.,
[Claude computer use](https://thebuildingcoder.typepad.com/blog/2024/10/au-api-wishes-and-revit-20253.html#5).
Deep research is still pushing new boundaries, though, e.g., solving a much larger part
of [Humanity's Last Exam](https://lastexam.ai/).
Things are certainly moving fast, boundaries pushed and new functionality published daily, with strong competition from many sides.
#### OAuth Auth0 in a Revit Add-In
Daniel [christev7HTEL](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/15843072) Christev kindly
shared a fix to enable using `OAuth` `Auth0` in a Revit add-in in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [WebView2 throws System.Runtime.InteropServices.COMException: 'The requested resource is in use. (0x800700AA)'](https://forums.autodesk.com/t5/revit-api-forum/webview2-throws-system-runtime-interopservices-comexception-the/td-p/13291882).
in case –; like me –; you wonder what the difference is between OAuth 2.0 and Auth0, check out the StackOverflow explanation
on [OAuth 2.0 vs Auth0](https://stackoverflow.com/questions/46782725/oauth-2-0-vs-auth0).
Daniel says:
Just wanted to post some info on a bug I came across + fix.
Maybe no one will run into this problem, but it took me a while to get to the bottom of it.
It started with trying to use `Auth0` in a Revit application; the default implementation throws the exception and Revit crashes:
- System.Runtime.InteropServices.COMException: 'The requested resource is in use. (0x800700AA)'
It is possible to fix this by instantiating your client with a `WebBrowserBrowser`:

```
IBrowser browser = new WebBrowserBrowser();

client = new Auth0Client(new Auth0ClientOptions
{
Domain = domain,
ClientId = clientId,
RedirectUri = "http://localhost:3003",
PostLogoutRedirectUri = "http://localhost:3003",
Browser = browser
});
```

or even creating an implementation to try to use the default `SystemBrowser`
but I still wanted `WebView2` here.
It turns out the prickly bit of code boils down to the default environment that is created.
Generally, when you are generating a WebView2 within an application, you should call the following code:

```
string appDataFolder = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
System.IO.Directory.CreateDirectory(appDataFolder);
var env = await CoreWebView2Environment.CreateAsync(null, appDataFolderACAD);
await webView.EnsureCoreWebView2Async(env);
```

It's important to create the environment, or else the default implementation will register a data folder running out of a secured folder that can't write like Program Files..
And this was what happened to me. WebView2 in Revit was trying to create a directory at:
- C:\Program Files\Autodesk\Revit 2025\Revit.exe.WebView2\`
which aside from being an absolute eye-sore, requires admin privileges to write to, and thus the resource cannot be used.
As someone new to WebView2, this really tripped me up, so here's to hoping it will help someone else at some point too.
So I created a custom `UserEnvironmentWebViewBrowser` where the `UserDataFolder` can be set, and defaults to appdata.
(This is by and large identical to the `WebViewBrowser` implementation, with the addition of the aforementioned):

```
public class UserEnvironmentWebViewBrowser : IBrowser
{
private readonly Func _windowFactory;

private readonly bool _shouldCloseWindow;

public UserEnvironmentWebViewBrowser(Func windowFactory, bool shouldCloseWindow = true)
{
_windowFactory = windowFactory;
_shouldCloseWindow = shouldCloseWindow;
}

public UserEnvironmentWebViewBrowser(
  string title = "Authenticating...",
  string? userDataFolder = null,
  int width = 1024,
  int height = 768)
:
  this(() => new Window
{
  Name = "WebAuthentication",
  Title = title,
  Width = width,
  Height = height
})
{
  if (userDataFolder != null && Directory.Exists(userDataFolder))
  {
    UserDataFolder = userDataFolder;
  }
}

public string UserDataFolder { get; set; }
  = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);

public async Task InvokeAsync(
  BrowserOptions options,
  CancellationToken cancellationToken = default(CancellationToken))
{
  TaskCompletionSource tcs
    = new TaskCompletionSource();
  Window window = _windowFactory();
  WebView2 webView = new WebView2();
  window.Content = webView;
  webView.NavigationStarting += delegate (object? sender,
    CoreWebView2NavigationStartingEventArgs e)
  {
    if (e.Uri.StartsWith(options.EndUrl))
    {
      tcs.SetResult(new BrowserResult
      {
        ResultType = BrowserResultType.Success,
        Response = e.Uri.ToString()
      });
      if (_shouldCloseWindow)
      {
        window.Close();
      }
      else
      {
        window.Content = null;
      }
    }
  };
  window.Closing += delegate
  {
    webView.Dispose();
    if (!tcs.Task.IsCompleted)
    {
      tcs.SetResult(new BrowserResult
      {
        ResultType = BrowserResultType.UserCancel
      });
    }
  };
  window.Show();

  var webView2Environment = await CoreWebView2Environment.CreateAsync(null, UserDataFolder);
  await webView.EnsureCoreWebView2Async(webView2Environment);
  webView.CoreWebView2.Navigate(options.StartUrl);
  return await tcs.Task;
}
}
```

You can also check out the solution in
my [RevitWebView2Bug GitHub repo](https://github.com/bulgos/RevitWebView2Bug).
Shoutout to [@grahamcook](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1070920) as
I found the answer within his [post](https://forums.autodesk.com/t5/net/using-the-webviewer2-package-version-issues/m-p/12941602.
Many thanks, Daniel and Graham, for sharing this.
#### Docling Markdown Generator
[Docling](https://ds4sd.github.io/docling/) by IBM simplifies document processing, parsing diverse formats – including advanced PDF understanding – and providing seamless integrations with the gen AI ecosystem, featuring:
- Parsing of multiple document formats incl. PDF, DOCX, XLSX, HTML, images, and more
- Advanced PDF understanding incl. page layout, reading order, table structure, code, formulas, image classification, and more
- Unified, expressive DoclingDocument representation format
- Various export formats and options, including Markdown, HTML, and lossless JSON
- Local execution capabilities for sensitive data and air-gapped environments
- Plug-and-play integrations incl. LangChain, LlamaIndex, Crew AI & Haystack for agentic AI
- Extensive OCR support for scanned PDFs and images
- Simple and convenient CLI
I tested docling on an arxiv scientific paper listed in the installation instructions, and it works perfectly right out of the box with very impressive results:
- Installation: `pip install docling`
- Testing: `docling https://arxiv.org/pdf/2206.01062`
The result is a 1.6 MB markdown file `2206.01062v1.md` complete with images, tables, text, headings, the whole shebang, perfectly formatted.
#### MacOS Copy Paste Sans Formatting
After thousands of unthinking repetitions, I finally searched and found a note
on [how to copy and paste text excluding formatting on Mac](https://www.macrumors.com/how-to/copy-paste-text-no-formatting-mac/):
> In Windows, the Copy and Paste key combinations are Control-C and Control-V, respectively.
On the Mac, it's similar using the Command (⌘) key instead of Control.
You can also paste text without its original formatting.
Not knowing that this is possible on a Mac, many users paste text into a plain-format text editor to strip it of any styling before copying and pasting it again to its intended destination (that's me).
But you don't have to do that.
To directly paste the copied text elsewhere as purely plain text, use the key combination Command-Option-Shift-V and it will be automatically stripped of any formatting.
Wow.
I should have thought of checking that out years ago.
#### DIY Open Source Redox Flow Battery
Now to round off with a non-digital topic:
I was previously not aware of any DIY efforts to create battery storage and therefore excited to discover
the [Flow Battery Research Collective](https://fbrc.dev) open source project targeted at
creating a simple DIY [redox flow battery](https://en.wikipedia.org/wiki/Flow_battery).
This [20-minute video](https://spectra.video/w/6BddEiwBqRMHSbC9qBLBz9) discusses progress so far and current status.
The [roadmap](https://fbrc.dev/posts/roadmap-faq-forum/) posits a research kit in the middle of this year to help discover optimal liquid chemical components, and hopes to be able to provide a kit for creating your own working 48V battery sufficient for powering a small home by the end of the year.
![Redox flow battery](img/redox_flow_battery.jpg "Redox flow battery")

By
[Colintheone](https://avs.scitation.org/doi/10.1116/1.4983210)
[CC BY-SA 4.0](https://commons.wikimedia.org/w/index.php?curid=59002803)

By the way, there are a number of non-DIY effort underway as well, with significant interest, support and funding, e.g.,
the [long-term energy storage challenge](https://www.sprind.org/en/impulses/challenges/energystorage)
by [SPRIN-D](https://www.sprind.org/en/we).
One example is the Swiss startup [Unbound Potential](https://youtu.be/e_3Yd8mKvmw?feature=shared).
#### Flexbase Plans Swiss 500MW Redox Flow
[Flexbase plant 500 Megawatt Redox-Flow-Speicher in der Schweiz](https://www.pv-magazine.de/2024/09/20/flexbase-plant-500-megawatt-redox-flow-speicher-in-der-schweiz/)