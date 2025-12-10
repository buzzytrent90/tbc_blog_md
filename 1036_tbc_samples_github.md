---
post_number: "1036"
title: "The Building Coder Samples on GitHub"
slug: "tbc_samples_github"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'windows']
source_file: "1036_tbc_samples_github.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1036_tbc_samples_github.html"
---

### The Building Coder Samples on GitHub

After several prompts from various quarters, I finally got around to publishing
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) on GitHub.

To access them from there, you can either open the
[repository](https://github.com/jeremytammik/the_building_coder_samples) or
grab the code through the direct
[zip file download](https://github.com/jeremytammik/the_building_coder_samples/archive/master.zip).

As discussed below in the section on
[branching and tagging](#4),
these links refer to the master branch.

For a specific version of the samples, e.g. from today, please
[see below](#4).

Another important aspect that I discovered in the process and always had suspected in the past is the importance of adding a software license.

#### The Importance of a License, Any Old License

Luckily, I ran into this nice
[coding horror](http://www.codinghorror.com) admonition
to
[pick a license, any license](http://www.codinghorror.com/blog/2007/04/pick-a-license-any-license.html) before
publishing.

If no license is specified, an implicit copyright is asserted.
People can read the code, but they have no legal right to use it.
To use it, you must contact the author directly and ask permission.

No serious professional programmer is going to use any published code without explicit permission.

You even have to specify a license to put your code in the
[public domain](http://en.wikipedia.org/wiki/Public_domain).
Alternatively, you can die, and then anybody interested will still have to wait for a long time.

I considered using the
[WTFPL](http://www.wtfpl.net) advocated
in the coding horror article.

Being a bit leery of the potential reaction of my colleagues, however, who might object to the wording of its name, I desisted and settled for the minimalistic and permissive
[MIT License](http://en.wikipedia.org/wiki/MIT_License) instead.

#### Creating a GitHub Repository

I had initially planned to take notes of every step I made to set up the repository and link it with my local file system, but unfortunately didn't.

I am using the Mac version of git, anyway, so my steps would not be of great interest to the majority of my readers.

However, while prompting me to publish The Building Coder samples to GitHub, Victor Chekalin, or Виктор Чекалин, very kindly listed the detailed steps that he recommends on a Windows platform as follows:

1. Register on [github](https://github.com) if you are not yet registered.
2. Create new repository on github.
   In the new repository window, type the repository name and select Add .gitignore CSharp.
   .gitignore contains the rules for folders and files that should not be stored in the repository.
   Then click Create Repository.
3. The repository is created on github. Now is the time to install Git client.
4. At first install
   [mysgit](http://code.google.com/p/msysgit/downloads/list?q=full+installer+official+git).
   This is the official Git client for Windows.
   Download and install it with default settings.
5. You may already start using git, but mysgit is a console client.
   I use the more comfortable
   [TortoiseGit](http://code.google.com/p/tortoisegit/wiki/Download) Windows
   client instead.
   Download and install it with default settings as well.
6. After you installed TortoiseGit, open the TortoiseGit settings and go to the Git node.
   Type your name and email.
7. Now we are ready to create local repository in your project folder.
8. Find the folder where your project is located.
   Right click on this folder and select Git Create repository here...
   In the next window click OK.
9. Right click on this folder again and select TortoiseGit > Settings.
10. In the Git > Remote node copy and paste the URL of the github repository you created.
    Click Apply.
    Tortoise Git asks you if you want fetch branches from the remote repo; click No.
11. Right click on the folder, select the TortoiseGit > pull Command to pull .gitignore from the github and click OK.
12. Commit your first changes: Git Commit > master.
    Type the commit comment, select all your files to add them to the repository and click OK.
13. The commit initially only applies to your local repository.
    To commit those changes to the remote repository as well, i.e. on github itself, you have to push them.
    Click Push. Click OK. Type your name and password.
14. That's it. The code is on github.
15. When you have made any changes in the code, commit your changes.
    Do not forget add files in the commit window if you add new files.
    Push the changes after committing them locally.

This pretty much matches my Mac experiences as well.

Actually, once I had the github repository up and running and everything safely backed up, I removed my original copy and created a new one in the original location using the following command:

```
  git clone master_url local_dir
```

#### Branching and Tagging

Luckily, I talked again with Victor before going live with this post, and he pointed out yet another very important aspect:

One thing to remember is that when a link to The Building Coder samples GitHub page or to the archive is posted, it generally refers to the main, or master, code stream.

As a result, a post written today, publishing the github link, will refer to something completely different in a year's time.

A user wishing to access the particular version discussed in the post will get the latest version of the examples.
These may already include a lot of changes.
As a result, the user sees completely different code than what is described in the post.

One solution is to create a
[branch](http://git-scm.com/book/en/Git-Branching-Basic-Branching-and-Merging).

Whenever you publish a github link, create a new branch, e.g., bc\_yyyymmdd, and provide a link to this branch.

Perhaps, instead of creating a new branch each time, you may create a tag and provide a link to the specific tag.

This is easier than branching.
For this case, it is exactly that you need.

Actually, while exploring the tagging mechanism, I discovered the easy method of creating an official
[release](https://github.com/jeremytammik/the_building_coder_samples/releases) directly
in the github web site, and used one single click to generate
[version 2014.0.104.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.104.2) as
a separate release, the initial github release.

Actually, a release is also simply a special kind of tag.

Many thanks to Victor for all his support and detailed instructions!