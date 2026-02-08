---
layout: post
title: "2015 MacBook Pro battery recall checker script"
date: 2019-06-21
coverImage: "mbfire.jpg"
---

[So this happened](https://support.apple.com/15-inch-macbook-pro-battery-recall). And [it's not good](https://appleinsider.com/articles/19/06/20/apple-issues-battery-recall-on-15-inch-macbook-pro-from-2015-through-2017). And now you might be worrying about how many MacBook Pros you manage that may be at risk.

Here's a script that might help. Feed it a text file containing the serial numbers of all the 2015 MacBook Pros in your fleet and remediate the ones that are "eligible". Data is output in CSV format, which you could redirect to a file.

How to use it (once you've downloaded and made it executable):

```
./mbpserialcheck.sh /path/to/inputfile.txt
```

To output directly to a CSV file:

```
./mbpserialcheck.sh /path/to/inputfile.txt > /path/to/outputfile.csv
```

If you can't see the script in this post (Wordpress has issues embedding from GitHub on some mobile platforms), here's a direct link for you: [https://gist.github.com/neilmartin83/9b6b2163edb71e2e6e578df54f0d599a](https://gist.github.com/neilmartin83/9b6b2163edb71e2e6e578df54f0d599a)

<figure>

https://gist.github.com/neilmartin83/9b6b2163edb71e2e6e578df54f0d599a

<figcaption>

  

</figcaption>

</figure>

Thanks to Nick Tong for the inspiration behind this. [He wrote a Jamf Extension Attribute](https://www.jamf.com/jamf-nation/discussions/32400/battery-recall-for-15-mid-2015-mbp#responseChild186454) you can use that'll tell you which MacBooks are eligible for recall the next time they submit inventory.
