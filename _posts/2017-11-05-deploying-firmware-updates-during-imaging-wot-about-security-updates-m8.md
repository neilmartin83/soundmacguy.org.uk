---
layout: post
title: "Deploying firmware updates during imaging - wot about security updates, m8?"
date: 2017-11-05
coverImage: "firmware.png"
---

**Edit: There be dragons! This workflow is completely unsupported by Apple and [they don't want us to image anymore](https://support.apple.com/en-gb/HT208020). It's a naughty stop-gap, but in my case, right now, needs must. As for myself, I'll only be doing this going forward for the remaining Security Updates released for macOS 10.12. I won't be doing this for 10.13 (and it probably won't work anyway!). We use Jamf Pro and I really hope that Jamf add support to their tools for automated re-provisioning leveraging startosinstall. [I've even asked them to - please upvote my feature request if it's important to you too](https://www.jamf.com/jamf-nation/feature-requests/6618/jamf-imaging-support-installer-based-workflows).**

So Apple released macOS 10.13.1 to the world and we've just had the obligatory Security Updates for 10.12.6 and 10.11.6. If, like me, you're still deploying a previous version of macOS starting with a base image, with the Security Updates baked in, a-la [AutoDMG](https://github.com/MagerValp/AutoDMG), you might be thinking about firmware. Because, yep, those Security Updates include new firmware which you won't get if you just restore that pre-baked image. [And that's bad, mmmkay?](https://duo.com/blog/the-apple-of-your-efi-mac-firmware-security-research)

Luckily, due to our awesome community, Allister Banks and Darren Wallace have done great work writing up workflows to [extract the firmware from the App Store installer](https://www.afp548.com/2017/08/31/uefi-10-13apfs-and-your-imaging/) and [get it into a traditional imaging workflow](http://www.amsys.co.uk/2017/09/deploying-firmware-updates-imaging/). But can we get the newer firmware out of those newly released Security Updates and use it in the same way? Why, yes, we can.<!--more-->

I'll keep it short and sweet. If you build a macOS 10.12.6 or 10.11.6 image with AutoDMG and include the Security Updates, just grab the newer **FirmwareUpdate.pkg** that's bundled with those Security Updates. Here are the links, straight from Apple's own Software Update Servers:

[Security Update 2017-001 for macOS 10.12.6](https://swscan.apple.com/content/downloads/18/44/091-41055/m0r3838sb33d0pjl7d4359vdyxcuy6xhda/FirmwareUpdate.pkg) - shasum:

d3f592a7b29dcf7c5973d97dfa1fc276c5bbdaa8

[Security Update 2017-004 for macOS 10.11.6](https://swscan.apple.com/content/downloads/20/57/091-41764/g7nmv5zqkuubye96a3gyt84xyepm5c2ybz/FirmwareUpdate.pkg) - shasum:

df1d942cf0597dda10b623c360ee0cd9994cfc1a

Once you have the package that matches your image's version of macOS, follow the same steps to expand and rebuild it, covered in Allister and Darren's posts above.

In case you're wondering, I sniffed these packages out using [Margarita](https://github.com/jessepeterson/margarita) on my own [Reposado](https://github.com/wdas/reposado) based Software Update Server, but you could also use [SUS Inspector](https://github.com/hjuutilainen/sus-inspector), as shown here:

![SUSInspect](/images/susinspect.png)

Bear in mind that Apple doesn't seem to include the same firmware updates across both Security Updates - namely, macOS 10.11.6 doesn't get as many. [Here's a nice spreadsheet that shows the details](https://docs.google.com/spreadsheets/d/1qGRVF1aRokQgm_LuTsFUN2Knrh0Sd3Gp0ziC_VIWqoM/edit), from Pepijn Bruienne.
