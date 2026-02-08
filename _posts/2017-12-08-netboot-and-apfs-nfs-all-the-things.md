---
layout: post
title: "NetBoot and APFS - NFS all the things!"
date: 2017-12-08
coverImage: "netboot.png"
---

**Edit: [I found a better way do do this with Docker!](https://soundmacguy.wordpress.com/2017/12/08/redux-netboot-and-apfs-nfs-all-the-things/) Still, this might be useful if you want to install the NFS server on the host itself...**

Just writing this down before I go completely nuts...

[NetBoot](https://en.wikipedia.org/wiki/NetBoot) - the gift that keeps on giving! So, it turns out, in my environment at least, you can't NetBoot a Mac that has an APFS formatted volume. My NetBoot environment is set up around [Pepijn Bruienne's awesome BSDPy](https://github.com/bruienne/bsdpy) like this:

- Ubuntu Server 16.04 VM host
- Docker containers providing these services:
    - BSDPy
    - TFTPD
    - HTTP

I set my server up following [Graham Gilbert's excellent guide](https://grahamgilbert.com/blog/2015/04/22/getting-started-with-bsdpy-on-docker/).

Booting both [NetBoot](https://help.apple.com/serverapp/mac/5.4/#/apdF4195967-3D58-4562-A715-5AB3632DCC04) and [NetInstall](https://help.apple.com/serverapp/mac/5.4/#/apd2181B01F-1CBE-4C19-B0D0-C49C9C3A4E64) sets running either macOS 10.12.6 or 10.13.2 just flat out didn't work. I didn't look at [NetRestore](https://help.apple.com/serverapp/mac/5.4/#/apdBE6DAA30-A254-4421-A7C2-8D1EEBFF039F) because imaging is dead, mkay?

DHCP/BSDP stage - fine. TFTP stage - fine. Then the progress bar that signifies the HTTP bit appears. And stays. Forever.

<!--more-->

Killing the APFS volume and re-partitioning the drive with good old GPT and HFS+ resulted in NetBoot working properly for both versions of macOS. I didn't want to break the news to our technicians that they couldn't NetBoot a newly purchased Mac (because we don't want 10.13 yet and want to revert it to 10.12!), without first booting it into recovery and nuking the drive first. Actually, you have to do this regardless because you need 10.13's Disk Utility to do the deed (but it still leaves the issue of being unable to boot an APFS Mac to a 10.13.x NetInstall...).

It turns out, after some trial and error, that it's HTTP's fault. Serving NetBoot images over NFS instead works perfectly. And it seems to be faster.

N.B You do sacrifice a bit of security as with NFS, even though your exports (shares) are read only, you can mount the directory and see its contents from clients. If you've got anything sensitive in your NetBoot images, this might be a concern. Anyway...

## NFS on Ubuntu, how do I do?

The point of this post is a quick and dirty how-to for getting NFS up and running with BSDPy in Ubuntu (switching over from HTTP). At first, I wanted to have NFS running in a Docker container but it didn't work for me (because I'm rubbish at Docker, probably). I went the easy route and installed it on the host itself. Here's what I did:

Run these commands as root:

1. Install the NFS server goodness:
    
    ```
    apt-get install nfs-kernel-server
    ```
    
2. Create a symbolic link of wherever your NBIs live (on my server they're in /usr/local/docker/nbi) to /nbi
    
    ```
    ln -s /usr/local/docker/nbi /nbi
    ```
    
3. Serve up your /nbi directory by editing the /etc/exports file:
    
    ```
    echo "/nbi *(async,ro,no_root_squash,insecure)" >> /etc/exports
    ```
    
4. Restart the NFS service:
    
    ```
    service nfs-kernel-server restart
    ```
    
5. Check that NFS is working properly - It's easy on a Mac: In **Finder**, click the **Go** menu, choose **Connect to Server...** and enter the URL: **nfs://your.server.ip.address/nbi** - you should be able to see your NetBoot images.
6. Configure BSDPy: Change these options and restart it (this assumes you've followed Graham's guide - otherwise it shouldn't be too difficult to work things out in your environment):
    1. Add **BSDPY\_PROTO\=****nfs**
    2. Remove **BSDPY\_NBI\_URL\=http://$IP** (don't set this one at all)
7. Profit!

## Sources:

[https://help.ubuntu.com/community/SettingUpNFSHowTo](https://help.ubuntu.com/community/SettingUpNFSHowTo)

[https://github.com/bruienne/bsdpy/blob/master/README.md](https://github.com/bruienne/bsdpy/blob/master/README.md)
