---
layout: post
title: Microsoft to Integrate Git into Windows
date: '2014-04-01 12:32:32'
tags:
- fun
- off-topic
---

It seems like every day I read about a new use for Git outside of source control. It's used by artists and authors to track changes to their work. It's used by researchers to branch experiments.

Well, today I came across the latest and most surprising Git integration. Just out of Redmond, Microsoft has announced that Git will be integrated into the next version of Windows via extensions to NTFS and the Windows kernel.

Microsoft's press release details the features Git will bring. The coolest one is the ability to create machine images as branches and easily switch between images by running `git checkout [branch]`. Imagine having two Git branches or images called *web* and *database*. Turning your machine into a fully configured web server is as simple as `git checkout web`. Need a database server? Then run `git checkout database`.

Even better you can push an image to a remote server with `git push [remote] [branch]` and checkout the branch on the remote server to deploy the image. Windows will perform either a partial reboot, full reboot, or restart selected services depending on what changed. In other words, you can tweak a server's settings on a local image, verify the changes, and then push the image to the server. All using Git.

Other features include:

* Tracking the history of every file to enable users to recover older versions.
* Enabling fine grained system restores down to single character changes to a file.
* A slick history view built into Windows File Explorer.
* Git support for Skydrive files.
* Simple registry restores.

Just when I think I've seen it all, something like this comes along and blows my mind. For more details, check out the [press release](http://media.joebuschmann.com/april_fools.jpg).