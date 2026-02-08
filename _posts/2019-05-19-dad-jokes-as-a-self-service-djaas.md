---
layout: post
title: "Dad Jokes as a (Self) Service (DJaaS?)"
date: 2019-05-19
coverImage: "image-4.png"
---

There's a new craze sweeping the community. "Dad jokes". The special kind of joke that makes you roll your eyes and cringe because it's so bad, yet at the same time you feel a great wave of embarrassment because you find it funny.

So why am I blogging about this? I have kids, so any new way to annoy them will naturally peak my interest. But what have dad jokes got to do with systems administration? Let's see...

<!--more-->

### How it all began

It started in a couple of places. A couple of weeks ago, the wonderful Tim Perfitt of [Twocanoes](https://twocanoes.com/) put together an app ([get it here](https://testflight.apple.com/join/XyzsCR12)) that lets you send a random joke to a unlucky recipient in Messages.

https://twitter.com/tperfitt/status/1124770444652032000

Meanwhile, on the [MacAdmins Slack](https://macadmins.herokuapp.com)...

![](/images/image-2.png)

Which led to copious use of the new **/dadjoke** command, especially in the #london channel.

### I can haz an API

The source of all this fun can be traced back to the website [https://icanhazdadjoke.com](https://icanhazdadjoke.com/). Head there and you'll get a webpage with a dad joke. So what? Dig a little deeper and you'll see it has a really easy to use API! Here's the documentation: [https://icanhazdadjoke.com/api](https://icanhazdadjoke.com/api). Let the fun begin...

Let's open a Terminal and start simple:

```
$ curl https://icanhazdadjoke.com
 What did the hat say to the scarf?
 You can hang around. I'll just go on ahead.
```

So using **curl** and pointing it at the endpoint with no arguments will GET us a plain text response with a random dad joke. As the API's documentation says though, it's considerate to specify your own custom **User-Agent**. This tells the endpoint who's asking for a joke and gives the creators a better understanding of usage. Use the **\-H** argument to include an extra header (in fact two) in the request:

```
$ curl -H "User-Agent: Sound Mac Guy (https://soundmacguy.wordpress.com)" -H "Accept: text/plain" https://icanhazdadjoke.com/
```

The first is the **User-Agent** (which can be your name and website, for example), and the second is telling the endpoint that we want its reply in plain text.

What can we do with this? Well, lots of things. If we look at the good old Bash scripting language, we can use [command substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html) to leverage the power of dad jokes. For example:

```
#!/bin/bash

dadJoke=$(/usr/bin/curl -H "User-Agent: Sound Mac Guy (https://soundmacguy.wordpress.com)" -H "Accept: text/plain" https://icanhazdadjoke.com/)

/bin/echo "$dadJoke"
```

This will create a variable called **dadJoke** and populate it with the output of the command we put between **$( )**, then return the contents of the **dadJoke** variable to [Standard Output](https://www.computerhope.com/jargon/s/stdout.htm).

### Having fun

Did you know macOS has a built-in speech synthesiser, namely the **say** command? Try this:

```
$ say $(curl -H "User-Agent: Sound Mac Guy (https://soundmacguy.wordpress.com)" -H "Accept: text/plain" https://icanhazdadjoke.com/)
```

Great, eh? I hope you're not tempted to **ssh** into any Macs you might manage and run that one remotely...

Your users don't have to miss out either. As Mac Admins, we've got tools to nag, prompt and notify them. Here's an example using AppleScript:

```
set theDialogText to (do shell script "curl https://icanhazdadjoke.com/")
 display dialog theDialogText
```

![](/images/image-3.png)

For those amongst us using Jamf, we've got a binary called jamfHelper. [Learn more about that, here](https://apple.lib.utah.edu/jamfhelper/). Here's an example of how you might use jamfHelper in a script:

```
#!/bin/bash

/Library/Application\ Support/JAMF/bin/jamfHelper.app/Contents/MacOS/jamfHelper \
     -windowType utility \
     -icon "/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/UserIcon.icns" \
     -title "Dad joke" \
     -heading "Dad says…" \
     -description "$(curl -H "User-Agent: Sound Mac Guy (https://soundmacguy.wordpress.com)" -H "Accept: text/plain" https://icanhazdadjoke.com/)" \
     -button1 "Oh, Dad…"
```

![](/images/image-4.png)

Why not put that one in a Self Service policy for your user's viewing pleasure?

So, what else might you do other than annoy your colleagues, users, friends and family? I'm using this to help me learn Python; specifically parsing JSON (the API can return dad jokes in this format too!). Or how about something more interesting like signalling successful completion of a CI or build process? Integrating it with a tool like [If This Then That](https://ifttt.com/)? Add a menu option to [NoMAD](https://nomad.menu/)/[Jamf Connect](https://www.jamf.com/products/jamf-connect/)? Put it in [Hello-IT](https://github.com/ygini/Hello-IT) or [Yo](https://github.com/sheagcraig/yo)? [BitBar](https://getbitbar.com/) looks interesting too... The possibilities are endless.

I'm sure many of you in the community have some great ideas - let's hear them in the comments!

And thank you to Tim Perfitt for coining the term DJaaS on Twitter - that's pure genius.
