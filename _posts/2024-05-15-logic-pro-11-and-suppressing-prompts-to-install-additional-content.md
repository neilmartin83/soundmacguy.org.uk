---
layout: post
title: "Logic Pro 11 and suppressing prompts to install additional content"
date: 2024-05-15
coverImage: "spinal_tap_-_up_to_eleven.jpg"
---

On May 13th, [Apple released Logic Pro 11.](https://www.apple.com/uk/newsroom/2024/05/logic-pro-takes-music-making-to-the-next-level-with-new-ai-features) This marks the first major update to one of the most popular digital audio workstation tools since 2013 - that was 11 years ago! And with new updates come new behaviours that might cause issues in environments where that software is deployed and managed. Logic Pro 11 is no exception and on first launch, we're greeted with a new welcome screen:

![](/assets/2024/05/15/screenshot-2024-05-15-at-00.45.41.png)

This is in addition to this dialog which has been around in previous versions, and can be suppressed (details below if that's not something you're familiar with...):

![](/assets/2024/05/15/screenshot-2024-05-15-at-00.45.16.png)

We will assume you're deploying additional content with a tool such as the fantastic [loopdown](https://github.com/carlashley/loopdown) so won't cover the specifics of that here. But for the record, in my testing, loopdown will download and install the entire library for Logic Pro 11 without anything missing.

Even with the complete library of content installed, upon launching Logic Pro for the first time, those dialogs will still present themselves. To suppress them both, deploy an MDM Configuration Profile with a custom payload as follows:

Domain: `com.apple.logic10`

Payload:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>DontShowAdditionalContentDowload</key>
	<true/>
	<key>RecentWhatsNewPanelVersion</key>
	<integer>99</integer>
</dict>
</plist>
```

The new key is `RecentWhatsNewPanelVersion` and we'll bump it up to a value that represents a version of Logic Pro that will likely never exist.

What are you waiting for? Go and make some awesome music! 🎸
