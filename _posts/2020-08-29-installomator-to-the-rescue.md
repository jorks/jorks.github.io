---
title:  "Installomator to the Rescue"
date:   2020-08-28 19:32:34 +1000
excerpt_separator: "<!--more-->"
tags:
  - Jamf
  - Scripting
---
Recently I needed to deploy Minecraft: Education Editon to a few hundred Macs enrolled in Jamf Pro. A simple task, except for the fact the customer only had on-site distribution points for packages, and all the Macs were off-site for remote learning during COVID. Immediately I started counting my options, set up an AWS cloud distribution point, host the package on a web server and script the download and install, or write a custom script to run on the devices.

Then I remembered the Mac Admins Podcast discussing [Installomator](https://github.com/scriptingosx/Installomator) by Armin Briegel over at [Scripting OS X](https://scriptingosx.com/). 

<!--more-->

The idea of Installomator is simple, download a package directly from the vendor and use a processor to handle the vendor's _flavour_ of distribution and install techniques. It is similar to running the AutoPkg processors on the client computer. After reading the goals of the project on the [Github README](https://github.com/scriptingosx/Installomator#goals) I was sold.

**Spoiler alert** - this tool delivered on all its promises.
{: .notice--success}

A quick `Command + F` in the body of the script, no mention of Minecraft. Fortunately, there was documentation and lots of it. 

### Define a Custom Application

The options availble to define a software package are easy to interperate (especaially coming from AutoPkg) and the scripts built-in debug logs help alot. Here is the **finished** definition:

```bash
minecraftee)
    name="Minecraft Education Edition"
    appName="minecraftpe.app"
    type="dmg"
    downloadURL="https://meedownloads.azureedge.net/retailbuilds/MacOS/Minecraft_Education_Edition.dmg"
    expectedTeamID="UBF8T346G9"
    ;;
```

#### Learnings

I started by installing Mindcraft manually while taking note of the URL, TeamID, type (DMG) and name. With this, I added my first iteration of the custom app definition to the script and ran `./Installomator.sh minecraftee`. It failed. 

The `DEBUG` option is enabled by default, so immediately I could see what was going wrong. The App name I noted from the DMG was `Minecraft: Education Edition` however the script was reporting that it couldn't find it.

```
ERROR: could not find: /Volumes/Minecraft Education Edition/Minecraft: Education Edition.app
```

A quick `ls` in the mounted DMG and it turns out the App's name is `minecraftpe.app`. The script already handles this situation by using the optional `appName` variable. This happens when a developer uses localization overrides for `CFBundleDisplayName` and `CFBundleName.`

I also noticed the downloaded DMG was named `Minecraft/ Education Edition.dmg`. Finder had replaced the colon `:` with a forward slash `/` as the colon is prohibited character in file names and paths (to play nice with Windows drive letter naming convention I believe `C:`). To stay on the safe side I removed the colon from the name. 

**Hot Tip** 💁‍♂️ Always test internet scripts with `set -x` to understand what is executing before deploying to production computers. 
{: .notice--info}