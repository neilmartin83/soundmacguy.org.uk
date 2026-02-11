---
layout: post
title: "ESET Endpoint Antivirus - Privacy Preferences Policy Control"
date: 2019-01-23
coverImage: "eset.png"
---

A quick one - if you're deploying ESET Endpoint Antivirus or Security to Macs running macOS 10.14.x Mojave, you (or your users!) will have encountered this dialog:

![](/assets/2019/01/23/image.png)

To compound the problem, if you follow the instructions in that dialog and you're not logged in as a local administrator, you'll soon hit a roadblock.

This is all part of Apple's new Privacy Preferences Policy Control (PPPC), or Transparency Consent Control (TCC), depending on what you want to call it. If you'd like to know more about that, check out [Carl's blog post](https://carlashley.com/2018/09/28/tcc-round-up/) and join the #tcc channel on the MacAdmins Slack.

But what can we do about ESET?

<!--more-->

The answer is pretty simple, yet not:

1. Create a Configuration Profile that allows ESET Endpoint Antivirus full disk access.
2. Deploy it with your MDM service to clients that are either User Approved or enrolled via Apple School or Business Manager (you can't install this profile any other way!).

I used Jamf's marvellous [PPPC Utility](https://github.com/jamf/PPPC-Utility) to make a quick profile and upload it to my Jamf Pro Server. Just drag the ESET Endpoint Antivirus or Security app into it and select **Allow** for the **All Files** permission. Then save your profile and deploy with your MDM of choice, or if you have Jamf, upload it directly to your Jamf Pro Server (that feature is particularly nice!). Here's what PPPC Utility should look like if you've set things up properly:

![](/assets/2019/01/23/eset.png)

[Click here](https://github.com/neilmartin83/configuration_profiles/blob/master/System%20-%20Privacy%20Preferences%20Policy%20Control%20-%20ESET%20Endpoint%20Antivirus.mobileconfig) for an example Configuration Profile for ESET Endpoint Antivirus that you can use with your MDM, if you like.

Be aware that I haven't done any testing with ESET Endpoint Security as we don't use it in our environment - the bundle ID is different as well, so my above profile won't work. If you do use Endpoint Security and know of other permissions it needs, give me a shout!
