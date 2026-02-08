---
layout: post
title: "Uninstalling Adobe Flash Player - in a Flash!"
date: 2020-01-24
coverImage: "byeflash-1.png"
---

It's been a while since my last post. That's because I joined [dataJAR](https://datajar.co.uk/) as a Systems Engineer in October after 15 years of working at the [University of East London](https://www.uel.ac.uk/). It's been a huge change and I'm loving every minute. I'm learning a huge amount from a fantastic team of wonderful people as I settle into the new role.

So, we're approaching a milestone in computing history. All good things must come to an end and love it or hate it, [Adobe's Flash Player is no exception](https://community.adobe.com/t5/flash-player/flash-player-end-of-life/td-p/10206952). As we enter a new decade, we say goodbye to this venerable piece of the interwebs, as it goes End of Life later this year.

As an admin, the announcement of Flash Player's demise a mixed bag. Pleasure/pain, perhaps. Pleasure, in that you won't have to manage the deployment of updates for [one of the most frequently updated software titles ever](https://helpx.adobe.com/uk/flash-player/flash-player-releasenotes.html) (weren't you using [Autopkg](https://autopkg.github.io/autopkg/) so you didn't have to anyway?). Then, as the serotonin wears off, the pain sets in. Because the slow realisation dawns on you; how are you going to automate the removal of this on all the computers you manage? Since it won't get any more updates, that means any as-yet-undiscovered security vulnerabilities will remain unpatched. So it has to go. And there are a few ways to do the deed...

<!--more-->

[Adobe provide a GUI uninstaller for Flash](https://helpx.adobe.com/uk/flash-player/kb/uninstall-flash-player-mac-os.html). If your users are admins, and you can rely on them to do the needful, then that's fine and there's nothing else for you to do. As well as it being available as a standalone download you can also find it in **/Applications/Utilities/Adobe Flash Player Install Manager.app**

[Adobe also provide the Creative Cloud Cleaner tool](https://helpx.adobe.com/creative-cloud/kb/cc-cleaner-tool-installation-problems.html#RemoveallorselectedproductsEnterpriseusersonly). With this, you could generate an XML file for it to use, then comment out everything except the entry for Flash Player, then package it all up with a script to run it silently. Thanks to Dom Adobe in the MacAdmins Slack for this tip! Unfortunately, in my brief testing, I couldn't get it to detect Flash Player, so it wouldn't generate the necessary entries in its XML file.

Alternatively, you could just run this command (as root):

```
"/Applications/Utilities/Adobe Flash Player Install Manager.app/Contents/MacOS/Adobe Flash Player Install Manager" -uninstall
```

Yep, that's it. Really. Put it in a script and push it out with your management tool. Then you're done.

I stumbled on this with **strings**. It's a utility that comes with the [Xcode Command Line Tools](https://macpaw.com/how-to/install-command-line-tools) and it finds ASCII strings in binary files. It's really useful for running against application binaries to sniff out hidden command line flags and such. So I tried it on the Adobe Flash Player Install Manager, and here's part of the output (there was a LOT more!):

https://gist.github.com/neilmartin83/f6bef5964c41d24b1cf3f5c923573ea0

My spidey senses led me to think that **\-uninstall** might be a flag that would, erm, uninstall. I tried it out and got lucky.

Have fun!
