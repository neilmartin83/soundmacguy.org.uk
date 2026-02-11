---
layout: post
title: "What's new with terraform-provider-axm (AppleCare, that's what)"
date: 2025-11-06
coverImage: "a3ed7df1-849d-44aa-ab45-f0d6963beb3c.png"
---

Hello Terraformers! Apple recently, and quietly bumped their [Apple School and Business Manager API](https://developer.apple.com/documentation/apple-school-and-business-manager-api) to 1.3+. Along came a [new endpoint](https://developer.apple.com/documentation/applebusinessmanagerapi/get-all-apple-care-coverage-for-an-orgdevice) for querying AppleCare coverage details for individual devices.

With that, [terraform-provider-axm](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs) has a shiny new data source; [axm\_organization\_device\_applecare\_coverage](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs/data-sources/organization_device_applecare_coverage). It gives you a list of every AppleCare agreement (resource) a device has (they can indeed have more than zero). Perhaps not a typical use case for Terraform, but maybe it'll come in handy. If you're working with Go, you can import the client and use the [GetOrgDeviceAppleCareCoverage](https://github.com/neilmartin83/terraform-provider-axm/blob/main/examples/client/GetOrgDeviceAppleCareCoverage/GetOrgDeviceAppleCareCoverage.go) function in your own projects.

<!--more-->

Back to Terraform; if you were to declare the data source and an output for it, like this:

```hcl
data "axm_organization_device_applecare_coverage" "example" {
  id = "FAKE123456"
}
output "applecare_coverage" {
  value = data.axm_organization_device_applecare_coverage.example
}
```

You'd see a result like that:

```
data.axm_organization_device_applecare_coverage.example: Reading...
data.axm_organization_device_applecare_coverage.example: Read complete after 1s [id=FAKE123456]
Changes to Outputs:
  + applecare_coverage = {
      + applecare_coverage_resources = [
          + {
              + agreement_number          = ""
              + contract_cancel_date_time = ""
              + description               = "Limited Warranty"
              + end_date_time             = "2020-10-20T00:00:00Z"
              + id                        = "FAKE123456"
              + is_canceled               = false
              + is_renewable              = false
              + payment_type              = "NONE"
              + start_date_time           = "2019-10-21T00:00:00Z"
              + status                    = "INACTIVE"
            },
        ]
      + id                           = "FAKE123456"
    }
```

This poor old device only has one AppleCare agreement that's long-expired... so no hope of a warranty for me! Ho hum.
