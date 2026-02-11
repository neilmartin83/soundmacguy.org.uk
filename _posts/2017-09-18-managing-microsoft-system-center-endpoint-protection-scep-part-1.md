---
layout: post
title: "Managing Microsoft System Center Endpoint Protection (SCEP) - Part 1"
date: 2017-09-18
---

If you're using Microsoft System Center Configuration Manager (SCCM) to deal with Windows machines in your environment, you may notice that it comes licensed with an antivirus/malware product; Endpoint Protection (SCEP), with versions for Windows, Linux and [macOS](https://support.microsoft.com/en-gb/help/2683548/frequently-asked-questions-about-system-center-2012-endpoint-protectio). This fits the bill nicely for organisations where their IT security policies dictate that such software is required on all company devices - just deploy this everywhere without having to deal with the expense and complexity of different products.

One thing I've noticed is that there seems to be a misconception that SCEP for Mac can be managed centrally with SCCM. This might be because SCCM admins who deploy SCEP for Windows can indeed enjoy this luxury, so why would the non-Windows versions be any different? After all, SCEP is SCEP is SCEP, right? Well, no. The macOS version is essentially a Microsoft-rebranded version of [ESET Cyber Security for Mac](https://www.eset.com/us/home/cyber-security/). And at first glance there seems to be no way to manage its settings/configuration centrally. As far as I know, there's no documentation on this so here's an account of what I've discovered and hopefully it'll help you manage your SCEP deployment.<!--more-->

## Some stuff you might like to configure:

- **On access scanner settings** - by default SCEP is configured to be pretty paranoid, scanning every file on creation, read and execution. This causes a huge performance hit. One example being (on a 2013 iMac with spinning disk), using Logic Pro for the first time after installing its entire content collection gives you an approx. 10 minute delay when you open the Loops library and it needs to scan all the loops to build a database. This takes less than 2 minutes when on-access scanning is disabled.
- **Exclusions** - sometimes you want to avoid having the on-access scanner looking at certain locations for performance or stability reasons. For example, we noticed that Adobe InDesign would give file errors when working live to an SMB network share. This went away when an exclusion to that share's location was added.
- **Scheduling** - By default, SCEP creates a few of scheduled tasks; a weekly system scan on Monday at 2am, checks for definitions updates (at user login but never more than once per 60 minutes as well as every 60 minutes whether a user is logged in or not), a startup file check at system startup and user login and a log file check every day at 3am (or ASAP if missed). Maybe you'd like to add your own tasks or modify/remove these.

# scep\_set is your friend

Buried within the SCEP application bundle is a binary, **scep\_set** that can be used to write out any of SCEP's settings to its preference file. The binary is here:

```
/Applications/System\ Center\ Endpoint\ Protection.app/Contents/MacOS/scep_set
```

The preference file is here:

```
/Library/Application\ Support/Microsoft/scep/etc/scep.cfg
```

If you run **scep\_set --help** you'll get this:

```bash
Usage: scep_set [OPTIONS..] [COMMAND]
System Center Endpoint Protection Configuration modifier

Commands:
      --set='NAME=VALUE'  set NAME=VALUE for given SECTION (or USERSPEC), or 
                            unset it if only NAME is given
      --create            create given SECTION (or USERSPEC)
      --delete            delete given SECTION (or USERSPEC)
      --last              make USERSPEC the last one
      --backup=FILE       back up configuration to FILE
      --apply=FILE        apply configuration from FILE
  If no command is given, --set is assumed.
Options:
      --cfg=FILE          configuration file
      --section=SECTION   operate on SECTION instead of "global"
      --user=USERSPEC     operate on USERSPEC in SECTION
Common options:
  -h, --help              show help and quit
  -v, --version           show version information and quit
```

Helpful, no?

For managing individual settings, all we're really interested in is using **scep\_set** with the **\--set** command and if you just use **scep\_set** by itself, this is assumed. So that's easy!

## Configuring Real-Time scanning settings

Unfortunately, with exception of the scheduler, SCEP's default settings aren't stored in **scep.cfg** so how are we supposed to know _what_ to set? The best method is to change something in SCEP's preferences dialog and compare the contents of **scep.cfg** before and after. For example, only allowing scanning files when they're created alleviates a considerable performance bottleneck:

![scep_realtime.png](/assets/2017/09/18/scep_realtime.png)

If you venture into the **Preferences, Real-Time Protection** tab and leave **Scan on:** **File creation** ticked (not the other options), you'll see this appear in **scep.cfg**:

```ini
[fac]
event_mask = "create"

```

**\[fac\]** is the section that concerns all the real-time scanner settings (I have no idea what it stands for).

**event\_mask = "create"** is the setting itself.

So that's the setting if we only want to enable real-time (on access) scanning when files are created. Similarly, if we wished to enable all three options of **File creation**, **File execution**, and **File open**, here's what the setting would look like:

```ini
event_mask = "open:create:exec"
```

So we can use **scep\_set** to modify this!

To set real-time scanning to **File creation** only, using **scep\_set**, you'd do this (remembering to begin with the full path to the **scep\_set** binary - I've omitted it here for readability):

