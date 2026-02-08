---
layout: post
title: "Apple - when it rains, it pours..."
date: 2017-08-21
---

Today, Apple published a couple of articles on their support website that are of particular interest to the admin community. Especially as they talk about things coming up in macOS 10.13 High Sierra!

[Prepare for changes to kernel extensions in macOS High Sierra](https://support.apple.com/en-us/HT208019)

[Upgrade macOS on a Mac at your institution](https://support.apple.com/en-us/HT208020)

A couple of things these articles confirm for us:

NetBoot (or at least NetInstall) is still alive and kicking! But watch out if you image Macs, as Apple now officially confirm that firmware updates don't get applied during imaging, but only when you run their installer-proper or install an OS update later (e.g. 10.12.5 to 10.12.6 or one of the security updates for the older OS's - but then you may not get the latest firmware, depending on your Mac). [Click here for an excellent table showing which firmwares come with each OS update](https://docs.google.com/spreadsheets/d/1qGRVF1aRokQgm_LuTsFUN2Knrh0Sd3Gp0ziC_VIWqoM/edit?usp=sharing). Huge thanks to Pepijn Bruienne!

And then there's this quote, which might make you breathe a sigh of relief (if you manage your Macs with an MDM):

> In macOS High Sierra, enrolling in Mobile Device Management (MDM) automatically disables SKEL. The behavior for loading kernel extensions will be the same as macOS Sierra.
> 
> In a future update to macOS High Sierra, you will be able to use MDM to enable or disable SKEL and to manage the list of kernel extensions which are allowed to load without user consent.

Secure Kernel Extension Loading (SKEL) is a security feature that would mean kernel extensions need to be manually authorised (interactively by the user). It looked set to cause a great deal of inconvenience for us in particular, specifically for things like third party drivers and anti-virus/malware products that make use of them. Until now, to disable SKEL you would use the **spctl** command in a NetInstall or recovery environment, so this change is great news.

So not only will SKEL not be a problem if you use an MDM, but soon we'll be able to optionally enable it (which would be of great benefit in some security conscious environments - but then if you have a third party security product that uses kernel extensions, then users being able to stop them loading presents a bit of an irony...).
