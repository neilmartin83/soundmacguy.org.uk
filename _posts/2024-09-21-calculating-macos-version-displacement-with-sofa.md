---
layout: post
title: "Calculating macOS Version Displacement with SOFA"
date: 2024-09-21
tags: 
  - "apple"
  - "mac"
  - "macos"
  - "security"
  - "technology"
coverImage: "custom_logo.png"
---

When we examine the major version of macOS that's installed on a specific computer, we tend to express how far behind, or in front it is from that which is now released from Apple. We use the term n-x. Where "n" means the current released version and "x" is the displacement.

At the time of writing, macOS 14 Sonoma is 1 version behind the current release of macOS 15 Sequoia. So it has a displacement of 1. Or we say "n-1".

One reason this is important is when we consider Apple's practices for patching security vulnerabilities. They historically release updates for the current and prior 2 major macOS releases, but publish the warning below in [https://support.apple.com/en-gb/guide/deployment/depc4c80847a/web](https://support.apple.com/en-gb/guide/deployment/depc4c80847a/web):

> Because of dependency on architecture and system changes to any current version of Apple operating systems (for example, macOS 14, iOS 17, and so on), not all known security issues are addressed in previous versions (for example, macOS 13, iOS 16 and so on).

So whilst you will get updates for macOS n-1 or n-2, not all vulnerabilities patched in n will be patched in the two earlier versions. We'd also expect version n-3 and earlier not to get any updates at all. Note this is observed behaviour and not officially documented by Apple.

With this in mind, the displacement becomes a moving target when a new major version of macOS is released. I've contributed a Jamf Pro Extension Attribute script to the awesome [SOFA project](https://sofa.macadmins.io). This script will calculate the displacement on any computer for you. It returns an integer so you can build Smart Computer Groups that find whatever you like. A result less than 0 means someone is running a beta version of macOS. A result greater than 2 means they're far behind and not getting security updates at all! It's inspired by, and based on the great work done by [Graham Pugh](https://grahamrpugh.com/2024/04/29/sofa-and-jamf-pro.html) and you can find it [here](https://github.com/macadmins/sofa/blob/main/tool-scripts/macOSVersionDisplacement-EA.sh).

Learn more about how the script works [here](https://github.com/macadmins/sofa/tree/main/tool-scripts#macosversiondisplacement-eash).

PSA: There are also significant efficiency improvements for how the SOFA JSON feed is cached locally with the current EA scripts in the repo. If you're using them, make sure you are running the current versions!