```bash
scep_set --section=fac 'event_mask="create"'
```

Then reload SCEP's Launch Daemon which will restart SCEP with the setting applied:

```bash
launchctl unload -wF /Library/LaunchDaemons/com.microsoft.scep_daemon.plist
launchctl load -wF /Library/LaunchDaemons/com.microsoft.scep_daemon.plist
```

Here are some further examples that will disable scanning "Potentially unwanted" and "Potentially unsafe" applications (what this terminology actually means is documented in SCEP's online help, if you're interested):

```bash
scep_set --section=fac 'av_scan_app_unwanted = no'
scep_set --section=fac 'av_scan_app_unsafe = no'
```

Discovering a specific setting in **scep.cfg** is just a case of changing it in SCEP's GUI preferences dialog and re-reading that file. Have fun!

## Configuring Exclusions

![scep_exclusions.png](/assets/2017/09/18/scep_exclusions.png)

As above, it's easiest to configure an exclusion in SCEP's preferences dialog then extract it from **scep.cfg**. Exclusions are in the **\[global\]** section, for example, if we want to exclude everything in the **Applications** folder it would look like this:

```ini
[global]
av_exclude = "/Applications/*.*::"
```

With the **scep\_set** command looking like:

```bash
scep_set --section=global 'av_exclude = "/Applications/*.*::"'
```

Excluding multiple paths (e.g. the Applications and Users folders) would look like this in **scep.cfg**, with each path separated by a pair of colons:

```ini
av_exclude = "/Applications/*.*::/Users/*.*::"
```

You know what to do...

By the way, I'm not implying that you should (or shouldn't) exclude either of those paths!

## Configuring Scheduling

![scep_scheduler.png](/assets/2017/09/18/scep_scheduler.png)

Under the bonnet, SCEP has a scheduler that can do all sorts of things (I listed the stuff it does out of the box earlier). Again, the easiest way to figure out what to feed **scep\_set** is to create schedules in the GUI. In **scep.cfg**, scheduled tasks live in the **\[global\]** section and all tasks are consolidated in a single line. If we only had the **Log maintenance** task, it would look like this:

```ini
[global]
scheduler_tasks = "1;Log maintenance;;0;0 3 * * * *;@logs;"
```

Breaking it down we have **1** (task number - not visible in the GUI), **Log maintenance** (task title), **0** (run at earliest opportunity if missed, **0 3 \* \* \* \*** (minute, hour, day of month, month, year, day of week - in this case, 3am every day - this is very similar to [cron](https://en.wikipedia.org/wiki/Cron)), **@logs** - the actual task.

To configure the above task with **scep\_set**:

```bash
scep_set --section=global 'scheduler_tasks = "1;Log maintenance;;0;0 3 * * * *;"'
```

**Note that this would delete all other scheduled tasks.**

By default, SCEP includes some out-of-the-box tasks to scan files at system startup/user logon and perform a full scan every week. The (very long) setting in **scep.cfg** looks like this - I've colour-coded each task to make them easier to separate:

```ini
scheduler_tasks = "1;Log maintenance;;0;0 3 * * * *;@logs;3;Startup file check;disabled;0;login;@sscan lowest;4;Startup file check;disabled;0;engine;@sscan lowest;20;Weekly scan;;;0 2 * * * 1;@uscan scan_deep:/;64;Regular automatic update;;;repeat 60;@update;66;Automatic update after user logon;;;login 60;@update;"
```

To delete an unwanted task (e.g. the Weekly scan), you'd **delete** the following snippet from the above line:

```
20;Weekly scan;;;0 2 * * * 1;@uscan scan_deep:/;
```

Then, incorporate the new line into a **scep\_set** command:

```bash
scep_set --section=global 'scheduler_tasks = "1;Log maintenance;;0;0 3 * * * *;@logs;3;Startup file check;disabled;0;login;@sscan lowest;4;Startup file check;disabled;0;engine;@sscan lowest;64;Regular automatic update;;;repeat 60;@update;66;Automatic update after user logon;;;login 60;@update;"'
```

# In conclusion

Hopefully, this will give you the gist of getting to grips with discovering SCEP's settings as they're stored in **scep.cfg** and using **scep\_set** to programatically set them.

I recommend configuring SCEP with a script, run as root, following installation (e.g. a Munki postinstall or an _After_ script in a Jamf Pro policy would work). Here's an example of such a script that also includes creating the **scep.cfg** file (as it isn't present immediately after installation but needs to be) and unloading/reloading the Launch Daemon at the right times (I also allowed some delay time to let SCEP fully launch):

{% gist 17caf6583bd43213a115a7ffe46777ca %}

Note that enabling/disabling these things _could_ pose a risk to the security of your environment. Don't blindly copy my settings; _carefully_ _consider_ every setting you wish to change for yourself. I'm not responsible if you get infested with viruses/malware or get in trouble with your IT security/infosec people!

In part 2 I'll explore how you can glean valuable data from SCEP's various logs to report on recent infections, definition versions and so on. Stay tuned!
