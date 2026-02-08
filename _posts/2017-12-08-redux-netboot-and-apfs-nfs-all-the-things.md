---
layout: post
title: "Redux: NetBoot and APFS - NFS all the things!"
date: 2017-12-08
---

I don't know what I'm doing. I poke things until they work. Then, I keep poking them to see if I can make them work better.

In my [previous post](https://soundmacguy.wordpress.com/2017/12/08/netboot-and-apfs-nfs-all-the-things/), I looked at setting up NFS with BSDPy by installing the NFS kernel server. I had tried setting up a Docker container I'd found, namely the fuzzle/docker-nfs-server container, and failed. After doing some more digging, I found the [macadmins/unfs3](https://github.com/macadmins/unfs3) container that uses the userspace nfs server.<!--more-->

After much faffing, this is how I got a working NFS server. This builds on [Graham Gilbert's guide for BSDPy, Ubuntu and Docker](https://grahamgilbert.com/blog/2015/04/21/getting-started-with-bsdpy-on-docker/).

If you've followed my [previous post](https://soundmacguy.wordpress.com/2017/12/08/netboot-and-apfs-nfs-all-the-things/) then you can undo the damage by running (as root):

```
apt-get remove nfs-kernel-server
```

Then reboot (to free up TCP/UDP ports 111).

## Docker me up!

Now, create an **exports** file. I put mine in **/usr/local/docker/** \- the following command will do that (run as root):

```
echo "/nbi (ro)" >> /usr/local/docker/exports
```

Next, the following two commands (also run as root) will a) pull down the macadmins/unfs3 container and b) run it, serving up your /nbi folder over NFS.

```
docker pull macadmins/unfs3
```

```
docker run -d \
  --privileged \
  -p 0.0.0.0:111:111/udp \
  -p 0.0.0.0:111:111/tcp \
  -p 0.0.0.0:2049:2049/udp \
  -p 0.0.0.0:2049:2049/tcp \
  -v /usr/local/docker/nbi:/nbi \
  -v /usr/local/docker/exports:/etc/exports \
  --name unfs3 macadmins/unfs3 \
```

If you're using Graham's script, just add these commands to it in the right places. Here's the complete script I use in my environment (note I've left the macadmins/netboot-httpd container there so I can switch between NFS and HTTP):

https://gist.github.com/neilmartin83/46627e73cce5c1b7de36fbfc4604d1b3

There are a couple of other differences! Pay attention to line 45 where we tell BSDPy to use NFS as its protocol. Also notice that the line in Graham's original script containing **BSDPY\_NBI\_URL=http://$IP** that would be present if we were using HTTP, has been removed. I also set up TFTP to use a block size of 1468 bytes which helps 2016/2017 Touchbar MacBook Pros in my environment (that's another story).

Finally, don't forget to tell your NetBoot images to use NFS instead of HTTP! Look at the **NBImageInfo.plist** file in your NetBoot image's **.nbi** folder and change the **<TYPE>** key from **HTTP** to **NFS**:

```
<key>Type</key>
<string>NFS</string>
```

## Sources:

[https://github.com/macadmins/unfs3](https://github.com/macadmins/unfs3)
