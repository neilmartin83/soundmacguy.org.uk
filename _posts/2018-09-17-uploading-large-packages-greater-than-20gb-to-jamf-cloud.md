---
layout: post
title: "Uploading large packages (greater than 20GB) to Jamf Cloud"
date: 2018-09-17
coverImage: "largedmg.png"
---

2022-05-03 Edit: Thanks again to Scott Blake (@mscottblake on the Slack), for informing me that [Amazon have changed the file size limit to 30GB now](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-limits.html#:~:text=Maximum%20cacheable%20file%20size%20for%20HTTP%20GET%2C%20POST%2C%20and%20PUT%20requests).

2019-05-20 Edit: Thanks to Scott Blake (@mscottblake on the Slack), we now know the root cause of this limit is due to Amazon's CloudFront service imposing it. [Click here to read the specifics](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-limits.html).

![CFLimit](/images/cflimit.png)

If you're rocking Jamf Cloud, you might be making use of the provided Jamf Cloud Distribution Service (JCDS), or "Cloud Distribution Point" as it's otherwise known. My place recently migrated from an on-premise service, and I've been eager to take advantage of the JCDS. Internet bandwidth is not really an issue and getting rid of our on-premise File Share Distribution Points means we'd claw back 3 servers and 1.5TB of space in total, which is nice.

The Jamf Pro 10.6 update improved the reliability of uploading large packages using the Jamf Admin application ([PI-004504](http://docs.jamf.com/10.6.0/jamf-pro/release-notes/Bug_Fixes_and_Enhancements.html)) so this meant I could seriously look at syncing our on-premise DPs up to it (this was previously a blocker for us). Whilst I'm pleased to say it worked for most packages, I found those over 20GB (some music production software and content in particular!), did not upload, no matter what. Trying them over the web interface made no difference. For those playing along at home, Jamf confirmed this is the result of PI-005548. The workaround I came up with is a bit "Heath Robinson" and pertains mainly to delivering large disk images (DMGs) to clients. It won't work _directly_ for PKGs but you could be creative using my solution as a base.<!--more-->

Why DMGs? In my case I have a large vendor-supplied disk image that contains their installer. In your case, you might have a large PKG; you'll need to wrap that up in a DMG for this to work. My objective is to get that disk image onto the clients, mount it and execute a script to run said vendor's installer contained within for a proper vendor-approved installation and to avoid re-packaging. A [turducken](https://en.wikipedia.org/wiki/Turducken), if you will, in much the same way [Rich Trouton does here](https://derflounder.wordpress.com/2013/12/06/repackaging-the-labview-2013-pro-installer/). So here's what I did:

1. Split the large >20GB DMG into 5GB parts.
2. Package each part to get it installed in a temporary directory (/var/tmp is good).
3. Create a policy to install these DMG-part-containing packages.
4. Once all the parts are in place on the client, mount the segmented DMG and run the vendor's installer.

## Splitting the DMG (carving the turducken).

The DMG spec allows for segmenting a disk image into smaller parts (or chunks) fairly easily. This harks back to a time when internet connections were slow and file systems had annoying maximum file size limitations.

Assume you have a big DMG, called **"25-gb-original.dmg"** sitting in your /var/tmp directory. Of course this is an example file name and path - yours could be anything. Run this command:

```
hdiutil segment -segmentSize 5G -o /var/tmp/segmented /var/tmp/25-gb-original.dmg
```

Let's break it down:

**hdiutil -** the command we're running itself - it does lots of stuff with disk images.

**segment** - this verb tells **hdiutil** to split up the file we point it to.

**\-segmentSize 5G** - specifies the size of each segment **hdiutil** creates. In this case, 5 gigabytes. Substitute the G for M or K for megabytes and kilobytes, accordingly.

**\-o /var/tmp/segmented** - the path to the outputted segmented DMG (**hdiutil** will append filenames appropriately).

**/var/tmp/25-gb-original.dmg** \- the path to our original disk image.

You'll end up with the following files in /var/tmp:

```
/var/tmp/segmented-dmg.dmg
/var/tmp/segmented-dmg.002.dmgpart
/var/tmp/segmented-dmg.003.dmgpart
/var/tmp/segmented-dmg.004.dmgpart
/var/tmp/segmented-dmg.005.dmgpart
```

Each one of those files is a 5GB part of the overall DMG. As long as they exist in the same folder, you can mount **segmented-dmg.dmg** by double-clicking, or using hdiutil etc as you normally would. macOS knows the DMG is segmented and handles the rest for you.

## Packaging the DMGs (baking the turducken).

Unfortunately, uploading those parts straight to your JCDS won't work. When I tried, I saw some strange results - some files were downloadable, others weren't and only the file with the .dmg extension was showing up in the Cloud Distribution Point settings list on the Jamf Pro Server. I can only put it down to Jamf having a process on the JCDS that inspects the files based on their type and that gets confused with segmented DMG parts.

With the files making up your segmented DMG in place, package them **individually** using your favourite packaging tool ([Whitebox Packages](http://s.sudre.free.fr/Software/Packages/about.html), Jamf Composer etc). Since packaging methodology is a bit out of scope here, I won't go into how to do it - there are resources such as [this wonderful book](https://scriptingosx.com/packaging-for-apple-administrators/) and numerous blog posts out there that cover the ins and outs. Ensure your packages each install **one part** of the segmented DMG to **the same** specified location (e.g /var/tmp).

## Final steps

Upload your resulting 5GB PKG packages to your JCDS with Jamf Admin or the web interface. It'll work!

1. Create your new policy and add a **Packages** payload.
2. Add each of your new DMG-part-containing-packages and make sure they're set to **Install.**
3. Profit!

The above policy will get your multi-part DMG onto your clients. What you do from there (or how you indeed profit) depends on what your DMG contains (running the vendor's installer, or using the **installer** command to install that large package inside etc), but it will probably involve a script and start by mounting the DMG, by doing something like this:

```
hdiutil attach /var/tmp/segmented-dmg.dmg
```

The rest is up to you!

## ProTip:

For disk images that contain _many_ installer packages (PKGs), the wonderful [InstallPKG](https://github.com/henri/installpkg) tool is a Godsend. I deploy it to all the Macs I manage. InstallPKG will mount the DMG and install every PKG inside it with _one_ command - so using **hdiutil** is not necessary:

```
installpkg -ih /var/tmp/segmented-dmg.dmg
```

I hope this helps folks who are trying to get large unwieldy installers into their Jamf Cloud Distribution Points! I also hope that Jamf fixes things so we can upload those occasionally large packages without having to do these kinds of workarounds, although I did enjoy diving into **hdiutil** quite a bit!
