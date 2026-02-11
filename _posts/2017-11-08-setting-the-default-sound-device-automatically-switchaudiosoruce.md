---
layout: post
title: "Setting the default sound device automatically - switchaudiosource"
date: 2017-11-08
---

We've got Macs in lots of weird and wonderful places like recording studios and video editing suites. Our Macs are bound to Active Directory so any student can log in with their university ID and get to work. These specialist areas often use a third party sound card/audio interface, so the first thing students should do is set _that_ as their default device in **System Preferences, Sound.** As well as having the usual stuff like web browsers and media players route audio through that lovely device, applications like **GarageBand** and **Logic Pro** will set their audio input/output to whatever you choose there too. Enter **switchaudiosource**.<!--more-->

# Using switchaudiosource

**Switchaudiosource** is a command line utility you can use to programatically set the input, output or system audio device. It works with macOS 10.12 (and might work with 10.13 - I haven't tested it yet - let me know if you have). [Here's its Github page with instructions for installing it with](https://github.com/deweller/switchaudio-osx) [**brew**](https://github.com/deweller/switchaudio-osx). I've packaged the binary [here](https://github.com/neilmartin83/switchaudiosource-pkg/raw/master/Dweller_SwitchAudioSource_1.0.0.pkg) in case you don't want do bother with brew. The shasum for my package is 46da82947bdf68423738b97d0a4cee06cd037598. My package installs **switchaudiosource** in **/usr/local/bin**.

Using **switchaudiosource** is easy and it gives good feedback. Type the command by itself to get help:

```bash
switchaudiosource
```

![sas1.png](/assets/2017/11/08/sas1.png)

List all audio devices currently connected:

```bash
switchaudiosource -a
```

![sas2.png](/assets/2017/11/08/sas2.png)

Set the output device to something specific, using a name from the list above (in this example, our RME RayDAT card - what an awesome bit of kit!):

```bash
switchaudiosource -s "HDSPe RayDAT (23635536)"
```

![sas3.png](/assets/2017/11/08/sas3.png)

Set the input device to something specific:

```bash
switchaudiosource -t input -s "HDSPe RayDAT (23635536)"
```

![sas4.png](/assets/2017/11/08/sas4.png)

Tell us the current default device:

```bash
switchaudiosource -c
```

![sas5.png](/assets/2017/11/08/sas5.png)

# Scripting switchaudiosource

You could invoke **switchaudiosource** with a script and run it at user login. This would automatically set the default audio device silently, with the user blissfully unaware. I use a Jamf Pro login policy and make sure that **switchaudiosource** runs in the context of the current logged in user. You could also use [Outset](https://github.com/chilcote/outset) which gives you the ability to run the entire script as the current logged in user, so no need for **sudo -u**.

Here's my Jamf Pro login policy example:

{% gist 672ff01679c9a0c3aef7a8f0f2a6099b %}

Because I use **switchaudiosource** in a few different spaces that use different audio devices, I make use of [Jamf Pro's parameters](https://www.jamf.com/jamf-nation/articles/146/script-parameters), with **$4** being the output device and **$5** being the input:

{% gist f57ccea6c38b6c9cc102254b2a1e0008 %}

![sas7.png](/assets/2017/11/08/sas7.png)

That way, I only need one script in my Jamf Pro Server.
