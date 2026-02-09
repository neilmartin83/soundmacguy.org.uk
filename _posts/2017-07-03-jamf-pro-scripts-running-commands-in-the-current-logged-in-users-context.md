---
layout: post
title: "Jamf Pro Scripts - running commands in the current logged in user's context"
date: 2017-07-03
---

I've already been using this technique for a while but today, thanks to our fantastic Mac Admins community, I've learned a little bit more about it, so it might be worth a blog post.

One interesting thing about [Jamf Pro](https://www.jamf.com/products/jamf-pro/) is that it can execute scripts during a policy run. Scripts executed this way are run as the root user, which is all well and good if you need to do stuff to the system as a whole with elevated privileges. But what if you need to run a command _as if it's being run by the current logged in user themselves_ as part of a policy? One example would be to use a utility like [mysides](https://github.com/mosen/mysides) to configure a their sidebar, or if you want to invoke [lsregister](https://ss64.com/osx/lsregister.html) to register an application so that user doesn't see something this the first time it's launched (kudos to @franton on the [MacAdmins Slack](https://macadmins.herokuapp.com/) for pointing out that this tends to be more of an issue for applications living outside /Applications as macOS takes care of those automatically, but I digress):

![audaemon](/images/audaemon.png)

<!--more-->

For a working solution, we need to do two things:

1. Determine who the current logged in user is and set that as a variable
2. Run your desired command as that user.

## Determine the current logged in user

{% gist e6751d1a3571e1d8da40bbe8d02e90be %}

Thank's to @macmule for this one - [click here to learn more](https://macmule.com/2014/11/19/how-to-get-the-currently-logged-in-user-in-a-more-apple-approved-way/).

## Run your command as that user

{% gist 040326412c1a4620c79b2cb69eda7723 %}

Keep your command in quotes and rinse and repeat that line for every further command.

There are a couple of commands that accomplish this, and after a nice little debate in #jamfnation on the [MacAdmins Slack](https://macadmins.herokuapp.com/), it tends to boil down to personal taste as to which is the best and why (and where not to put hyphens - thanks @franton and @dog for pointing that out!).

Here's an snippet of a script that runs [dockutil](https://github.com/kcrawford/dockutil) to configure a user's dock:

{% gist 5ec5ffb22ada1720fffbc07c5e79d27e %}

As a footnote, it's worth noting that you can run scripts as the current logged in user with a fantastic tool, [Outset](https://github.com/chilcote/outset), that's used by many. This post is a way to achieve a similar goal if you have Jamf Pro and want to use its built-in framework. Skinning cats and all that...
