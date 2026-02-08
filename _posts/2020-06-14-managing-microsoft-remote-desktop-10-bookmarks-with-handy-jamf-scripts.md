---
layout: post
title: "Managing Microsoft Remote Desktop 10 Bookmarks with handy Jamf Scripts"
date: 2020-06-14
coverImage: "msrdscript.png"
---

Microsoft introduced a handy feature in version 10 of their Remote Desktop client with little fanfare; the ability to manage bookmarks, feeds and gateways using the command line. I think it deserves some more attention.

<!--more-->

It was possible to add bookmarks with the previous version 8 of Microsoft Remote Desktop using [this method devised by Ben Toms](https://macmule.com/2013/10/22/how-to-create-a-microsoft-remote-desktop-8-connection/), but sadly, version 10 no longer supports this way of doing it.

## Introducing the `--script` argument

Be aware that these commands must run in the current logged in user's context and will manage Microsoft Remote Desktop entries only for them. There's isn't an option to set these up for all users on a Mac at once.

Open a Terminal and run:

`"/Applications/Microsoft Remote Desktop/Contents/MacOS/Microsoft Remote Desktop" --script`

Behold the output:

```
Usage:

  --script  

  Modules:

    bookmark  Create, edit or delete a connection bookmark.
    feed      Subscribe to a resource feed, or edit or delete a subscription.
    gateway   Create, edit or delete a Remote Desktop gateway.

  To get help for a specific module:

    --script  help

  Examples:

    --script bookmark help
    --script feed help
    --script gateway help
```

So, if we want to manage a bookmark, say create one, we can use the `help` verb get more information about the commands and syntax we'd need to achieve this. For example, to see the commands available for bookmarks, run:

`"/Applications/Microsoft Remote Desktop/Contents/MacOS/Microsoft Remote Desktop" --script bookmark help`

```
Create, edit or delete a connection bookmark.

Usage:

  --script bookmark <command> <unique ID> <parameters>

  Commands:

    write   Create or edit a connection bookmark.
    delete  Delete a bookmark.
    list    List all stored bookmarks.
    export  Output a bookmark as a formatted string.

  To get help for a specific command:

    --script bookmark <command> help

  Examples:

    --script bookmark write help
    --script bookmark delete help
    --script bookmark list help
    --script bookmark export help
```

Then, to drill down further and see more information about creating (writing) a bookmark, run:

`"/Applications/Microsoft Remote Desktop/Contents/MacOS/Microsoft Remote Desktop" --script bookmark write help`

```
Create or edit a connection bookmark identified by a unique ID.

Usage:

  --script bookmark write <unique ID> <parameters>

  Parameters:

    --hostname <string>
      The hostname of the remote server. This value is required when creating
      the bookmark.

    --username <string>
      The username to use when connecting to the remote server. A user account
      will be created if an existing one cannot be found.

    --password <string>
      The password associated with the username.

    --friendlyname <string>
      The friendly name of the bookmark.

    --group <string>
      The name of the group wherein the bookmark should reside. A group will
      be created if an existing one could not be found.

    --gateway <string>
      The unique ID of an existing Remote Desktop gateway. If a gateway with
      the given ID does not exist, this parameter will be ignored.

    --gatewayhostname <string>
      The hostname of an existing Remote Desktop gateway. If a gateway with
      the given hostname does not exist, a new one will be created with the
      given hostname. If multiple gateways exist with the same hostname, the
      first one will be selected.

    --bypassgateway <true | false>
      If set to true, the Remote Desktop gateway will not be used if the Remote
      Desktop server is on the same network as the client machine. This
      parameter is true by default.

    --admin <true | false>
      If set to true, the client will connect to a session that is used for
      administrative purposes.

    --swapmousebuttons <true | false>
      If set to true, the left and right mouse buttons will be swapped.

    --autoreconnect <true | false>
      If set to true, the client will attempt to automatically reconnect if
      the connection is interrupted. This parameter is true by default.

    --useallmonitors <true | false>
      If set to true, all the local monitors will be used in the remote
      session.

    --fullscreen <true | false>
      If set to true, starts the connection in full screen mode (vs windowed
      mode). This parameter is true by default.

    --scaling <true | false>
      If set to true, the graphics output of the remote session will be
      scaled to fit inside the client window.

    --resolution "<integer integer>"
      The width and height of the remote session in pixels. The width and
      height must be positive integer values specified as a single string
      inside quotation marks (for example, "800 600").

    --colordepth <16 | 32>
      The color depth of the remote session in bits per pixel. Allowed
      values are 16 and 32.

    --retina <true | false>
      If set to true, the graphics output of the remote session is optimized
      for Retina displays. A value of true is not recommended for connections
      to versions of Windows prior to Windows 10 and Windows Server 2016.

    --dynamicdisplay <true | false>
      If set to true, the resolution of the remote session is dynamically
      updated to match the client window size. Dynamic display is only
      supported by Windows 8.1, Windows Server 2012 R2 and later.

    --audioplayback <0 | 1 | 2>
      Specifies how to handle audio streams sent by the remote server.
      0: Play the audio on the local computer.
      1: Play the audio on the remote server.
      2: Don't play any audio.
      This parameter is set to 0 by default.

    --redirectmicrophones <true | false>
      If set to true, microphones attached to the computer are redirected to
      the remote session.

    --redirectcameras <true | false>
      If set to true, video capture devices attached to the computer are redirected to
      the remote session.

    --redirectprinters <true | false>
      If set to true, printers attached to the computer are redirected to the
      remote session.

    --redirectfolders <true | false>
      If set to true, selected folders are redirected to the remote session.
      Due to sandboxing restrictions, users must manually select folders to
      redirect in the client UI.

    --redirectsmartcards <true | false>
      If set to true, smart card readers attached to the computer are redirected
      to the remote session.

    --redirectclipboard <true | false>
      If set to true, the local and remote clipboards are kept in sync. This
      parameter is true by default.

    --remoteappprogram <string>
      The file path of the remote app to launch on the remote server.

    --remoteappcmdline <string>
      Command line parameters to pass to the remote app when it is launched
      on the remote server.

    --remoteappworkingdir <string>
      The working directory assigned to the remote app when it is launched on
      the remote server.

    --rdpfilecontents <string>
      The contents of an RDP file which should be used to create a bookmark.
      The "\n" character must be used as a delimiter between properties.

  Examples:

    --script bookmark write 8725187 --hostname jumpbox.contoso.com --resolution "1024 768" --fullscreen false --group "Work PCs"
    --script bookmark write 5653454 --rdpfilecontents "full address:s:hostname\ngatewayaccesstoken:s:gyu7hj8o\nenablecredsspsupport:i:0"
```

## Welp. That's a lot of stuff! Can we make it more digestible?

Indeed! So, here are a couple of scripts I wrote that help make it manageable (for example, taking care of replacing pre-existing bookmark(s) with the same name, because Remote Desktop works with unique identifiers). Because Jamf is what I use, they're written around that, but you can adapt them to your management tool of choice.

Run them as part of a login (whilst login hooks are still a thing) or self-service policy, because someone must be logged in (the Microsoft Remote Desktop binary needs to run in their context, remember?). If users have the Microsoft Remote Desktop application open when these scripts run, they must quit and re-open it before they see the changes.

### Add or replace a bookmark

https://gist.github.com/neilmartin83/00fb2d045272219736d29e244b717527

Use this to add a Microsoft Remote Desktop bookmark entry for the specified computer. The bookmark's unique ID is generated automatically. Parameters should be filled out as follows:

**Parameter 4:** specify the hostname or IP address of the bookmark you wish to add e.g. `mypc.com` (required)

**Parameter 5:** specify the friendly name of the bookmark you wish to add, without quotes e.g. `My PC` (optional)

**Parameter 6:** specify the resolution, without quotes e.g: `1024 768` (optional)

**Parameter 7:** specify the group, if needed, without quotes e.g. `Company PCs` (optional)

**Parameter 8:** specify the username, if needed, without quotes. Or enter `loggedInUser` to use the current user's username! (optional)

**Parameter 9:** specify the domain, if needed, without quotes e.g. `myorg.com` (optional)

**Parameter 10:** specify other arguments as needed. Avoid specifying parameters that use quotes. (optional)  
For more information on available arguments, run:  
`"/Applications/Microsoft Remote Desktop.app/Contents/MacOS/Microsoft Remote Desktop" --script bookmark add help`

**Parameter 11:** Set to `YES` if you want to REPLACE all existing bookmarks that have the same hostname, if present. (optional). If this is not set, an additional bookmark for this hostname will be created if it already exists.

Here's an example of the parameters when they're filled out:

![](/images/image.png)

And here's some example output from the policy log, after a successful run:

![](/images/image-1.png)

A new bookmark will be created in Microsoft Remote Desktop, configured with your specified settings:

![](/images/image-3.png)

![](/images/image-4.png)

![](/images/image-5.png)

### Delete a bookmark

https://gist.github.com/neilmartin83/a35db3288c1a6e0f24db88df8dcde65c

Use this to delete all bookmarks with the specified friendly or host name. Beware that if there are multiple bookmarks with the same name, this deletes them ALL! Friendly names take precedence, hostnames are used when the bookmark does not have a friendly name set. Parameters should be filled out as follows:

**Parameter 4:** specify the name of the bookmark you wish to delete, without quotes, e.g. `My PC` (if it has a friendly name) or `mypc.com` (if it only has the hostname). (required).

Here's an example of the policy log, following a successful run:

![](/images/image-2.png)

## Further learnings...

Check out the #microsoft-rdc channel in the MacAdmins Slack to ask questions and chat to others about their scripting workflows. Special thanks go to Elton Saul (@eltons) and Gieta Laksmana (@gietal) from Microsoft, who hang out there and provide an invaluable resource for the Mac Admin community. Without them, the scripting functionality in Microsoft Remote Desktop 10 wouldn't even exist.
