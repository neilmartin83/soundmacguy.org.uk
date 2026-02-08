---
layout: post
title: "Adobe device licensing and expiry dates and Jamf - Oh my!"
date: 2018-11-30
coverImage: "adobe-logo1.png"
---

### It's all Ben's fault.

![Screen Shot 2018-11-30 at 05.47.04](/images/screen-shot-2018-11-30-at-05-47-04.png)

### The problem:

Adobe device license serial numbers have an expiry date. Adobe have the **[AdobeExpiryCheck](https://helpx.adobe.com/enterprise/kb/volume-license-expiration-check.html)** command line tool you can run that'll tell you if you have a device license serial number and what the expiry date is. This tool returns results that look like this:

![Screen Shot 2018-11-30 at 05.53.15](/images/screen-shot-2018-11-30-at-05-53-151.png)

That's a huge load of text and a date that's in a format we might want to change if we were to say, want to make it useful for our management tool of choice. Oh, and the AdobeExpiryCheck tool comes as a zip file download, so isn't really mass-deployment friendly, if we want to get it on our entire fleet of computers.

Down the rabbit hole we go.<!--more-->

### Packaging the tool - Autopkg!

With some great help from Eric Holtam (@eholtam on the Slack) with some regex and tips on using the **URLTextSearcher** processor, I created a download and pkg set of recipes for [Autopkg](https://github.com/autopkg/autopkg) that'll package **AdobeExpiryCheck** and install it in /usr/local/bin (that's in the PATH so you can just type the name of the command itself to run it in Terminal).

If you're not using [Autopkg](https://github.com/autopkg/autopkg) already, why on earth not? It's awesome. The [Getting Started](https://github.com/autopkg/autopkg/wiki/Getting-Started) page will get you going.

Add my recipe repository:

```
autopkg repo-add neilmartin83-recipes
```

Make the package:

```
autopkg run AdobeExpiryCheck.pkg
```

Profit!

### Extension Attributes!

You can do a some good things with Jamf Extension Attributes that spit out data as a date. Such as create Advanced Searches to generate reports on your entire fleet. Or Smart Groups whose criteria could be "more than 0 days ago" - this example would populate computers as their licenses expire. However, this doesn't let you do things like show how many devices have serial numbers that'll expire within a certain number of days (to give us advanced warning), but we have another solution for that...

I've created a few Extension Attributes that should help. Grab them via the links and upload them to your Jamf Pro instance!

**[Adobe Creative Cloud - Device License Present](https://github.com/neilmartin83/Jamf-Pro-Extension-Attributes/blob/master/Adobe%20Creative%20Cloud%20-%20Device%20License%20Present.xml)** - determines whether **AdobeExpiryCheck** exists and can find an active Device License serial number. Reports "Yes" if it does, "No" if it doesn't, or "Unable to check" if it can't find **AdobeExpiryCheck**

**[Adobe Creative Cloud - Device License Expiry Date](https://github.com/neilmartin83/Jamf-Pro-Extension-Attributes/blob/master/Adobe%20Creative%20Cloud%20-%20Device%20License%20Expiry%20Date.xml)** \- parses the results from **AdobeExpiryCheck** and returns the expiry date in a proper YYYY-MM-DD date format.

**[Adobe Creative Cloud - Device License Remaining Days](https://github.com/neilmartin83/Jamf-Pro-Extension-Attributes/blob/master/Adobe%20Creative%20Cloud%20-%20Device%20License%20Remaining%20Days.xml)** \- calculates the number of remaining days until expiry and returns the value as an integer.

So, with all that in place, you could use the **Device License Expiry Date** Extension Attribute to generate reports with Advanced Searches (just tick it under the **Display** tab).

The **Device License Remaining Days** Extension Attribute could be used in conjunction with the **Device License Present** to create Advanced Searches or Smart Groups that populate if the expiry date is approaching soon, or lapsed. Then you could make things happen (like email you an alert or deploy a new license etc). For example:

**Computers with Adobe Device Licenses expiring within 60 days:**

![Screen Shot 2018-11-30 at 10.06.31](/images/screen-shot-2018-11-30-at-10-06-31.png)

Huge hat-tips to [Ben Toms](https://macmule.com/), [Patric Fergus](http://foigus.wordpress.com/), [Steve Wood](http://www.geekygordo.com/) and [Eric Holtam](https://osxbytes.wordpress.com/) for the inspiration and support.

Enjoy!
