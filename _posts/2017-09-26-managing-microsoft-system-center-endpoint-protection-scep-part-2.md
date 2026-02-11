---
layout: post
title: "Managing Microsoft System Center Endpoint Protection (SCEP) – Part 2"
date: 2017-09-26
---

In [Part 1](/2017/09/18/managing-microsoft-system-center-endpoint-protection-scep-part-1.html), we looked at how it was possible to configure pretty much anything in SCEP with the venerable **scep\_set** command. Here, we're going to focus on something else. It's often in an organisation's information security policy to ascertain whether the devices you manage are "compliant" with a set benchmark, whatever it may be.

For many, that benchmark may include the need for an antivirus/malware solution that has up-to-date definitions. We might also want to know how many "infections" each client has encountered. As systems administrators, the expectation is that we should be able to know this stuff and report on it. Turning to SCEP, if we look in the user interface of the application itself, it does indeed give us a window into its activity logs for each Mac, _individually,_ but again, when it comes to integration with management tools that could report on the entire fleet, it appears we're stuck.

Or are we?

![scep_logs.png](/assets/2017/09/26/scep_logs.png)

<!--more-->

# Scraping logs

What if we could dig into SCEP's actual log files and pull out the information we need? As it turns out, SCEP's log files are just plain text, so let's have some fun with them!

**Note that for readability I have removed the paths to the log files in the example code snippets below, so be sure to add them back or else you'll be scraping files that don't exist!**

## What version of definitions do I have now?

Check out this bad boy:

```
"/Library/Application Support/Microsoft/scep/modules/data/updfiles/lastupd.ver"
```

This file contains a wealth of information but if you look in the **\[CONTINUOUS\_ENGINE1\]** section, there's an entry, **versionid**. With a little creative use of **grep** and **cut** we can isolate this:

```bash
grep -A 2 CONTINUOUS_ENGINE1\] "lastupd.ver" | grep versionid | cut -d "=" -f 2
```

In my case, at the time of writing, this returns the number **16135**. Check out this KB on ESET's website (remember, SCEP for Mac is rebadged ESET!). They update it regularly with the latest definitions version number: [https://support.eset.com/kb3264/](https://support.eset.com/kb3264/)

![eset_version.png](/assets/2017/09/26/eset_version.png)

What do you know, it matches!

[Here is a Jamf Pro Extension Attribute that returns this information](https://gist.github.com/neilmartin83/46a02662f0245d36f61bd7ae4ec2a9bf).

## When was the last time SCEP updated its definitions?

Look at this little gem:

```
"/Library/Application Support/Microsoft/scep/modules/data/data.txt"
```

If you look inside this file, some entries stick out, particularly **LastUpdate**. Let's scrape it out with some more **grep** and **cut** goodness:

```bash
grep -A 1 "LastUpdate=" "data.txt" | grep "LastUpdate=" | cut -d "=" -f 2
```

Which gives us (at the time of writing) **1506316156**. That's a date but not as we know it - the format is UNIX time. We can convert this to something human (and Jamf Pro in my case) readable with the **date** command:

```bash
date -r 1506316156 "+%Y-%m-%d %H:%M:%S"
```

Then we get **2017-09-25 06:09:16** - sweet!

Put it together and here's what you'd use to return the date at which SCEP last updated its definitions, in the format we want:

```bash
date -r `grep -A 1 "LastUpdate=" "data.txt" | grep "LastUpdate=" | cut -d "=" -f 2` "+%Y-%m-%d %H:%M:%S"
```

The above date's format is perfect for a Jamf Pro Extension Attribute which returns the "Date" Data Type. [Click here to download such an Extension Attribute](https://gist.github.com/neilmartin83/f564f5bf45bc19b3f244825294a0d586)! Because it outputs a date, you can use Smart Group criteria such as "less than x days ago" with it. Here's an example Smart Group you might use to report on whether Macs have the current version of SCEP with a recently updated set of definitions (Application Title is System Center Endpoint Protection.app). This might be used as part of a wider compliance reporting mechanism.

![scep_jss_compliane.png](/assets/2017/09/26/scep_jss_compliane.png)

## What have the on access and on demand scanners been up to?

SCEP logs statistics for its on demand and on access scanning in the following files, which again can be leveraged to provide specific information:

### **On demand log:**

```
"/Library/Application Support/Microsoft/scep/logs/stats.ondemand"
```

### **On access (realtime scanner) log:**

```
"/Library/Application Support/Microsoft/scep/logs/stats.onaccess"
```

These log files have an identical structure, so any methods you use to scrape one will work on the other. Here's the contents:

```bash
scanned: 655529
error: 40046
infected: 0
cleaned: 3
spam: 0
accepted: 1571350
deferred: 0
discarded: 3
rejected: 0
as_ham: 0
as_likely_ham: 0
as_likely_spam: 0
as_spam: 0
```

If you wanted to report on the number files cleaned by the on access scanner, you could use the following in a bash script:

```bash
cat "stats.onaccess" | grep cleaned | cut -d ' ' -f2
```

To report a different statistic, in the **grep** part above, you'd just replace **cleaned** with another item in the log, e.g. **infected**. To report on the on demand scanner, just change **stats.onaccess** to **stats.ondemand** above so you're **cat-**ing the correct log file (and don't forget to include the full path which I have removed here to make the code more readable!). Cleaned files are those which SCEP found to be infected and subsequently cleaned. Files are reported as infected if SCEP could not clean them.

### Some examples of Jamf Pro Extension Attributes:

[Number of realtime objects cleaned (on access)](https://gist.github.com/neilmartin83/fd154951ab657f6cd10f687c17a203e5)

[Number of realtime objects infected (on access)](https://gist.github.com/neilmartin83/ff99c62e1a150c9b2afb8d6159c4cfb4)

As above, those EA's could easily be modified to report on any statistic in either of SCEP's on access or on demand logs.

# Thank you!

My inspiration for writing this stuff has to come from somewhere. I'd like to point to [this thread on Jamf Nation](https://www.jamf.com/jamf-nation/discussions/15170/managing-microsoft-system-center-2012-endpoint-protection) as it got me going in the right direction, with specific thanks to Nicole Deal (@ndeal), Kenneth Edgar (@kedgar) and Alan Trewartha (@alan.trewartha).
