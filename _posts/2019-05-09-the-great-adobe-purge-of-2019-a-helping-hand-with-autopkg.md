---
layout: post
title: "The Great Adobe Purge of 2019 - a helping hand with Autopkg"
date: 2019-05-09
coverImage: "adobesdl.jpg"
---

### Before we begin...

Read this: [https://dazwallace.wordpress.com/2019/05/08/the-great-adobe-purge-of-19/](https://dazwallace.wordpress.com/2019/05/08/the-great-adobe-purge-of-19/)

Then read that: [https://helpx.adobe.com/enterprise/kb/remove-unauthorized-versions.html](https://helpx.adobe.com/enterprise/kb/remove-unauthorized-versions.html)

Then realise that the macOS "package" Adobe provides is actually a zip file that contains this lot:

![](/assets/2019/05/09/image.png)

...which means admins need to do work to deliver and run this on our endpoints. :-(

Here are a couple of Autopkg recipes I put together to try and make things a bit easier:

<!--more-->

###### AdobeUninstallUnauthorizedVersions.download

Downloads [http://download.adobe.com/pub/adobe/creativecloud/UninstallUnauthorizedVersions\_mac.zip](http://download.adobe.com/pub/adobe/creativecloud/UninstallUnauthorizedVersions_mac.zip) - probably not that useful by itself.

###### AdobeUninstallUnauthorizedVersions.pkg

Builds an installer package from that downloaded zip archive that does the following:

- Installs the contents of the downloaded zip archive to a temporary location (**/private/tmp**).
- Runs a postinstall script that calls the **AdobeCCUninstaller** binary to do the needful.
- Takes the output from **AdobeCCUninstaller** and shoves it into a date and timestamped log here:

```
/Library/Logs/Adobe/AdobeUninstallUnauthorizedVersions_yyyy-mm-dd-hhmmss.log
```

**AdobeCCUninstaller** also generates/appends to a bunch of extremely verbose logs in **/Library/Logs/Adobe/Installers** for each app that it attempts to uninstall. However, I found its actual output to be more useful for the following reasons:

- It starts with a list of all the apps it's going to try and uninstall.
- It ends with a summary of what it has and hasn't uninstalled.
- It's easy to read and understand, which makes it quick to identify failures for individual apps.

### To use my Autopkg recipes:

My Autopkg recipe repo can be found at [https://github.com/autopkg/neilmartin83-recipes/](https://github.com/autopkg/neilmartin83-recipes/) if you wish to inspect my recipes before running them (which I strongly recommend you do!).

- Download and install **Autopkg** from [http://autopkg.github.io/autopkg/](http://autopkg.github.io/autopkg/)
- In **Terminal**, add my recipe repo and run the recipe:

```
autopkg repo-add neilmartin83-recipes
autopkg run AdobeUninstallUnauthorizedVersions.pkg
```

- Grab the resultant .pkg file from your Autopkg cache directory. The default location would be **~/Library/AutoPkg/Cache/com.github.neilmartin83.pkg.AdobeUninstallUnauthorizedVersions**
- Profit.
