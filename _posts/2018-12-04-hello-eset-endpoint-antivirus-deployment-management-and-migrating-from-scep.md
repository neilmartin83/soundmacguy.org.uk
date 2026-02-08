---
layout: post
title: "Hello ESET Endpoint Antivirus! Deployment, management and migrating from SCEP."
date: 2018-12-04
coverImage: "eset.png"
---

Following Microsoft's [announcement](https://soundmacguy.wordpress.com/2018/11/15/farewell-scep/) that System Centre Endpoint Protection (SCEP) for macOS and Linux is to be discontinued by the end of this year, their recommended migration path is to switch to ESET Cyber Security. In fact, ESET are granting anyone wishing to switch a year's subscription for free, which is nice.

It's been difficult to find a central repository of information around how to migrate, what the options are and how to deploy and configure ESET. Things seem to be scattered around forum posts, knowledge base articles, or behind a wall of reverse engineering and poking. So let's have a look (with a Jamf-twist although these things should be possible with other popular deployment tools)...<!--more-->

## Getting started

Firstly, visit https://www.eset.com/int/scep/ and fill out the form to get your year's worth of licenses. It's worth creating an account at http://eba.eset.com as well to keep your license details centrally stored and managed.

The next place you'll get pushed to is here, which has instructions on how to apply your license to the installer package using ESET's **add\_token** tool: https://support.eset.com/kb7026/ - and it tries to steer you to download the full-fat Endpoint Security product that includes a firewall amongst other things.

Interestingly, inside the application bundle, in /Contents/MacOS, we find a similar set of binaries that are in the same place with SCEP - our old friends **scep\_set**, **scep\_daemon** and so on are replaced with **esets\_set** and **esets\_daemon** etc. These may be useful...

### What if I just want the Antivirus product, like SCEP?

Fear not, just grab ESET Endpoint Antivirus for Mac here: https://support.eset.com/kb3614/

It's much closer to SCEP's level of functionality.  The **add\_token** tool works on this package too, so you can apply your license to it in the same way. The rest of this blog post will talk specifically about experiences with ESET Endpoint Antivirus for Mac.

Note: the article above doesn't mention that when you download **add\_token**, you need to set it to be executable in order for it to actually run - so do this once you've unzipped it:

```
chmod +x /path/to/add_token
```

### Any installation considerations?

- When you mount the downloaded DMG, it contains an alias for the installer package - you have to ctrl+click it and choose **Show Original** to get the actual package itself.
- ESET Endpoint Antivirus installs a Launch Daemon and Launch Agent:
    - **com.eset.esets\_daemon.plist** - Launch Daemon running at system level all the time. This loads **esets\_daemon** which provides real-time scanning, scheduling and all the background-level stuff ESET needs.
        
    - **com.eset.esets\_gui.plist** - Launch Agent loaded in the user context when they log in. This launches the GUI parts of ESET; **esets\_gui** (this is also what loads when you double-click the application). It includes a menu bar applet and the application user interface.
        
- If SCEP is present, ESET's installer will cleanly remove it, export and import its settings.
    - User level (GUI) settings are copied to all user accounts on the Mac in ~/.esets/gui.cfg - watch out, that folder is owned by root... New users get the default gui.cfg which you may not like - more on this later.
    - I didn't test thoroughly which settings were exported/imported as I'd be deploying ESET to Macs without SCEP as well and be looking to provide a complete configuration with the install, so this wasn't that useful for me.
- The package scripts do make assumptions that user homes are in /Users - so watch out if they're not in your environment.
- If you run the GUI installer, you get to choose which parts are installed/enabled. Unfortunately I looked at the InstallerChoices XML and they don't appear there so you can't control them that way with a command line install/deployment with your management tools. They are:
    - **Device Control** - lets you control access to externally connected devices (block for certain users etc) - disabled by default
    - **Web Access Protection** (inspects HTTP/S traffic on specified ports) - enabled by default
    - **Email Protection** (inspects POP/IMAP traffic) - enabled by default
    - **LiveGrid** (Uploads information/files that are found to be infected or suspicious to ESET) - enabled by default
- The installer package appears to work fine when invoked via the command line (i.e. installer command) or deployed through management tools like Jamf. It's very similar to SCEP's one.
- If you don't want the GUI to start straight after installation (e.g if you want to configure it first - see below), the following file must be present before installation begins:
    - ```
        /Library/Application Support/ESET/esets/cache/do_not_launch_esets_gui_after_installation
        ```
        
- If you're deploying to macOS 10.13 or 10.14 (High Sierra/Mojave), bear in mind this comes with a kernel extension you'll need to whitelist or your users will have to allow it manually during installation. The Team ID to whitelist is **P8DQRXPVLP.**
- If you're deploying to macOS 10.14, you need to allow ESET full disk access. See [this post](https://soundmacguy.wordpress.com/2019/01/23/eset-endpoint-antivirus-privacy-preferences-policy-control/) for more details.

### Remote management?

ESET provide **ESET Remote Administrator** which can remotely configure and manage ESET Endpoint Antivirus and Cyber Security. You do have to download and provision it on your own server (and they also provide a ready-made Azure VM).

If you're interested in setting ERA up and using it, check out https://support.eset.com/kb5982/

The rest of this post assumes you're not going to use ERA and that you wish to deploy and configure things with your own management tool of choice.

## Configuring it (the "system" part)

### License Activation

You have a couple of ways to activate your license; you could use the **add\_token** tool against the installer package, as suggested by ESET, which you'd have done before installation.

Alternatively, you can use **esets\_daemon** afterwards. This is particularly useful if you need to re-activate when your license has expired:

```
esets_daemon --wait-respond --activate key=XXXX-XXXX-XXXX-XXXX-XXXX
```

