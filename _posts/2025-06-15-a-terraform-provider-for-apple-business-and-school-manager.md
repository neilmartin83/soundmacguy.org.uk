---
layout: post
title: "A Terraform Provider for Apple Business and School Manager"
date: 2025-06-15
tags: 
  - "automation"
  - "azure"
  - "microsoft"
  - "security"
  - "technology"
coverImage: "chatgpt-image-jun-15-2025-06_30_13-am.png"
---

**Edit 2025-06-21 - you can now [assign and unassign devices](https://soundmacguy.wordpress.com/2025/06/18/manage-device-assignments-to-mdm-servers-with-terraform/) to MDM Services with the provider!**

[I did a thing](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs). Say hi to this little pet project of mine. A Terraform provider with some data sources to grab stuff from the shiny new [Apple Business/School Manager API](https://developer.apple.com/documentation/apple-school-and-business-manager-api)!

<!--more-->

It's early days and there may will be dragons... But right now there are data sources for these things:

- Get a list of all the devices and their details

- Get details for a specific device by its ID (serial number)

- Get a list of all the device management services (MDM Servers)

- Get a list of device serial numbers assigned to a specific device management service

- Get details about the device management server assigned to a specific device, by the device ID (serial number)

Check out the [documentation](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs) to see more, and if you'd like to ridicule contribute, the [source code](https://github.com/neilmartin83/terraform-provider-axm)! If you’re a Go-pher, check out the client and client\_oauth files for functions you might want to use in your own projects to interact with this API.

As with any open source endeavor, I wholeheartedly welcome feedback, criticism, suggestions and pull requests (and I'd really appreciate a little help implementing device assignment/un-assignment as a resource - got a bit stuck with that and gave up for the moment).

I also don't have much of an imagination, so I'm keen to hear from you!

- How will you use it?

- What will you do with it?

- What problems will it create or solve in your organisation?

- What do you want to see it do in the future?

I'm so thankful to Apple for giving us this API. I'm excited to see Apple add more capabilities in the future (please!). My little wish-list so far:

- MDM Server creation (we post a public key and it spits back the token!)

- Volume Purchase Token creation

- Apps and Books purchasing/assignment/transferring between Locations

With awesome projects like the [jamfpro provider](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs), it feels like we're getting closer than ever to being able to manage all our Apple stuff with an infrastructure-as-code mindset. Exciting times indeed!

I for one will relish the day when I don't need to log into AxM every year to renew a token...️

Thank you, fruit company! ❤
