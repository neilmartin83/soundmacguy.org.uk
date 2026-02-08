---
layout: post
title: "It's time to talk about time (time servers and configuration profiles)"
date: 2018-09-26
coverImage: "clocks.jpg"
---

For endpoint devices, making sure the date and time is set correctly is really important. If it isn't, you'll soon see things breaking that depend on certificates, authentication, kerberos and so on. And who wants to have the wrong time on their device anyway? Nowadays the best way to sort this conundrum is to sync our endpoints with a time server using NTP.

Apple do a pretty good job of making sure Mac computers sync to their time servers (time.apple.com, time.euro.apple.com etc) out of the box so if you're happy with using those servers, there's not much else to do. However, in some situations, you may need to use a different time server, perhaps your domain controller, if you're reliant on AD binding, or maybe you're on a security-conscious network that blocks access to external time servers. You might even need to specify more than one time server so your Mac computers can failover if they can't connect to the first server they try (e.g. if you're using your domain controller and the Mac goes home so it's off your network).

[Rich Trouton has an excellent post on how to achieve this via the command line](https://derflounder.wordpress.com/2011/09/23/setting-multiple-network-time-servers-from-the-command-line/). Since then, with macOS 10.12.4, Apple quietly slipped in a [Time Server configuration profile payload](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf). A question in #profiles on the MacAdmins Slack about how that might work with multiple time servers perked my interest, so here's what I found...

<!--more-->

The Time Server payload sits in the **com.apple.MCX** preference domain and it allows you to configure time server (**timeServer** key) and time zone (**timeZone** key). Here, I'll be looking at the **timeServer** key specifically. I'd be concerned that setting the time zone would stop users changing it, or having it change automatically based on location - a problem if they travel a lot, but fine for static desktops I suppose.

It turns out you can indeed specify multiple time servers if you separate them with a comma. For example, to set **uel.ac.uk** and **time.euro.apple.com** the key/value pair would look like this:

```
<key>timeServer</key>
<string>myserver.myorg.com,time.euro.apple.com</string>
```

It is important to note that time servers are contacted in the order they're set, so the first one should be the one you want to sync to the most, with others acting as failover in case the first can't be reached.

What's this profile payload actually doing? It's modifying **/etc/ntp.conf** directly!

If you **cat** out the file, you can see what it looks like before the profile is applied, with no change from the "out of the box" setting:

```
$ cat /etc/ntp.conf 
server time.euro.apple.com.
```

And here's what it looks like after the profile is installed:

```
$ cat /etc/ntp.conf 
server myserver.myorg.com
server time.euro.apple.com
```

Also note that in **System Preferences**, as well as the server addresses being locked, the **Set date and time automatically** option is also locked after the profile is installed:

![timeserverlock.png](/images/timeserverlock.png)

That means it's not possible to change this setting here. If it was previously disabled, it will be locked in that state. That may cause headaches. However, it is still possible to toggle it using the **systemsetup** command:

```
/usr/sbin/systemsetup -setusingnetworktime on
```

and

```
/usr/sbin/systemsetup -setusingnetworktime off
```

Finally, here's an example configuration profile you might want to modify/use in your environment:

https://gist.github.com/neilmartin83/4d946f80b0ec54df841b2edfb07ba90d
