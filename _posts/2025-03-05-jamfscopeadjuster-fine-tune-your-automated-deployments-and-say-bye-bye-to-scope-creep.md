---
layout: post
title: "JamfScopeAdjuster - Fine-tune your automated deployments and say bye bye to scope creep!"
date: 2025-03-05
coverImage: "image-2.png"
---

I've been working with the awesome [Graham Pugh](https://grahamrpugh.com/) and digging into the wonderful [JamfUploader](https://github.com/grahampugh/jamf-upload) suite of AutoPkg processors as part of my recent work. If you like working with a Jamf Pro instance using API automations, then this is for you! The result so far are a couple of new processors I've contributed to the family:

- [JamfExtensionAttributePopupChoiceAdjuster](https://github.com/grahampugh/jamf-upload/blob/main/JamfUploaderProcessors/READMEs/JamfExtensionAttributePopupChoiceAdjuster.md) (add or remove choices in an existing popup-menu type extension attribute)

- [JamfMobileDeviceAppUploader](https://github.com/grahampugh/jamf-upload/blob/main/JamfUploaderProcessors/READMEs/JamfMobileDeviceAppUploader.md) (does what it says on the tin)

- [JamfScopeAdjuster](https://github.com/grahampugh/jamf-upload/blob/main/JamfUploaderProcessors/READMEs/JamfScopeAdjuster.md)

Here, we'll explore [JamfScopeAdjuster](https://github.com/grahampugh/jamf-upload/blob/main/JamfUploaderProcessors/READMEs/JamfScopeAdjuster.md). What it is, why you need it, and how you'd use it.

If you're new to JamfUpload, I invite you to check out [this presentation](https://grahamrpugh.com/2022/07/22/macadmins-presentation-jamfuploader.html).

<!--more-->

1. [What is JamfScopeAdjuster?](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#what-is-jamfscopeadjuster)
2. [Why do I need it?](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#why-do-i-need-it)
3. [Examples](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#examples)
    1. [Apply a scope adjustment to an existing object](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#apply-a-scope-adjustment-to-an-existing-object)
    2. [Apply more than one scope adjustment to an existing object](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#apply-more-than-one-scope-adjustment-to-an-existing-object)
    3. [Apply scope adjustments to a template XML file as part of a larger recipe](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#apply-scope-adjustments-to-a-template-xml-file-as-part-of-a-larger-recipe)
4. [Appendix: Supported objects and scopeable types](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#appendix-supported-objects-and-scopeable-types)
    1. [Object Types](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#object-types)
    2. [Scopeable Types](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#scopeable-types)
5. [Want more?](https://soundmacguy.wordpress.com/2025/03/05/jamfscopeadjuster-fine-tune-your-automated-deployments-and-say-bye-bye-to-scope-creep/#want-more)

## What is JamfScopeAdjuster?

Put simply, JamfScopeAdjuster is a custom AutoPkg processor. It tweaks the scope of Jamf Pro objects like policies, configuration profiles, apps and more. You can add or remove targets and exclusions without using the Jamf Pro web GUI. Instead, you can do it in an AutoPkg recipe.

It's available right now in the in both the "bleeding edge" and stable repositories for JamfUploader: [https://github.com/grahampugh/jamf-upload](https://github.com/grahampugh/jamf-upload) and [https://github.com/autopkg/grahampugh-recipes](https://github.com/autopkg/grahampugh-recipes). Choose wisely!

JamfScopeAdjuster supports the following scoping goodies:

- Add or remove a single target, limitation or exclusion in a single item's scope

- Support for directory service user groups, network segments, buildings, departments, computer groups and mobile device groups (scopeables)!

- If an item already has a scope set, this will not change it, except to add or remove what you specify

- Works on existing objects in a Jamf Pro instance (want to re-scope something you already deploy?)

- Can also be supplied a template XML file in the same way as other JamfUpload processors (want to adjust scope on something you're preparing to upload to Jamf Pro as part of a larger recipe workflow?)

- Supports chaining multiple JamfScopeAdjuster processors together for more complex scoping shenanigans

- By default, strips XML and only uploads the `<scope>` section of API objects. This creates less risk.

- By default, it raises a ProcessorError if the recipe stops when you try to add something already in scope of an object, or if you try to remove something that is not in the object's scope.

## Why do I need it?

If you already use AutoPkg and JamfUpload to work with items in your Jamf Pro instance(s), you already know the answer. But put simply, there are numerous advantages to using a tool like this in your automations:

- Avoid manual busywork that comes with logging into a Jamf Pro instance, editing something, saving it, ad nauseam...

- Ensure what you do is repeatable and reliable

- Do you work with more than one Jamf Pro instance? Apply scope adjustments at scale across as many instances as you have, with [multitenant-jamf-tools](https://github.com/grahampugh/multitenant-jamf-tools) and autopkg-run! Yes, you can re-scope that profile in 150 Jamf Pro instances with a single command...

- Do whatever your scripting imagination allows - e.g Loop through a CSV of user groups and exclude them from every policy, profile or app in your instance (if you like)

- Infrastructure as code is just cool

- Get time back for [more important things](https://mashable.com/article/websites-to-waste-time)

## Examples

### Apply a scope adjustment to an existing object

I.e: "I have a policy: 'My Awesome Policy'. I want to add an exclusion to it for a computer group: 'All Desktop Computers'."

Let's consider the _[input variables](https://github.com/grahampugh/jamf-upload/blob/main/JamfUploaderProcessors/READMEs/JamfScopeAdjuster.md)_ we need to feed JamfScopeAdjuster. They are case sensitive, as they often need to match their counterparts in the Jamf API. In the simple example above, we'll have:

- **object\_type** - `policy`

- **object\_name** - `My Awesome Policy`

- **scoping\_operation** - `add`

- **scoping\_type** - `exclusion`

- **scopeable\_type** - `computer_group`

- **scopeable\_name** - `All Desktop Computers`

An AutoPkg recipe to do this may look like this and use just 3 processors.

```yaml
Description: Add or remove a scopeable object as a target, limitation or exclusion to or from an API object.
Identifier: JamfScopeAdjusterSingleItem
MinimumVersion: "2.3"

Input: {}

Process:
  - Processor: com.github.grahampugh.jamf-upload.processors/JamfObjectReader
    Arguments:
      object_name: My Awesome Policy
      object_type: policy

  - Processor: com.github.grahampugh.jamf-upload.processors/JamfScopeAdjuster
    Arguments:
      scoping_operation: add
      scoping_type: exclusion
      scopeable_type: computer_group
      scopeable_name: All Desktop Computers

  - Processor: com.github.grahampugh.jamf-upload.processors/JamfObjectUploader
    Arguments:
      replace_object: "True"

```

1. **JamfObjectReader** gets the policy object named My Awesome Policy from your Jamf Pro instance

3. **JamdScopeAdjuster** adds an exclusion for the computer group All Desktop Computers

5. **JamfObjectUploader** uploads the modified scope to the original object in Jamf Pro

Your AutoPkg output will contain some useful information - check out the bold bits:

```
JamfObjectReader: Session token received
JamfObjectReader: Checking for existing 'My Awesome Policy' on https://myinstance.jamfcloud.com
com.github.grahampugh.jamf-upload.processors/JamfScopeAdjuster
JamfScopeAdjuster: Added exclusion: computer_group with name 'All Desktops'.
JamfScopeAdjuster: Wrote processed XML to file: /Users/neil.martin/Library/AutoPkg/Cache/JamfScopeAdjusterSingleItem/myinstance_policy_106_My Awesome Policy.xml
com.github.grahampugh.jamf-upload.processors/JamfObjectUploader
JamfObjectUploader: Checking for existing 'My Awesome Policy' on https://myinstance.jamfcloud.com
JamfObjectUploader: Existing token is valid
JamfObjectUploader: Checking for existing 'My Awesome Policy' on https://myinstance.jamfcloud.com
JamfObjectUploader: policy 'My Awesome Policy' already exists: ID 106
JamfObjectUploader: Replacing existing policy as replace_object is set to 'True'
JamfObjectUploader: Uploading policy...
JamfObjectUploader: policy 'My Awesome Policy' update successful
```

You will get a copy of the processed XML file in your recipe cache directory. This also serves as the `object_template` that is passed to further subsequent processors afterwards.

Before:

![](/assets/2025/03/05/image.png)

After:

![](/assets/2025/03/05/image-1.png)

### Apply more than one scope adjustment to an existing object

This is easy - just add more JamfScopeAdjuster processors to your recipe. In the example below, we will also add a user group limitation and remove a computer group target.

```yaml
...

Process:
  - Processor: com.github.grahampugh.jamf-upload.processors/JamfObjectReader
    Arguments:
      object_name: My Awesome Policy
      object_type: policy

  - Processor: com.github.grahampugh.jamf-upload.processors/JamfScopeAdjuster
    Arguments:
      scoping_operation: add
      scoping_type: exclusion
      scopeable_type: computer_group
      scopeable_name: All Desktop Computers
      strict_mode: "False"

  - Processor: com.github.grahampugh.jamf-upload.processors/JamfScopeAdjuster
    Arguments:
      scoping_operation: add
      scoping_type: limitation
      scopeable_type: user_group
      scopeable_name: Naughty People

  - Processor: com.github.grahampugh.jamf-upload.processors/JamfScopeAdjuster
    Arguments:
      scoping_operation: remove
      scoping_type: target
      scopeable_type: computer_group
      scopeable_name: All Graphics Computers

  - Processor: com.github.grahampugh.jamf-upload.processors/JamfObjectUploader
    Arguments:
      replace_object: "True"

...
```

Notice I've added `strict_mode: "False"` to the first instance of JamfScopeAdjuster. This ensures we don't fail if a scopeable in the object already exists when we try to add it. It also ensures we don't fail if the scopeable does not exist when we're trying to remove it. You only need that variable once, on the first instance of JamfScopeAdjuster. Per AutoPkg behaviour, it's carried forward to subsequent processors.

### Apply scope adjustments to a template XML file as part of a larger recipe

This is useful if you're running a recipe that creates a new object and uploads it to Jamf Pro. JamfUpload makes extensive use of [XML templates](https://github.com/grahampugh/jamf-upload/wiki/JamfUploader-Templates) for this purpose. You may need to apply some scope changes. Your template may not have the XML structure to accommodate these changes.

JamfScopeAdjuster offers flexibility here and will add the necessary modifications to a template on the fly. It does not overwrite the original. If your template does not have a complete `<scope>` section, JamfScopeAdjuster will add the tags we need. It will also do so if the section is missing altogether.

For example, here is how it could be inserted as part of a larger recipe:

```
...

  - Processor: com.github.grahampugh.jamf-upload.processors/JamfScopeAdjuster
    Arguments:
      object_template: my_awesome_template.xml
      scoping_operation: add
      scoping_type: target
      scopable_type: computer_group
      scopeable_name: All Desktop Computers
      strict_mode: "False"
      strip_raw_xml: "False"

...
```

- **object\_template** - set to the path of the template XML file you want JamfScopeAdjuster to modify.

- **strip\_raw\_xml** - set to "False" because we want JamfScopeAdjuster to pass on the modified XML template with all other sections in-tact. By default, it will delete everything except `<scope>`. We don't want that for templates as we need to keep them complete!

**NOTE!** The **object\_template** variable's value changes to the path of the modified XML file. JamfScopeAdjuster creates this file, and it lives in the recipe cache directory. Any subsequent processors expecting **object\_template** will process the file at this new path and not the original one. This is an intentional part of how JamfScopeAdjuster is designed to work. It helps avoid excessive input variable usage with other JamfUpload processors. However, you should be aware!

As with existing objects, you can chain multiple JamfScopeAdjusters in succession when working on templates too.

## Appendix: Supported objects and scopeable types

### Object Types

| **Name** | **Value** |
| --- | --- |
| Mobile Device Configuration Profile | `configuration_profile` |
| Mac App Store App | `mac_application` |
| Mobile Device App | `mobile_device_application` |
| Computer Configuration Profile | `os_x_configuration_profile` |
| Policy | `policy` |
| Restricted Software | `restricted_software` |

### Scopeable Types

| **Name** | **Value** | **Scoping Type** |
| --- | --- | --- |
| Computer Group | `computer_group` | target, exclusion |
| Mobile Device Group | `mobile_device_group` | target, exclusion |
| Directory Services User Group | `user_group` | limitation, exclusion |
| Network Segment | `network_segment` | limitation, exclusion |
| Building | `building` | target, exclusion |
| Department | `department` | target, exclusion |

## Want more?

Then join us in the [MacAdmins Slack](https://www.macadmins.org/) in **[#jamf-uploader](https://macadmins.slack.com/archives/C01AVR04ES1)**
