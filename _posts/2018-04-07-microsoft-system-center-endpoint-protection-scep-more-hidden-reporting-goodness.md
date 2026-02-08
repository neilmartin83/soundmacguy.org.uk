---
layout: post
title: "Microsoft System Center Endpoint Protection (SCEP) - More hidden reporting goodness"
date: 2018-04-07
coverImage: "scep-icon.png"
---

I thought I was done with SCEP (see parts [1](https://soundmacguy.wordpress.com/2017/09/18/managing-microsoft-system-center-endpoint-protection-scep-part-1/), [2](https://soundmacguy.wordpress.com/2017/09/26/managing-microsoft-system-center-endpoint-protection-scep-part-2/) and [3](https://soundmacguy.wordpress.com/2017/11/19/managing-microsoft-system-center-endpoint-protection-scep-part-3/)) but whilst undertaking an exercise looking into using SCEP on some Linux servers (and specifically looking at how it can provide reporting data to [SCOM](https://en.wikipedia.org/wiki/System_Center_Operations_Manager) via a Management Pack), I inadvertently came across a little-documented command line argument for one of its binaries, **scep\_daemon**.

The documentation for the Linux SCEP SCOM Management Pack (what a mouthful!) vaguely alluded to feeding data to SCOM via a **\--status** argument. This argument isn't mentioned anywhere else in SCEP's Linux documentation, nor listed when you invoke **scep\_daemon --help** on either platform.

The Linux version of SCEP is also a rebranded version of ESET, just like its macOS counterpart and the above **scep\_daemon** binary is also present in that version, so I thought I'd experiment in macOS...<!--more-->

There is a brief mention in the macOS documentation on the installer ISO, but the path to the binary is wrong (it says /Applications/.scep/scep\_daemon). The **scep\_daemon** binary is actually here:

```
/Applications/System\ Center\ Endpoint\ Protection.app/Contents/MacOS/scep_daemon
```

But we'll refer to it as **scep\_daemon** from now on (just to keep my examples shorter and sweeter).

Running the macOS **scep\_daemon** binary with the **\--status** argument surprisingly yields the following:

```
$ scep_daemon --status
RTPStatus=Enabled
ClientVer=4.5.32.0
AVSigsVer=17183 (20180407)
AVSigsDate=2018-04-07T11:23:00
AVSigsServer=http://um02.eset.com/scep/
AntivirusAntispywareModVer=1536.1 (20180328)
SystemState=Maximum protection
AutomaticUpdateSignature=Enabled
StartupScanAfterLogon=Disabled
StartupScanAfterUpdate=Disabled
RTPEventMask=create
RTPAdvHeuristic=Disabled
RTPAdvHeuristicExec=Disabled
RTPAdvHeuristicCreate=Disabled
RTPStatistics=Infected:0|Cleaned:0|Deleted:0
ScanStatistics=Infected:0|Cleaned:0|Deleted:0
```

The results pretty much speak for themselves in terms of what they mean and you can easily scrape them to get individual snippets.

For example, to get the status of the Real Time Protection (on access) scanning engine:

```
scep_daemon --status | grep RTPStatus | cut -d '=' -f 2
```

This will return "Enabled" or "Disabled". You could easily spin this into an Extension Attribute for Jamf Pro, for example:

https://gist.github.com/neilmartin83/5d82c4454d88df5c60bddd4f5cdb2ab2

You could report on it with an Advanced Search or even use it as the criteria for a Smart Group, creating a remediation policy that runs a script to re-enable protection if it's disabled. We just need a little help from our old friend, **scep\_set**, for example (see [part 1](https://soundmacguy.wordpress.com/2017/09/18/managing-microsoft-system-center-endpoint-protection-scep-part-1/) for a more thorough overview of using it):

https://gist.github.com/neilmartin83/76105fb6c413a4f36b086006138d0bcb

As a bonus, if you've ever ran scheduled or ad-hoc on demand scans, **scep\_daemon --status** will report extra results including the type of scans run (Quick Scan and Deep Scan), the directory path they were targeted to, when they were last run and if they were interrupted, for example:

```
LastQuickScanStatistics=Date:2018-04-07T15:42:48|Paths:/; |Total:2254|Infected:0|Cleaned:0|Status:Interrupted by user|Vdb: 17184 (20180407)
LastDeepScanStatistics=Date:2018-04-07T15:43:02|Paths:/etc; |Total:257|Infected:0|Cleaned:0|Status:Completed|Vdb: 17184 (20180407)
```
