---
layout: post
title: "Adobe Shared Device Licensing - the 90 minute sign-in reminder is dead!"
date: 2019-09-03
coverImage: "sdl-account-confirmation.png"
---

A major pain point of Adobe's new Shared Device Licensing (SDL) is was a dialog that would appear 90 minutes following sign-in to a Creative Cloud application. This dialog would prompt the user to confirm that they were still themselves, offering the option to sign-out or continue.

It would also interrupt background processing and rendering in applications like After Effects and Premiere Pro. This impacts people in environments where they would quite reasonably leave a video render churning away overnight, for example. I support shared-use video edit suites where students do this.

On 2019-08-23, Adobe announced, [via this forum thread](https://forums.adobe.com/thread/2648036), that the infamous dialog is no more! Although the official [Shared Device Licensing Deployment Guide](https://helpx.adobe.com/lv/enterprise/using/sdl-deployment-guide.html) is now out of date as it still has an FAQ that mentions it...

Even better, is this appears to apply to installations that pre-date the announcement. I tested it specifically with Premiere Pro 13.1.4 and Photoshop 20.0.4 and an SDL "license only" package I created in July. For those playing at home, the Creative Cloud Desktop App (CCDA) was the current version 4.9.0.504, having auto-updated itself. I left the applications open for over 90 minutes. The dialog did not appear during app usage or if I closed and re-opened an app. It didn't come back when opened a different Creative Cloud app either.

Thank you, Adobe!
