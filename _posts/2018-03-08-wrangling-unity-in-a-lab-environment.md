---
layout: post
title: "Wrangling Unity in a lab environment"
date: 2018-03-08
coverImage: "unity1.png"
---

It's time for another one of these "how do you suppress all the stuff you don't want students to see and get the thing working the way you want the first time it runs" posts. In the red corner, it's me, Neil. And in the blue corner, it's Unity - a popular game development platform! Ding ding!

- Round 1: Get the packages.
- Round 2: Get the application licensed on your Macs.
- Round 3: Suppress automatic updates and make sure Unity doesn't sign in to your license holding account (because that's _really_ bad!!!).

<!--more-->

## Get the packages

That's easy enough - just grab the Download Assistant from [the website](https://store.unity.com/download?ref=personal). Run it, then when you hit the **Destination Select** part, click **Advanced** and you can specify the folder where the packages for the components you chose will go. In that folder you'll also get a handy script that can install them all at once, which may be useful. They have some documentation to support this too, [here](https://docs.unity3d.com/Manual/DeployingUnityOffline.html).

The packages don't contain any nasty surprises and deploy straightforwardly with management tools like Munki and Jamf. They'll also cleanly upgrade previous versions of Unity.

## Get the application licensed on your Macs

This gets a bit more interesting. Unity recently started offering free licenses for education, which is awesome for my institution. [The documentation](https://docs.unity3d.com/Manual/CommandLineArguments.html) reveals some promising command line arguments to license Unity programatically. Alas, the application crashes if you run it in batch mode without a user logged in, or not in their context (i.e. as root triggered by a Jamf policy). Here's a script I wrangled to firstly check for the license file, then run Unity as the logged in user if said file isn't there:

{% gist bde2353e9a6efc913476073103fbf19b %}

This is supposed to be run by a Jamf Policy (triggered at login). The parameters $4, $5 and $6 are your Unity serial number, license account username and password, respectively. You could easily wrangle this into [Outset](https://github.com/chilcote/outset) by getting rid of the **sudo -u "$loggedInUser"** part and putting the actual credentials in place of the parameters - but then make sure you delete that script after it's run as you don't want that stuff on disk in your labs!

## Suppress automatic updates and make sure Unity doesn't sign in to your license holding account

So if you open Unity after licensing it, you'll notice it's signed in as the account that holds your licenses! That means whoever's using Unity can get into your account and do naughty things like change your email/password etc. If anyone from Unity is reading this, please stop that behaviour, pretty please!

It also prompts when updates are available. We like keeping our labs up to date too, but we don't want to bug our users with that stuff, especially as they're not admins and can't update anyway. Sometimes, our teaching staff need to stick with a specific version as the changes updates bring can disrupt the courses they're halfway though teaching.

To get around this, we need to set some preferences _as the logged in user_. Unity does use the **com.unity3d.UnityEditor5.x** preference domain, but seems to ignore Configuration Profiles and anything in the machine-wide /Library/Preferences.

The keys you'd to set to manage updates away and stop Unity signing in with your license account are **EditorUpdateShowAtStartup (integer 0)** and **UnityConnectWorkOffline (integer 1)**. Here's the script I use, again, designed to be run by Jamf at user login:

{% gist 2bff33f30b95de01c6add07f60c69694 %}
