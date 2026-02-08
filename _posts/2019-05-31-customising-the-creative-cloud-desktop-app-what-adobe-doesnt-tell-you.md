---
layout: post
title: "Customising the Creative Cloud Desktop App - what Adobe doesn't tell you"
date: 2019-05-31
coverImage: "adobesdl.jpg"
---

```
ServiceConfig.xml
```

That file ^^. If you've ever deployed the Adobe Creative Cloud Desktop App (CCDA) or any Adobe application that uses it, you might have come across this little nugget. On macOS it lives in **/Library/Application Support/Adobe/OOBE/Configs/** and on Windows it's in **C:\\Program Files (x86)\\Common Files\\Adobe\\OOBE\\Configs\\**. This is especially interesting if you wanted to have a bit more control over what's shown or hidden post-deployment (because sometimes we change our minds).

[Ben Toms goes into the gory details here](https://macmule.com/2018/12/13/customising-the-adobe-creative-cloud-desktop-app/). So do read that if this is something you're encountering for the first time.

You'll find there's more to this once you scratch beneath the surface...

<!--more-->

Fast forward to 2019 with the advent of Shared Device Licensing, and we have this option when we're creating a package in the Admin Console:

![](/images/image-5.png)

If we leave it ticked to disable file syncing (which you might want to do for computers that shared by many people, to help performance), users are greeted with this in the **Files** panel of the CCDA:

![](/images/image-6.png)

### But I want to hide the thing...

Telling folks they don't have access to something because IT said so is a surefire way to annoy them and generate lots of questions. Wouldn't it be better to just remove that panel altogether? Ignorance is bliss, right? It turns out, through some trial and error, that you can. Let's have a look at **ServiceConfig.xml** again, focusing on the **FilesPanel** key:

![](/images/image-8.png)

Notice that **masked** is set to **true**. This _disables_ the panel's functionality but leaves it visible. Now look at the **AppsPanel** key and notice it has a **visible** key, that's set to **false**. Also notice that our CCDA screenshot above is missing its **Apps** panel (which is good for labs!)... A clue. So what if we were to add a **visible** key under **FilesPanel**, like so?

![](/images/image-9.png)

Here's the result when the CCDA is next launched:

![](/images/image-10.png)

Voila, no more **Files** panel!

### But, but, I want to hide the other things...

That's not all! We can do the same with the **Fonts, Stock and Behance** panels too by adding some more to our **ServiceConfig.xml**. That might be useful in situations where your users don't have access to those services; users with only the "Spark + 2GB storage" product (which is available for all staff and students for free in education) won't have **Fonts** or **Stock**. If you [disable Public Link Sharing in your Admin Console](https://helpx.adobe.com/uk/enterprise/using/asset-settings.html), users can't do much with **Behance** either. The CCDA is taunting them, and that's just not cricket.

The tags/keys to add are undocumented as far as I can tell. I found them by sheer luck/trial and error. They are:

- **StockPanel**
- **BehancePanel**
- **FontsPanel**

So here's a **ServiceConfig.xml** example that will make them all go away, except **Learn** (because education):

https://gist.github.com/neilmartin83/ae8f6b6b94af9f335a61f537a2564b79

And the result... lovely and clean:

![](/images/image-11.png)

### Beware, there be dragons...

As with most pleasurable things, there is often an element of pain. This is no exception. Hiding panels effectively stops the CCDA from working/syncing things from their respective services. If your users have access to any services that you hide panels for, things might break when they're using apps which leverage that functionality. I tested this with **Fonts** specifically. With the panel hidden, in-app syncing/downloading of cloud fonts did not work when the user signed in had that service enabled. So, don't hide panels for services your users have access to.
