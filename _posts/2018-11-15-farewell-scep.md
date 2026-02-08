---
layout: post
title: "Farewell SCEP!"
date: 2018-11-15
coverImage: "scep-icon.png"
---

No, not the certificate thing. Microsoft System Center Endpoint Protection for macOS and Linux, [as announced at Microsoft's Blog](https://techcommunity.microsoft.com/t5/Configuration-Manager-Blog/End-of-Support-for-SCEP-for-Mac-and-SCEP-for-Linux-on-December/ba-p/286257). There were rumours of [its impending demise on Twitter](https://twitter.com/ccmexec/status/978994886996385794) back in March but nothing concrete from Microsoft until now.

So, no more support after the end of 2018 and definition updates may stop any time after that. There is the mention that you might be able to get ESET Endpoint Security for Mac if you contact your Microsoft TAM. The cost implications, or what it means to "qualify" are unclear.

As far as I know, ESET Endpoint Security for Mac uses a command line tool **esets\_set** which is functionally on par with **scep\_set**. That means my blog posts for managing SCEP ([Part 1](https://soundmacguy.wordpress.com/2017/09/18/managing-microsoft-system-center-endpoint-protection-scep-part-1/), [Part 2](https://soundmacguy.wordpress.com/2017/09/26/managing-microsoft-system-center-endpoint-protection-scep-part-2/), [Part 3](https://soundmacguy.wordpress.com/2017/11/19/managing-microsoft-system-center-endpoint-protection-scep-part-3/), and [More Reporting](https://soundmacguy.wordpress.com/2018/04/07/microsoft-system-center-endpoint-protection-scep-more-hidden-reporting-goodness/)) would probably be relevant for it.

Linux is left with no migration path - so if you manage SCEP on that platform, you're on your own.

Goodbye SCEP, it was nice knowing you!
