---
layout: post
title: "Suppress the upgrade dialog for Keynote, Numbers and Pages"
date: 2026-01-29
---

Following [Apple's release of new versions of these apps](https://soundmacguy.wordpress.com/2026/01/28/keynote-numbers-and-pages-15-1-now-unified-across-all-apple-operating-systems/), users will be presented with the following dialog upon opening the previous versions:

![](/images/image-5.png)

To see this dialog, the following conditions apply:

- The previous version has updated to its latest release (14.5)

- The new 15.1 version has not been installed

If the user clicks "Not Now" the dialog does not reappear on subsequent relaunches. However, it may reappear in future as the preferences that control it are time/counter based.

<!--more-->

Administrators may wish to suppress this dialog in some scenarios, for example:

- Shared computers that must retain the previous versions for consistency/experience

- Environments where users cannot use the App Store to install new versions by themselves

- Situations where administrators will be managing the migration to the new versions and wish to avoid this user-facing dialog

**The following configuration profile will prevent this dialog from appearing:**

{% gist 0e8b320cf9f95f89260292dd8a6b66f1 %}

Please be aware - users will still see prompts regarding collaboration not working in the old versions if they try to use this feature. In these cases, I would argue that they need to be aware and be upgraded as soon as possible.
