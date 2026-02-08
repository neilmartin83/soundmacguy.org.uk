---
layout: post
title: "Infrastructure as Code @ Jamf - Highlights and resources from JNUC 2025"
date: 2025-10-11
tags: 
  - "ai"
  - "cloud"
  - "devops"
  - "technology"
  - "terraform"
coverImage: "tff.png"
---

This year's JNUC saw a lot of excitement around IaC and its place in managing and orchestrating the Jamf Platform. This was my first time attending and presenting at JNUC and I'm still buzzing from the experience... Here are some of the takeaways!

<!--more-->

## The Jamf Platform Terraform Provider

With the announcement of the Jamf Platform APIs, Jamf published its very own Terraform provider: [terraform-provider-jamfplatform](https://github.com/Jamf-Concepts/terraform-provider-jamfplatform). I'm proud to have played a part in building it, and even prouder to watch it garner a spot in the opening keynote:

https://www.youtube.com/watch?v=ij4kLJsGAe8&t=3001s

Jamf also reinforced its commitment to continue contributions to, and endorsement of the fantastic and well-established community [Jamf Pro provider](https://github.com/deploymenttheory/terraform-provider-jamfpro) from friends at Deployment Theory/Lloyds Banking Group. Use both providers together to leverage the full suite of API capabilities that are available:

- Use **terraform-provider-jamfpro** for comprehensive management of objects in a Jamf Pro instance backed by the Classic and Jamf Pro APIs. Policies, profiles, groups, apps, everything including the kitchen sink!

- Use **terraform-provider-jamfplatform** to manage microservices backed by the Platform API; resources for Blueprints and Compliance Benchmarks and plus data sources for Unified Inventory.

- More! Use [**terraform-provider-jsctfprovider**](https://github.com/Jamf-Concepts/terraform-provider-jsctfprovider) to manage resources in Jamf Security Cloud!

I've put together [a GitHub repository](https://github.com/neilmartin83/terraform-jamfplatform-examples) with a few practical examples so you can get stuck in right now. It shows how the Platform and Pro providers work together. I hope it provides a little inspiration for your projects.

## Session: Automating Apple Endpoints Management. Git, CI/CD and Terraform for an Efficient Jamf Pro Administration

It was a great privilege to share the stage with [Tristan Valente](https://www.linkedin.com/in/tristan-valente-49628990) from Jamf partner [Netopie](https://www.linkedin.com/in/tristan-valente-49628990). Together, we explored Netopie's journey with GitOps/IaC and Tristan shared their experiences and best-practices. This was followed with a comprehensive introduction into Terraform, highlighting specific configuration examples using the Jamf Pro provider.

![](/images/img_5435.png)

![](/images/img_0055.png)

![](/images/image-from-ios.png)

The session video is [available here](https://reg.jnuc.jamf.com/flow/jamf/jnuc2025/home25/page/sessioncatalogphase2/session/1744962492146001SJxI) for attendees to view and I'll update this post when it's made public.

Grab the slides and presenter notes below:

[Download](https://soundmacguy.wordpress.com/wp-content/uploads/2025/10/final-1221-automating-apple-endpoints-management-git-cicd-and-terraform-for-an-efficient-jamf-pro-administration.pdf)

## Session: Infrastructure as Code with Jamf

[This session](https://reg.jnuc.jamf.com/flow/jamf/jnuc2025/home25/page/sessioncatalogphase2/session/1743101994075001oK2x) was delivered by colleagues [Ryan Legg](https://www.linkedin.com/in/rlegg) and [Shane Brown](https://www.linkedin.com/in/sbrown01) in the Sales Engineering team, alongside [Javier Robles](https://www.linkedin.com/in/roblesjavier/) and [Anthony Telljohann](https://www.linkedin.com/in/anthony-telljohann) from Jamf partner [Vanguard](https://www.vanguard.com).

It explored Vanguard's journey and how they leverage IaC to bring together powerful, repeatable automations in a Jamf environment. Attendees learned how these tools saved time, reduced errors, and delivered consistency across their organisation's management and security operations.

The SE team provides [these Terraform modules](https://github.com/Jamf-Concepts/terraform-jamf-platform) for inspiration on starting your own Terraform projects that manage your Jamf infrastructure.

![](/images/img_0048.jpg)

![](/images/img_0050.png)

## Session: Automating Jamf Security Platform Configuration

[This session](https://reg.jnuc.jamf.com/flow/jamf/jnuc2025/home25/page/sessioncatalogphase2/session/1744817837989001fqg3), delivered by colleagues [Dan Cuddeford](https://www.linkedin.com/in/cuddeford) and [Ryan Legg](https://www.linkedin.com/in/rlegg) examined various approaches organisations could take toward managing an API-based SaaS tool, with a focus on automating the configuration of Jamf Security platforms.

It centered on Terraform as the chosen Infrastructure as Code (IaC) platform for the team’s implementation and explored the process of building a Terraform provider to support those platforms.

The [Jamf Security Cloud provider](https://github.com/Jamf-Concepts/terraform-provider-jsctfprovider) is the result!

## Community Resources

A vibrant community is growing around IaC with Jamf - get involved!

- Join [#terraform-provider-jamfplatform](https://macadmins.slack.com/archives/C09HP9V1K5H) in the MacAdmins Slack to discuss the new Jamf Platform provider and related workflows

- Join [#terraform-provider-jamfpro](https://macadmins.slack.com/archives/C06R172PUV6) to discuss the well-established community Jamf Pro provider and related workflows

Thank you to everyone who has played a part in this journey so far - all the open source maintainers, session attendees and colleagues. This looks to become something quite special and I can't wait to see what happens next. ❤️
