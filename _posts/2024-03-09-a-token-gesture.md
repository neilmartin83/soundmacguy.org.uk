---
layout: post
title: "A Token Gesture"
date: 2024-03-09
coverImage: "get-started-what-is-a-token.png"
---

If you work with the Jamf Pro classic API, be aware that [Basic Authentication is deprecated and will go away](https://learn.jamf.com/en-US/bundle/jamf-pro-release-notes-current/page/Deprecations_and_Removals.html). **Specifically this month, so it's time to get your house in order!**

> Basic authentication in the Classic API is no longer enabled by default for new Jamf Pro instances for enhanced security. Support for Basic authentication will be removed in **March 2024**. Jamf will provide additional information at a later date. To disable Basic authentication before support is removed, navigate to Settings > User accounts and groups, click Password Policy, and then deselect the Allow Basic authentication in addition to Bearer Token authentication checkbox.

The removal of Basic Authentication is great in terms of security and now you've got a couple of choices for how you authenticate with the API... but the main thing to consider is that both methods involve two steps:

1. Authenticating with a username/password (or similar...) credential to obtain a Bearer Token that is valid for a limited period of time and can be refreshed/replaced.

3. Using the Bearer Token and its replacements for all subsequent interactions with the API from your script/application etc.

<!--more-->

## API Roles and Clients

Introduced in August 2023 with Jamf Pro 10.4, think of these as being similar to Jamf Pro Users (API Clients) and Jamf Pro Groups that define permissions (API Roles). The difference is that you can't log into the Jamf Pro web interface with API Clients. This is the recommended way for your scripts/apps/solutions to interact with the Jamf Pro API going forward.

[Read more here about how to set them up, and what the authentication process will look like inside your scripts/applications.](https://learn.jamf.com/en-US/bundle/jamf-pro-documentation-10.50.0/page/API_Roles_and_Clients.html)

## Obtaining Bearer Tokens with a Jamf Pro User

Most solutions out there are likely using traditional Jamf Pro user accounts that have a specific set of permissions defined for the endpoints they are allowed to interact with and what they can do to them. As well as access those endpoints directly, you can also log into the Jamf Pro web interface as well using their credentials, like any other Jamf Pro user account.

If your scripts/applications/solutions using the API perform basic (username/password) authentication for each request, update them to get and use a Bearer Token.

[The awesome Jamf Developer Documentation has a great example and explanation of how to achieve this here:](https://developer.jamf.com/jamf-pro/docs/jamf-pro-api-overview#bearer-tokens)

You can test that code snippet yourself to make sure it works in your environment.

## Resurrections

As for me, I have a couple of old projects that a few people may still be using (thank you!). Actually, the the main reason for this blog post is because someone reached out to me about this issue in relation to those projects!

They both provide different onboarding/provisioning workflows focused on education environments. I've lifted up those rocks and added support for Bearer Token authentication for them - so if you're relying on either of these workflows, check out the changes I've made!

[Jamf Nation-Roadshow London 2018](https://github.com/neilmartin83/Jamf-Nation-Roadshow-London-2018)

[MacADUK-2019](https://github.com/neilmartin83/MacADUK-2019)

Bear in mind that both workflows use deprecated and unsupported products (DEPNotify and NoMAD Login) so I'd recommend migrating to an alternative solution where possible! It's unlikely I'll make any changes to them again myself.
