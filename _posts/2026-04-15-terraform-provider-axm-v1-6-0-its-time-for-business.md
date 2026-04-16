---
layout: post
title: "terraform-provider-axm v1.6.0 - It's time for Business"
date: 2026-04-15
---

Hello Terraformers! It's been a while since the last update on [terraform-provider-axm](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs). Version 1.6.0 is a bit chunkier than usual and it brings a few new resources and data sources, thanks to Apple releasing the [Business API](https://developer.apple.com/documentation/apple-school-and-business-manager-api/apple-school-manager-and-apple-business-api-changelog).

Let's peruse!

## Blueprints

You can now create, update and delete Blueprints directly from Terraform. If you've been managing these by hand in the Apple Business web UI, those days (and it's only been 2 since Apple released Business... hehe) are over.

A Blueprint ties together apps, configurations, packages, devices, users and user groups. The `axm_blueprint` [resource](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs/resources/blueprint) lets you declare all of that in one block:

```hcl
resource "axm_blueprint" "onboarding" {
  name        = "New Hire Onboarding"
  description = "Standard setup for new starters"

  app_ids = [
    "app-id-1",
    "app-id-2",
  ]

  configuration_ids = [
    "config-id-1",
  ]

  device_ids = [
    "SERIALNUM1",
    "SERIALNUM2",
  ]
}
```

All six relationship types are supported - `app_ids`, `configuration_ids`, `package_ids`, `device_ids`, `user_ids` and `user_group_ids`. Terraform will diff your desired state against what's in Apple Business and add or remove relationships accordingly. Drift detection works too, so if someone sneaks in a change via the UI, your next `plan` will see it.

There are also data sources for looking stuff up:

- **`axm_blueprint`** - fetch a single Blueprint by ID, including all its relationships
- **`axm_blueprints`** - list every Blueprint in your organisation

And of course, the `list` resource is there for those of us who partake in [Query](https://developer.hashicorp.com/terraform/language/query), with filters for `name`, `name_contains` and `status`.

## Configurations

The **`axm_configuration`** [resource](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs/resources/configuration) lets you manage Configuration Profiles; specifically the `CUSTOM_SETTING` type, which is the one you can create via the API.

```hcl
resource "axm_configuration" "wifi" {
  name                     = "Corporate Wi-Fi"
  configured_for_platforms = ["PLATFORM_IOS", "PLATFORM_MACOS"]
  configuration_profile    = file("wifi.mobileconfig")
}
```

For all other configuration types (the ones Apple manages), there are read-only data sources:

- **`axm_configuration`** - fetch a single configuration by ID (includes the profile payload for `CUSTOM_SETTING` types)
- **`axm_configurations`** - list all configurations

The `list` resource supports filtering too - `name`, `name_contains` and `configuration_type`.

## New data sources

Beyond Blueprints and Configurations, we also have data sources:

| Data Source | Description |
|---|---|
| `axm_app` / `axm_apps` | Look up apps (VPP/Apps and Books) |
| `axm_package` / `axm_packages` | Look up packages |
| `axm_user` / `axm_users` | Look up managed Apple Account users |
| `axm_user_group` / `axm_user_groups` | Look up user groups |
| `axm_audit_events` | Query the audit log with date range filters |

The singular variants fetch by ID; the plural ones grab the lot. Having all this data accessible in Terraform opens up some interesting possibilities - cross-referencing users with devices, auditing changes, building reports, that kind of endeavour.

## Go forth and grab it

```hcl
terraform {
  required_providers {
    axm = {
      source  = "neilmartin83/axm"
      version = "~> 1.6.0"
    }
  }
}
```

Full docs are on the [Terraform Registry](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs) and source is on [GitHub](https://github.com/neilmartin83/terraform-provider-axm). This is really just a little hobby that satisfies my interest in working with APIs. It scratches an itch, but perhaps someone out there will find it interesting or even useful. As always, feedback, berating, issues and PRs are welcome!

I really hope the glorious fruit company keeps expanding this API - I've still got my fingers crossed for MDM server token creation and VPP token management. One day... 🤞
