---
layout: post
title: "Memcached - disabling UDP on Ubuntu"
date: 2018-03-01
coverImage: "memcached.png"
---

Another day, another vulnerability! This time, it's memcache's turn! [Read all about it here](https://blog.cloudflare.com/memcrashed-major-amplification-attacks-from-port-11211/).

So if you're hosting your own Jamf Pro Server in a clustered environment, you've probably got memcached running. You'll want to disable UDP access to it in order to mitigate against this vulnerability. Jamf Pro doesn't use UDP for memcached anyway. Here's what I did for my memcached endpoint running Ubuntu 16.04 LTS (the package for memcached is version 1.4.25 at the time of writing).<!--more-->

## Check if you're vulnerable:

Log into your memcached server with an account that has sudo access and run this command:

```bash
echo -en "\x00\x00\x00\x00\x00\x01\x00\x00stats\r\n" | nc -q1 -u 127.0.0.1 11211
```

If you see any output such as this (and more), then memcached is listening on UDP port 11211 (its default):

```bash
STAT pid 1049
STAT uptime 143063
STAT time 1519915398
STAT version 1.4.25 Ubuntu
STAT libevent 2.0.21-stable
...
```

Time to disable that bad boy!

## Disable UDP!

Back up the file**/etc/memcached.conf** just in case:

```bash
sudo cp /etc/memcached.conf /etc/memcached.conf.old
```

Edit the original in your favourite text editor (**nano** for me!):

```bash
sudo nano /etc/memcached.conf
```

Add these lines to the end - don't change anything else:

```conf
# Disable UDP
-U 0
```

Save the file **(hit CTRL+X then type Y and return)** and then restart the memcached service with this command:

```bash
sudo service memcached restart
```

Then go back to the beginning of this post and run the command to check for the open UDP port again. You should get no output this time.

You don't need to stop Tomcat on your Jamf Pro Servers whilst you do this and you shouldn't notice any outage.

Rinse and repeat if you have multiple memcached endpoints.
