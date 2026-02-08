---
layout: post
title: "COVID-19, Folding@home, MacAdmins and You"
date: 2020-04-13
coverImage: "coronavirus-800x450-1.jpg"
---

Coronavirus is wrecking lives throughout the world and scientists are working hard to find ways to fight it. As MacAdmins, [we can help](https://foldingathome.org/covid19/). If you've seen images of the virus itself, you'll notice it's strewn with lots of little "spikes". Known as the "Demogorgon" (scientists come up with cool names!), these protein structures act like a "key" to unlock our cells, allowing the virus to get inside them and do its work.

I'm no biologist by any means, and [these folks explain it way better than I ever could](https://foldingathome.org/2020/04/03/capturing-the-covid-19-demogorgon-aka-spike-in-action/). Understanding how these proteins behave, or "fold" is the key to developing ways to stop them. The [Folding@home](https://foldingathome.org/) project has been running for many years and now it's providing the distributed compute power needed to run simulations that help gain this understanding in the context of COVID-19.

By [downloading](https://foldingathome.org/start-folding/) (and optionally configuring) a client, your Mac (or Macs) can help in the war effort. You can help anonymously, or set up an identity to keep track of your stats (the number of work units you've processed and points you've gained). You can even join a team where your stats are accumulated with other members'. Folks in the MacAdmins community have stepped up to donate computing resources already. [Eric Holtam](https://osxbytes.wordpress.com/) has created the [MacAdmins Folding@home Team](https://stats.foldingathome.org/team/257618), and we have a channel; #foldingathome in the [MacAdmins Slack](https://www.macadmins.org/), ready to rock the lockdown.

**If you want to get involved:**

1. Join us in the **#foldingathome** channel on the [MacAdmins Slack](https://www.macadmins.org/).
2. [Download](https://foldingathome.org/start-folding/) and install the client.
3. [Get your passkey](https://apps.foldingathome.org/getpasskey).
4. Join the MacAdmins Team (ID **257618**).
5. Read the rest of this post.

<!--more-->

## Can I mass deploy the client?

Yes. If deploying this across your fleet of Macs tickles your pickle, read on... The client comes as a signed, notarized (although not notarized for more recent versions since the time of writing this) package:

<figure>

![](/images/image.png)

<figcaption>

Signed and notarized! But not for the latest version...

</figcaption>

</figure>

The client binary itself installs in **/usr/local/bin** along with applications to configure it and view your stats. We also get a LaunchDaemon to start the client when your Mac boots.

The **postinstall** script is deployment-friendly too. It makes use of detecting a command line install (i.e. deployment by tools like ARD, Jamf or Munki etc) and won't open your web browser after installation as it otherwise would had you installed the package via the GUI.

Here's a snippet that shows how it will exit when installed by command line (the part which opens your browser is after this and won't run!).

```
#Don't launch GUI if CLI install
[ "$COMMAND_LINE_INSTALL" == "1" ] && exit 0
```

Vendors take note, this is a great example of how packaging your applications [should be done](https://wiki.afp548.com/index.php/Guidelines_for_Mac_software_packaging)!

The client runs a local web server on port **7396** where you can see more information about the current Work Unit being processed, as well as configure some basic settings. Security-wise, the web server only accepts connections from **localhost** (the computer it's running on). Trying to connect to it from a different computer on the same network will yield a 403 error.

![](/images/image-1.png)

## What about mass-configuring it?

The Folding@home client gets and sets configuration options in the file:

**/Library/Application Support/FAHClient/config.xml**

For a system like a Mac Mini, without an external GPU (so not using GPU acceleration), configured only to join a team and set a username, the file looks like this. It has things you can modify:

https://gist.github.com/neilmartin83/8bc55e8dce4c945b8ac1bf431f2e5d50

The **<!-- User Information -->** section (lines 5-8) has everything you need to join a team and set your username:

**<passkey v='xxxxxxxxxxxxxxxxxxxxxxxxx'/>** : This is a unique key that identifies you. Replace the x's with a passkey you get from [https://apps.foldingathome.org/getpasskey](https://apps.foldingathome.org/getpasskey). You must have a passkey in order to set a username and team.

**<team v='xxxxxx'/>**: The team ID you want to join. Leave this set to 0 if you don't want to join a team. The **MacAdmins** team ID is **257618** - go on... you know you want to!

**<user v='xxxxxxxxxxxx'/>**: Your username. 'nuff said.

On a basic level, the client can also be set to run with different levels of **Folding Power**; **Light**, **Medium** (default) and **Full**. This essentially means how much CPU/GPU resource it uses, along with how aggressively it's used. [There's a good explanation of Folding Power here](https://foldingathome.org/faqs/fah-v7/v7-introduction/web-control/folding-power-slider/).

Along with **Folding Power**, you can configure whether the process runs all the time, or only if the computer is idle.

As an example, if we set a more conservative configuration with **Folding Power** to **Light** and **When** to **Only when idle**, our **config.xml** will change to look like this:

https://gist.github.com/neilmartin83/11774203fb3db06273a5db37d0b53b9d

We have a new **<!-- Slot Control -->** section (lines 5-6)

```
<!-- Slot Control -->
<power v='LIGHT'/>
```

To set **Folding Power** to **Full**, changing the **power v** tag value from **'LIGHT'** to **'full'**. To set **Folding Power** to **Medium**, delete this tag altogether.

The **<!-- Folding Slots -->** section has also changed (lines 13-16):

```
<!-- Folding Slots -->
<slot id='0' type='CPU'>
<idle v='true'/>
</slot>
```

The **idle v** tag controls whether the process is running at idle (**'true'**), or all the time (**'false'**).

When you make a configuration change, you must restart the client for it use the new settings. That means reloading the LaunchDaemon (as root):

```
launchctl unload /Library/LaunchDaemons/org.foldingathome.fahclient.plist
launchctl load /Library/LaunchDaemons/org.foldingathome.fahclient.plist
```

I'm running Folding@home on a 2014 i5 Mac Mini with the default settings. It produces some heat but not enough to make the internal fan spin loudly (I can't hear it). The body of the Mac Mini itself is lukewarm to the touch. I also don't notice any performance hits in day-to-day usage (web browsing etc).

If you perhaps take the bold step to deploy Folding@home in a lab of 20 Macs, it's worth considering how you'll configure it, keeping in mind the heat produced, energy usage and user experience.

## More?

This blog post only scratches the surface. Folding@home has [many more things you can configure](https://foldingathome.org/support/faq/installation-guides/configuration-guide/). There are also [command line options](https://foldingathome.org/support/faq/installation-guides/mac/command-line-options/) if you want to interact with the client more directly.

[The FAQ](https://foldingathome.org/support/faq/) covers everything else in great detail and is well worth reading.
