---
layout: post
title: "Adobe Shared Device Licensing - Answers to questions you probably didn't want to ask"
date: 2019-02-01
coverImage: "adobesdl.jpg"
---

Shared Device Licensing (which I'll refer to as **SDL** from now on) for Adobe Creative Cloud is here! If you want the skinny on it, go check out [Mr Mule's fantastic write up](https://macmule.com/2019/01/30/adobe-shared-device-licensing/).

So yesterday, it showed up in my Admin Console. Naturally, I spent much of my time mucking about with it, trying out all sorts of experiments to see what the results would be, or if something would break. Here's an FAQ-style breakdown...

<!--more-->

### Do users have to sign in to use SDL apps?

Yes, SDL apps only function when a user signs in with an Adobe ID, Enterprise ID or Federated ID. You can't use them otherwise, at all.

Adobe state that for schools (K-12 in the US and primary/secondary in the UK), only Enterprise or Federated IDs are supported. They also state that users under the age of 13 cannot use personal Adobe IDs.

You can restrict which kinds of ID are users allowed to sign in with via the Admin Console. On top of this, you can also restrict IDs by domain and originating public IP address (e.g. so apps only activate from your organisation's network).

### If you have to sign in to use the app, and the sign-in is a named person, how is SDL different from named-user licensing?

Signing into an SDL app will not count against that person's own named user license (NUL) activations (i.e. it won't prompt them to sign out if they're signed into other computers with their NUL). SDL is licensing the apps at the device level. The apps will work regardless of whether the ID signing in has named licenses assigned or not. No matter how many different users sign into the apps on the same computer, The SDL license count against that computer will always be 1.

As some named license products include more cloud storage, users will see a difference there. E.g. a user with the 100GB All Apps plan will be able to use their 100GB of cloud storage, whereas a user with the 2GB Spark for Education plan will have 2GB. SDL only concerns the apps, not cloud storage/services.

### I use federated Adobe IDs for my users. My Macs are AD-joined, or I have NoMAD, so they have a Kerberos ticket. Does SDL leverage this to automatically sign in/activate?

Sort of. This is assuming you're all set up with federated IDs and you have working SSO between your IdP (e.g. ADFS/Azure) and Adobe. The first time you launch an app (e.g Photoshop), you'll see the regular sign-in window. If you enter _any_ email address with your domain on the end (even with a user that doesn't exist), you're punted over to your IdP's sign-in page where the Kerberos ticket for the _user who's logged in to the Mac_ is automatically used instead. No password prompt, straight through. You can't actually sign in as any other federated user unless you blast things with **kdestroy**.

The apps behave as above if you sign in directly and the Creative Cloud Desktop App (CCDA - the bit that sits in the menu bar) will sign in by itself after you sign in to an app. BUT - if you try to sign into the CCDA itself first, it doesn't use the Kerberos ticket. You'll just get redirected to your IdP's sign-in page and have to enter your credentials there to proceed.

### What happens if I install an SDL package over the top of an existing serialized installation - e.g. the 2018 apps and older?

The 2019+ apps will work with SDL (you need to sign into them with an Adobe ID to activate). Officially, the serialized older apps _should_ revert to Named User Licensing and adopt anylicenses you have that are assigned to the Adobe ID you signed in with (and revert to Trial mode if that Adobe ID doesn't have any licenses).

However, I had mixed results when I tried this with an Adobe ID that didn't have any named licenses assigned. On one occasion, the serialized license seemed to remain and I was able to use the 2018 apps. I also ran the **AdobeExpiryCheck** tool and it reported that the serial number was still present with its original expiry date; some time in 2021.

In all honestly, you shouldn't expect 2018 and older serialized licenses to work at all after installing 2019 SDL apps side by side.

### What if I install an SDL package over an existing Named User Licensed (NUL) installation of 2018 apps or older?

The 2019+ apps will work as SDL. The 2018 apps and earlier will still work as NUL - i.e. if the Adobe ID you're signed in with does not have a license assigned, they run in trial mode.

### What if I install a NUL 2019 app package over an existing SDL installation of other 2019 apps?

The 2019 app will work with SDL too, nice. I tested this by installing NUL Premiere Pro 2019 on top of SDL Photoshop 2019.

### Can I build a "blank/no-apps" SDL package? Can I install it on top of a NUL install of other 2019 apps? Does it convert them to SDL?

Yes, yes and yes.

### What happens if I run the uninstaller package you get as part of the SDL package?

Any apps that package installed are removed. The SDL is also removed. CCDA is left behind. I haven't tested to see how it handles a situation where one or more SDL apps are left behind (e.g. if you created a package for Photoshop, and uninstall that, but also have Premiere installed on that Mac from a different SDL package). Is it clever enough to leave the SDL alone in that case? I wonder...

### What about Device Pool (or Device Licenses)?

I don't know for sure - I haven't got these in my environment. But folks on the Slack have said they saw a "Migrate" button in their Admin Console and were warned that after switching over the SDL, their Device Licenses would expire in 30 days. I thought it worth noting as they often get confused with Enterprise serialized licenses. They're not the same thing and it's really important to understand which one you have.

[Read Ben's post for more about this.](https://macmule.com/2019/01/30/adobe-shared-device-licensing/)

### In my lab, if user Jane logs in and activates SDL, then logs out, followd by user John logging in next, what happens?

John has to sign in with his Adobe ID the first time he runs an app in order to get the SDL goodness. Activations don't traverse user accounts.

### What if Jane comes back to the same Mac. Does she have to sign in to activate SDL again?

Providing her account is still there (I'll come to what happens if you delete accounts soon), SDL will be "cached" for Jane - she won't have to sign in again, but may see a dialog asking to confirm that she's herself if she hasn't used the apps after some time on that Mac.

### I've enabled Cloud file syncing. This makes a large folder in each user home directory that fills up with all their cloud-saved stuff. What happens if I run some sort of process that deletes that folder, or its contents, when the user isn't logged in?

BAD THINGS HAPPEN! I tried this at the suggestion of a certain [Darren Wallace](https://dazwallace.wordpress.com/) and the whole cloud sync process just outright broke. I had to delete the user account and re-create it to get things working again. Luckily, the files were safe and sound in the cloud side (assets.adobe.org).

Don't do that.

### I delete entire user home directories/accounts on logout/overnight in my labs. What happens to the Adobe things?

So - if you log out, delete that user's account/home directory, then log back in _as the same user_ straight away, I noticed that cloud file sync gets upset (i.e. won't work, gives errors etc). There are some XML files cached in **/Library/Application Support/Adobe** and at least one of them has data referencing the user who activated SDL.

If you log straight back in with a _different_ user account, all is well. If you delete the user account/home, **restart**, then log back in as them, cloud file syncing is happy.

You could just disable cloud file syncing when you create your SDL package, but Adobe say:

> Certain Creative Cloud Desktop Applications require you to keep file syncing enabled.
> 
> Adobe.

They don't say _which_ apps though. I'm going to test that soon...

### I've disabled Cloud file syncing, which apps don't work?

- Premiere Rush CC - if you enable the **Sync with Creative Cloud** option, it complains:

![](/assets/2019/02/01/image.png)

### So there's no Apps panel in the CCDA that comes with my SDL package?

Nope, no option when you're building your SDL package in the Admin Console to have that, or the ability for standard users to install apps with elevated privileges. Ho hum.

But it looks like you might be able to edit this file, as outlined in [this post](https://macmule.com/2018/12/13/customising-the-adobe-creative-cloud-desktop-app/):

```
/Library/Application Support/Adobe/OOBE/Configs/ServiceConfig.xml
```

I haven't tried so I'd be interested to hear back if it works with SDL.

### Acknowledgements

Thanks to [Darren Wallace](https://dazwallace.wordpress.com/), [Patrick Fergus](https://foigus.wordpress.com/) and [Ben Toms](https://macmule.com/) for poking me with questions and test scenarios as well as their previous great work on the never-ending subject that is Adobe.
