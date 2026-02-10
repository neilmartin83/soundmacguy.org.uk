---
layout: post
title: "Bypassing the SecureToken dialog for mobile accounts"
date: 2018-06-02
coverImage: "st-dialog.png"
---

Ahh [SecureToken](https://derflounder.wordpress.com/2018/01/20/secure-token-and-filevault-on-apple-file-system/); the gift that keeps on giving! macOS 10.13.4 introduced this new, undocumented dialog that would appear on first login under the following conditions:

- If the filesystem is APFS
- _Whether or not_ FileVault is enabled
- If the Mac is bound to a directory service (e.g. Active Directory or LDAP)
- If there is a local administrator account present that has logged in at least once (e.g. the one created during the Setup Assistant).
- If the account currently logging in will be a directory based mobile account (i.e. it hasn't been created yet and is logging in for the first time)

The text reads:

> **Enter a SecureToken administrator's name and password to allow this mobile account to log in at startup time.**
> 
> You can Bypass this to continue creating your mobile account, but you may not be able to log in with this account when the computer starts up until your administrator resolves this issue.

<!--more-->

I think Apple has good intentions with this dialog, especially if you're using mobile accounts in conjunction with FileVault - this gives us an easier way to grant users SecureToken without having to use **sysadminctl**. But the problems with it are this:

1. The context of SecureToken is quite technical, this dialog would be confusing to a lot of people.
2. If you click **Bypass**, your mobile account is created and you'll _never see this dialog again._ If the Mac is encrypted with FileVault, you won't be able to unlock it with your mobile account credentials. Moreover, if the Mac is not encrypted with FileVault, you won't be able to _enable_ it with that mobile account.
3. **For shared Macs in lab settings that are not encrypted with FileVault and _never will be_, this dialog is not necessary, as well as confusing.**

With the macOS 10.13.5 update, Apple quietly gave us a preference key to suppress it:

The domain is **com.apple.MCX** and the key is a boolean: **cachedaccounts.askForSecureTokenAuthBypass** 

Set it to **TRUE** and you're done!

Here's a Configuration Profile that would do it:

{% gist 250e2e7a959b03fd1076a868959635da %}

As a footnote, another way to avoid this dialog would be to switch to network based directory accounts. However, that would mean that the Mac must always be able to reach your directory service in order for users to log in, since network accounts aren't cached (even if their home areas are local and do persist on disk).

Also - I have observed that this dialog does not present when binding to a directory as part of the DEP Pre-Stage, skipping the local user account creation and logging in with a directory account as the very first user. Logging in with a second, different directory account following this does not present the dialog either. I observed this behaviour whether or not FileVault was enabled. Because of this, the following workflow is possible:

1. The technician proceeds through the Setup Assistant and enrols the Mac via DEP.
2. Mac binds to Active Directory as part of the DEP Pre-Stage Enrolment configuration and skips initial user account creation.
3. The technician logs in with their directory account to kick off our provisioning workflow (naming the Mac and choosing its role - staff or student use).
4. The Mac restarts and intended user logs in with their directory account.
5. FileVault is enabled for the user automatically by policy and the boot drive encrypts successfully without requiring any form of SecureToken granting.
6. The user account has SecureToken but the technician account does not (good!). Only the user account is shown on the FileVault pre-authentication boot screen.
