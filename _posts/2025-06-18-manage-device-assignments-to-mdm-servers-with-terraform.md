---
layout: post
title: "Manage Device Assignments to MDM Services with Terraform"
date: 2025-06-18
coverImage: "image.png"
---

Just a quick one - the [latest release of the axm provider](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs) can now manage device assignments to MDM Servers (or Services as Apple calls them now)!

Add a resource block for `axm_device_management_service` and off you go.

[Read the docs for more details and enjoy!](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs/resources/device_management_service)

**Top tip:** to get the MDM Service ID you want, navigate to it via Apple Business/School Manager -> Preferences and it will be visible in the address bar at the end of the URL (e.g. `FAKE0000111122223333444444444444`)

It works, with an example like this enforcing a state of 3 devices being assigned to a chosen MDM Service:

```hcl
resource "axm_device_management_service" "example" {
  id = "FAKE0000111122223333444444444444"
  device_ids = [
    "FAKE000ABC123",
    "FAKE111DEF456",
    "FAKE222GHI789"
  ]
}
```

![](/images/image-1.png)

One example could be to use this in conjunction with the awesome [jamfpro](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs) provider. You can enforce device assignments to the MDM Service(s) your Jamf Pro instance is using in the Apple Business/School Manager end itself.

Use the [jamfpro\_device\_enrollments](https://registry.terraform.io/providers/deploymenttheory/jamfpro/latest/docs/data-sources/device_enrollments) data source to get the MDM Server ID (`server_uuid`) from Jamf Pro and pass it to `id` in this provider's `axm_device_management_service` resource block.

A word of warning - you might hit weird rate limit/token authentication issues if you run this very often (e.g. when testing). Just give it a few minutes and try again. This API is new and we're still learning a lot about it...

Voila!
