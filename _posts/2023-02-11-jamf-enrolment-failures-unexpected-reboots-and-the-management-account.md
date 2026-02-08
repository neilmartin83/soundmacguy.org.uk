---
layout: post
title: "Jamf Enrolment Failures, Unexpected Reboots and the Management Account"
date: 2023-02-11
coverImage: "management.png"
---

When is a local administrator not a local administrator? When it's a Management Account. When is a Management Account not a Management Account? When it's a local administrator... it seems macOS doesn't like this conundrum...

Earlier last week, following an update to Jamf Pro 10.43, my colleagues and I started to notice an uptick in failed macOS enrolments. It turns out we weren't the only ones - others in the community were experiencing the same thing. The failure condition is quite specific and we could repeat it fairly consistently (on most attempts). Essentially:

1. Start with a clean/new shiny Mac that's in Apple Business/School Manager and assigned to a Jamf Pro PreStage, ready to enrol.

3. Power up and reach the Setup Assistant as usual.

5. Proceed past the Region/Language Choosers and hit the Remote Management screen.

7. Select the continue button.

9. Watch as the MDM Enrolment Profile is installed, followed by any other profiles, if they've been specified in the Pre Stage.

11. Mac shuts down and / or reboots(!)

13. We see the Login Window, showing an avatar for our local administrator account we created in the Pre Stage settings (it's also the same username as the Jamf Management Account).

15. The computer record in Jamf Pro is incomplete, with much of its data missing (because it's only received telemetry from Apple's `mdmclient` and not the `jamf` binary at this stage). There's no **Policy history** but we can see MDM commands have been sent successfully in the **Management history**...

17. Scream into a pillow.

<!--more-->

## What's going on here?

To answer that question, it's time to dive into the logs and look for clues! Specifically, `/var/log/jamf.log`

Some lines stand out:

Jamf tries to create the Management Account:

```
Wed Feb 01 09:10:36 MacBook Air jamf[899]: Creating user _jssadm…
```

Bad things happen:

```
Wed Feb 01 09:12:01 MacBook Air jamf[317]: The SSL Certificate for https://my.jamf.server must be trusted for the jamf binary to connect to it.
Enrolling computer…
Wed Feb 01 09:12:03 MacBook Air jamf[335]: Skipping trustJSS command…
Wed Feb 01 09:12:03 MacBook Air jamf[335]: An error occurred while enrolling computer: Permission Error - The user specified does not have permission to perform the action.
Wed Feb 01 09:12:03 MacBook Air jamf[335]: Restoring JAMF.keychain since an error occurred.
Wed Feb 01 09:12:03 MacBook Air jamf[335]: Error Domain=com.jamf.jamfsecurity.error Code=-25300 "searchForItems:conversionBlock:error: : The specified item could not be found in the keychain." UserInfo={NSLocalizedDescription=searchForItems:conversionBlock:error: : The specified item could not be found in the keychain.}
Wed Feb 01 09:12:03 MacBook Air jamf[335]: Security Error - A security error has occurred.
Wed Feb 01 09:12:03 MacBook Air jamf[335]: Error Domain=com.jamf.jamfsecurity.error Code=-25300 "searchForItems:conversionBlock:error: : The specified item could not be found in the keychain." UserInfo={NSLocalizedDescription=searchForItems:conversionBlock:error: : The specified item could not be found in the keychain.}
Wed Feb 01 09:12:03 MacBook Air jamf[335]: Device Signature Error - A valid device signature is required to perform the action.
Wed Feb 01 09:12:03 MacBook Air jamf[335]: Removing existing launchd task /Library/Application Support/JAMF/tmp/com.jamfsoftware.task.policy.plist…
Wed Feb 01 09:12:04 MacBook Air jamf[335]: Enroll return code: 70
Wed Feb 01 09:12:04 MacBook Air jamf[397]: Checking for policies triggered by "enrollmentComplete"…
Wed Feb 01 09:12:05 MacBook Air jamf[397]: Error Domain=com.jamf.jamfsecurity.error Code=-25300 "searchForItems:conversionBlock:error: : The specified item could not be found in the keychain." UserInfo={NSLocalizedDescription=searchForItems:conversionBlock:error: : The specified item could not be found in the keychain.}
Wed Feb 01 09:12:05 MacBook Air jamf[397]:
There was an error.

    Device Signature Error - A valid device signature is required to perform the action.
```

What went wrong? It's not clear, but we can see some problems...

1. There's a permission issue that seems to stop the "user specified" (who is that?) from completing the enrolment.

3. It looks like the `jamf` binary had problems working with a certificate it needed from a keychain in order to enrol the computer.

5. Communication between the Jamf Pro server and the computer wasn't trusted, and therefore not allowed (the `jamf` binary won't do anything!).

Focusing on point 1 above - could the "user specified" be the Jamf Management Account? Digging further, I noticed this:

- Jamf Pro was configured to create a Management Account

- The Computer PreStage Enrolment was configured to **Create a local administrator account before the Setup Assistant**.

- Both accounts had the same username (\_jssadm).

Could we be hitting a race condition where the jamf binary tries to create the Management Account even though the local administrator with the same username is already there? And if that happened, could that make macOS go crazy?

## Wait, what's the Management Account and why does it matter?

The Management Account has been a staple in Jamf, and previously the Casper Suite for a long time. Its original purpose was to have an account on the computer that could be used for SSH access for Jamf Remote, a tool used to run ad-hoc policies and start tunnelled Screen Sharing (VNC) sessions on computers connected to the same local network. Jamf Remote is gone, but now the Management Account lives on as a means to allow management of FileVault enabled users. [Read more about it here](https://learn.jamf.com/bundle/jamf-pro-documentation-current/page/Management_Accounts.html).

You create this account (if you want to), in **Settings > User-initiated enrolment > macOS**

What's important is this:

- You should **Enable user-initiated enrollment for computers** and enter credentials for the Management Account. The computer record needs that information for Jamf to consider it as "Managed" and send MDM commands etc.

- You don't have to create the Management Account if you don't need a workflow for FileVault users as described above. The Management Account does not need to exist on the computer at all!

## Solutions?

If you create a Management Account and a local administrator account in the PreStage Enrolment settings, they must not share the same username. Think Highlander - _there can be only one_!

Choose one option (and choose wisely, depending on your enrolment, local administrator and FileVault workflows, because they may be impacted):

1. Change the username of the Management Account in **Settings > User-initiated enrolment > macOS**, or your local administrator account in the **Computer PreStage Enrolment** setting so they don't match.

3. Disable the option to Create management account in **Settings > User-initiated enrolment > macO**S

5. Disable the option to **Create a local administrator account before the Setup Assistant** in the **Computer PreStage Enrolment** settings.

7. Stop creating both accounts altogether and re-evaluate your life choices? ;-) Use FileVault Recovery Keys if you need to gain access to a computer without the user being present. Consider using a tool like [Privileges](https://github.com/SAP/macOS-enterprise-privileges) to elevate local administrator rights.

If you scour the MacAdmins Slack and Jamf Release Notes, you may find a few Product Issues that relate to account creation at the Setup Assistant, but I couldn't see anything specific that would describe the actual failure condition seen here.
