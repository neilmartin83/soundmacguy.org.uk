---
layout: post
title: "Managing Microsoft System Center Endpoint Protection (SCEP) – Part 3"
date: 2017-11-19
coverImage: "scepuser1.png"
---

The Mac Admins community is interesting (amongst other things!). What's really interesting is when someone contributes something, others often come forward and build on their work, ever-advancing it towards a state of pure awesome.

@glaurung got in touch with me on the [Slack](https://macadmins.herokuapp.com/) after I published [part 1](https://soundmacguy.wordpress.com/2017/09/18/managing-microsoft-system-center-endpoint-protection-scep-part-1/) and [part 2](https://soundmacguy.wordpress.com/2017/09/26/managing-microsoft-system-center-endpoint-protection-scep-part-2/) of my thoughts on managing SCEP. Here's what he said:

![glaurung.png](/assets/2017/11/19/glaurung.png)

This definitely builds on what I've done so far, and after some tinkering, I think I've in turn built on that... Read on!<!--more-->

## Configuring User-specific GUI Preferences

I wasn't aware that there were user-specific GUI preferences you could also manage (because there isn't anything there I wanted to manage in my environment - I was lazy and only learned what I needed to get the job done). However, this looks interesting, and I started digging. SCEP stores these settings here (so totally ignoring Apple's standards then!):

```
~/.scep/gui.cfg
```

Looking at the contents of this file, it appears to be structured in the same way as SCEP's system wide preference file. We've got sections! Let's have a peek:

https://gist.github.com/neilmartin83/aea25a876fbf59022357acda36918bab

@glaurung used **[sed](https://www.gnu.org/software/sed/manual/sed.txt)** to manipulate the contents of this file directly but as the file is using SCEP's known structure, I thought I'd try **scep\_set**, remembering that it has a **\--cfg** option:

```
--cfg=FILE          configuration file
```

Note that you can't use **~** when you specify the path - it has to be the FULL path to the file, e.g. /Users/neil/.scep/gui.cfg

Lo and behold, **scep\_set** works!

In this case, following @glaurung's example, we're going to disable the notification SCEP pops up when it updates definitions. By default, SCEP runs a scheduled task to check for new definitions every hour (I covered this in [part 1](https://soundmacguy.wordpress.com/2017/09/18/managing-microsoft-system-center-endpoint-protection-scep-part-1/)). Definitions are often updated many times a day, so I can see why this might annoy people. On the other hand, folks might like to know SCEP is doing its thing and keeping up to date.

In the GUI, the preference can be found in the **Update** section, **Advanced Options**:

![scepgui2.png](/assets/2017/11/19/scepgui2.png)

Looking in **gui.cfg** it's easy to work out the actual preference:

```ini
update_success_tray_report_enabled=yes
```

ProTip - you can take a copy of this file, change a preference in the GUI, then compare the new original with the copy to see what's changed - decent text editors like [BBEdit](https://www.barebones.com/products/bbedit/) can compare the contents of two files, or you could use the **diff** command. Anyway...

We want to set this to **no**. The **scep\_set** command (for my username) is:

```bash
scep_set --cfg=/Users/neil/.scep/gui.cfg --section gui 'update_success_tray_report_enabled=no'
```

The SCEP GUI application has to be re-launched for this to take effect with the following 2 commands:

```bash
killall scep_gui
open "/Applications/System Center Endpoint Protection.app"
```

## Scripting it

If you want to set these GUI preferences, you have to do this for each user individually. This means any **scep\_set** commands you use must be run as the logged in user. If you use something that runs scripts as root (like [Jamf Pro](https://www.jamf.com/products/jamf-pro/)), then you need to run this at user login, and:

1. Work out who the logged in user is (thanks [Ben Toms](https://macmule.com/) - @macmule).
2. Make sure SCEP's GUI application is running (thanks [Richard Purves](http://www.richard-purves.com/) - @franton).
3. Run **scep\_set** commands as them to set whatever preferences you want.
4. Kill and relaunch SCEP's GUI application.

Here's what I'd do for a script run by a Jamf Pro Login Policy:

{% gist 26af72058b7a946755f2f8ba94e4f0b4 %}

If you use [Outset](https://github.com/chilcote/outset) to run scripts at login, you shouldn't use **sudo -u** **"$loggedInUser****"** as the script is run as that user anyway (and it won't work). You still need to get the logged in username to use in the path to the **gui.cfg** file - **scep\_set** doesn't recognise ~.

Thanks to @glaurung for highlighting these extra preferences!