Where XXX-XXXX-XXXX-XXXX-XXXX is substituted with your activation key.

System/computer level settings are set in the following areas (from ESET's preferences dialog):

![ESET_SYSTEM](/images/eset_system.png)

This is where things are quite different than with SCEP. The file that's managed by the old **scep\_set** is gone and replaced by a single json file here:

```
/Library/Application Support/ESET/esets/cache/data/settings.json
```

So, **esets\_set** is useless here. The ability to granularly configure settings is no more.

It's now a case of:

1. Install ESET Endpoint Antivirus on a test Mac or VM
2. Open up its Preferences GUI
3. Configure the settings the way you want
4. Export the configuration to a file
5. Package the exported configuration
6. Deploy and import the exported configuration

For step 4, you can export your settings from the Setup pane in the GUI:

![ESET_IMPORTEXPORT](/images/eset_importexport.png)

Alternatively, you can use **esets\_daemon**:

```
esets_daemon --export-settings /path/to/settings/file
```

To import the settings using **esets\_daemon**, do this:

```
esets_daemon --import-settings /path/to/settings/file
```

It is important to note that you should unload ESET's Launch Daemon and ensure the GUI is not running when you import (if the GUI is running and loses contact with the Launch Daemon, it will interrupt the user with a very scary dialog):

```
killall esets_gui
launchctl unload "/Library/LaunchDaemons/com.eset.esets_daemon.plist"
esets_daemon --import-settings /path/to/settings/file
launchctl load "/Library/LaunchDaemons/com.eset.esets_daemon.plist"
```

Consider that if you are doing this whilst the user is logged in, they will see the menu bar icon vanish. Also consider whether you wish to re-load the GUI for them, or wait for it to load via the provided Launch Agent when they next restart or log in.

It is really important to note that if you disable certain functionality such as Email and/or Web Access Protection, the GUI, with its default settings, will make this clear to the user with lots of warnings and melodrama.

## Configuring it (the "user/GUI" part)

User-level settings are set in the following areas (from ESET's preferences dialog):

![ESET_USER](/images/eset_user.png)

Those settings are stored in the following file:

```
~/.esets/gui.cfg
```

The good news is that this file follows the same format as the one for SCEP, so [my old post here](https://soundmacguy.wordpress.com/2017/11/19/managing-microsoft-system-center-endpoint-protection-scep-part-3/) around using **scep\_set** to manipulate it is still relevant, as well as the advice regarding inspecting the file when you change preferences to work out how they're stored.

Alternatively, you can set things up in the preferences GUI, capture this file, then on your target Macs kill the **esets\_gui** process and copy this file over. When the GUI next launches, it'll pick up those settings. This should be done in the logged in user's context for every user.

## What does Neil do? (installation/configuration scripts and a custom LaunchAgent)

As I use Jamf, I have a policy which does the following:

1. Cache the ESET Endpoint Antivirus installer package
2. Install a package containing my exported settings
3. Run the following script:

https://gist.github.com/neilmartin83/5d0a3845dd4abbfdff244ac21dae6900

Let's break that down:

Lines 3-11: Variables you should change to suit your environment (descriptions are provided in the script).

Lines 17-23: If we're using our own GUI settings, this prevents the installer launching the GUI after installation.

Lines 25-26: Install it.

Lines 28-31: Import your exported configuration file.

Lines 33-92: Generate our "master" per-user GUI settings file. You could copy and paste the contents of your own gui.cfg file here (lines 40-91).

Lines 93-109: Generate a script and make it executable - this is intended to run as the logged in user and will copy over the master gui.cfg file (unless previously copied) and open the ESET Endpoint Antivirus application itself.

Lines 110-129: Overwrite the installed Launch Agent with our own that runs the script generated above. This ensures that every new user that logs in gets our GUI settings and not the defaults.

Lines 131-139: If there is a user logged in, configure the GUI application and launch it, in their context (i.e. as them). This portion of the script will not run if no user is logged in.

## Reporting (here, have some Extension Attributes)

It is possible to obtain a lot of useful information from **esets\_daemon** with:

```
esets_daemon --status
```

This is functionally identical to SCEP's **scep\_daemon**, which [I cover in lots of detail in this post](https://soundmacguy.wordpress.com/2018/04/07/microsoft-system-center-endpoint-protection-scep-more-hidden-reporting-goodness/).

I've written some Jamf Pro Extension Attributes that leverage this and they're available in my GitHub repo, here:

https://github.com/neilmartin83/Jamf-Pro-Extension-Attributes

So far, I've made EA's for the following:

- Activation Status
- Definitions Date
- Definitions Version
- Last In-depth Scan Date
- Last Smart Scan Date
- Number of Realtime Cleaned Objects
- Number of Realtime Infected Objects
- Realtime Protection Status

They can be uploaded directly to Jamf Pro as-is, or you could extract the script portions of them for use in other management tools.

## Other observations

A couple of tidbits I observed that don't really fit anywhere above:

- Non-privileged users can enable Web, Phishing and Email protection via the GUI if it's disabled, but they can't disable it after the fact. This may be a bug as these settings present as being greyed out in the preferences dialog.
- If the Mac is joined to a directory service (e.g. Active Directory), opening the Privileged Users preference pane will appear to lock-up **eset\_gui** as it populates its list of users from your directory (it doesn't get them all, but it gets _a lot_).

## Special thanks

Thank you to the following people who provided nuggets of information, encouragement and otherwise accompanied me on this journey into the depths of ESET!

- Steve Maser (@stevemaser on the Slack)
- Jon Crain (@joncrain on the Slack)
- David Whiteley (@deej on the Slack)
- The folks in this thread on the ESET forums: https://forum.eset.com/topic/17688-eset-endpoint-antivirus-is-not-configurable/
