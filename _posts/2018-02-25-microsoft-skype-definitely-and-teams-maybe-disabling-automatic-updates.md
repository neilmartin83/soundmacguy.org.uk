---
layout: post
title: "Microsoft Skype (definitely) and Teams (maybe) - Disabling automatic updates"
date: 2018-02-25
coverImage: "teamsskypesquirrel.png"
---

With version 8.x of Skype and since the debut of Teams, Microsoft have been using the [Squirrel](https://github.com/Squirrel/Squirrel.Mac) framework to manage automatic updates of these applications. This is undesirable in lab/managed environments where users typically aren't local administrators, as they're often presented with a dialog like this, which they can't do much with, other than ask IT for help:

![SquirrelUpdate](/assets/2018/02/25/squirrelupdate.png)

If we're packaging and deploying these applications (which we normally would be in the environments we manage), then they're usually owned by root and can't be modified by standard user accounts. So, what is an admin to do?<!--more-->

It isn't possible to disable this via managed preferences or a configuration profile as the Squirrel framework doesn't provide a preference domain or preference for it. However, it turns out that it does have an environment variable we can set. **Bear in mind that setting this knocks out automatic updates for all apps that use Squirrel** (except those that [Nerf that shit so hard](https://github.com/Squirrel/Squirrel.Mac/issues/192#issuecomment-285703068)).

All you need to do is run this command, as the current logged in user:

```bash
/bin/launchctl setenv DISABLE_UPDATE_CHECK 1
```

You'll notice that upon restarting, the auto-updating-annoying-behavior will come back. To get this to stick, run the above command in a script at login with [Outset](https://github.com/chilcote/outset) or put it in a Launch Agent, like this one (copy to **/Library/Launch Agents**):

{% gist b7491abae358d445444f8606a2f12cd3 %}

Thanks to the awesome Tim Sutton for uncovering this with the Slack application a while back (before they changed it) - check out his post and please support the issue he raised on Github:

[https://macops.ca/disabling-squirrel-updates/](https://macops.ca/disabling-squirrel-updates/)

[https://github.com/Squirrel/Squirrel.Mac/issues/192](https://github.com/Squirrel/Squirrel.Mac/issues/192)

Thanks also to Rick Heil for the [idea](https://rickheil.com/disabling-auto-updates-on-slack-for-mac/) of putting this into a Launch Agent.

You can tell if an application uses the Squirrel framework by **ctrl+clicking** it, choosing **Show Package Contents**, then looking inside:

![SkypeSquirrel](/assets/2018/02/25/skypesquirrel.png)

Note that I have tested/verified that this works with Skype, which seems to check for and download updates on every launch. Teams seems to check for updates 15 minutes after launch, then every 45 minutes (or does it?), if you take a peek at line 54234 and beyond in **/Applications/Microsoft Teams.app/Resources/app.asar**:

```ini
// Check for updates 15 mins after app start and then every 45 mins
const INITIAL_CHECK = 15 * 60 * 1000;
const CHECK_FREQUENCY = 3 * 60 * 60 * 1000; // TODO (jhreddy) change the following from 3 hours to 45 mins.
```

_With_ this environment variable set, I noticed that Teams did download the update in the background 15 minutes after launch, but _did not_ prompt to install it. I confess that I didn't bother to test if Teams prompted to update 15 minutes after launch _without_ the environment variable set. It's late and I ran out of patience. ;-) If you know better (or worse), please drop a comment.

And yep, I know the icon at the top of this post is a chipmunk and not a squirrel... :-D
