---
title:  "Reviving a Mac Pro 5,1 for ESXi 6.7 or 7.0"
date:   2020-08-27 19:32:34 +1000
tags:
  - ESXi
toc: true
---
Recently I was offered a free "_decommissioned_" Mac Pro (Mid 2010) to re-purpose as I pleased. I like the idea of giving new life to an old Mac, the only issue with this free computer, was that it doesn't run new software - but we can fix that. And so began my journey to install ESXi 7.0 and get Big Sur running on a Mac from 2010.

While old, it was a beefy Mac Pro with great specs for visualising macOS VMs, and I've needed an always-on workhorse for Mac Admin and Jamf Pro tasks. So, what can a free computer get you these days:

```Shell
Mac Pro (Mid 2010) MacPro5,1
Intel Xeon W3530 2.80 GHz Processor 
34GB ECC RAM 🤓
1TB SSD & 2TB HDD
AMD something something card.
```

The plan for this new-old Mac is to host daily workflows:

- Automated macOS testing workflows using a combination of AutoDMG, vFuse, VMWare for testing packages, DEP and beta releases.
- Host a dedicated AutoPkg environment. I'm also interested to try [PatchBot](https://macintoshguy.wordpress.com/2020/07/03/patchbot/), just announced for vJNUC2020.

### TL;DR - The Process

1. Install the highest supported ESXi version for your hardware
2. SSH in and use the `esxicli` to upgrade to desired ESXi version
3. **Before** rebooting to apply the update, edit the `/altbootbank/boot.cfg` file, append `allowLegacyCPU=true` to the `kernelopt` line
4. `reboot` to apply the update
5. If the edit was successful it will bypass the incompatible CPU warning
6. When you create VMs, you may need to add `monitor.allowLegacyCPU = “true”` in the `.vmx` file.

If you get stuck, read on.

### ESXi Versions & Compatibility

If your host Mac supports the latest ESXi, then just install that and forget about this hot mess.

If however, you are using an older hand-me-down Mac, you have the option to hack a newer version of ESXi onto unsupported hardware - which is very achievable. The main reason you would do this is guest VM OS compatibility.

Newer versions of macOS require a higher `virtualhw.version`, which in turn, require a higher ESXi version.

- macOS 10.14 Mojave - Minimum `virtualhw.version = "14"` requires ESXi 6.7
- macOS 10.15 Catalina - Minimum `virtualhw.version = "15"` requires ESXi 6.7 Update 3 ([VMWare KB Article](https://kb.vmware.com/s/article/78980))
- macOS 11 Big Sur 🤷‍♂️ - virtuallyGhetto has a [post](https://www.virtuallyghetto.com/2020/06/macos-10-16-big-sur-beta-1-on-esxi.html) with it running on ESXi 6.7

Let's move on.

### Create an EXSi USB Installer

Check out these very detailed guides on VirtuallyWired, complete with pictures and terminal output:

- [Create a Bootable ESXi 6.X Installer USB Flash Drive on macOS](https://virtuallywired.io/2019/08/18/create-a-bootable-esxi-installer-usb-flash-drive-on-macos/)
- [Create a Bootable ESXi 7 USB Installer on macOS](https://virtuallywired.io/2020/08/01/create-a-bootable-esxi-7-usb-installer-on-macos/)

_Note: Start by creating an installer that is compatible with your host Mac._

### Install ESXi

With the USB installer inserted, boot the Mac holding `option` and follow the prompts. Configure your network preferences and [enable SSH](http://www.virtubytes.com/2017/04/21/enable-ssh-vmware-esxi-6-5/).

### Upgrade ESXi

After installing the compatible version of ESXi, it's time to upgrade it to an incompatible version. 

Originally I started down a rabbit hole trying to modify the ESXi installer to trick it into thinking the CPU was compatible - this process is much easier. 

**Hot Tip** It is easy to confuse ESXi installable versions and patches. Installers are suffixed by the build number `ESXi-6.7.0-8169922-standard` and patches are suffixed by the release date `ESXi-6.7.0-20200804001-standard`.
{: .notice--info}

SSH into your ESXi host:

1. Enter Maintenance Mode

	```bash
	vim-cmd /hostsvc/maintenance_mode_enter
	```

2. Allow http downloads through the firewall

	```bash
	esxcli network firewall ruleset set -e true -r httpClient
	```

3. Find your desired version - Refer to the [VMWare article for build numbers](https://kb.vmware.com/s/article/2143832)

	```bash
	esxcli software sources profile list -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml
	```

4. Install the desired version and **do not restart** to apply updates

	```bash
	esxcli software profile update -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml -p ESXi-6.7.0-8169922-standard
	```
	
5. At this point, there is a new `boot.cfg` file staged for the next reboot. View this file `cat /altbootbank/boot.cfg` and confirm the desired build at the end of the file (eg `build=6.7.xxxx`).

5. Edit the file `vi /altbootbank/boot.cfg` and append `allowLegacyCPU=true` to the `kernalopts` line. It should look like this:
   
	```yaml
	bootstate=1
	kernel=b.b00
	kernelopt=installerDiskDumpSlotSize=2560 no-auto-partition allowLegacyCPU=true
	modules=jumpstrt.gz --- useropts.gz <<snip snip blah blah long list of modules that go on and on and on>>
	build=6.7.0-0.0.8169922
	updated=2 
	```
        
3. You can now `reboot` to complete the update. 

### Apply ESXi Patches

When you apply ESXi patches, changes to the `boot.cfg` should persist. I recommend you check the **new** `/altbootbank/boot.cfg` before you reboot.

ESXi patches are cumulative, so you can install the latest patch available for your release. To find that patch, your can `grep` the output of of the `esxcli software sources profile list` command from step 3. For example:

```bash
# Find the desired patch
esxcli software sources profile list -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml | grep 6.7.0-2020
# Install the patch
esxcli software profile update -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml -p ESXi-6.7.0-20200804001-standard
# Check the kernalopts still have allowLegacyCPU=true
cat /altbootbank/boot.cfg
# Reboot to apply patch
reboot
```

### Creating VMs
When creating VMs, you must consider the ESXi and macOS requirements, ensuring the correct `virtualhw.version = "xx"` is used. If you use vFuse, use the `--esx` to create a pre-allocated ESX-type VMDK and you may also need to add `monitor.allowLegacyCPU = “true”` in the `.vmx` file.

### Useful Links (aka Credits)

If you have ever been down the path of using a Mac as an ESXi hypervisor, you are probably familiar with some of the challenges. A collection of helpful resources:

- [virtuallyGhetto](https://www.virtuallyghetto.com/apple) blog by William Lam. Most of your Googling about ESXi home-labs will lead you back to virtuallyGhetto, so start at the source.
- [Der Flounder](https://derflounder.wordpress.com/category/vmware-esxi/) by Rich Trouton. Has a great collection of posts about ESXi with a Mac focus
- [VMWare Compatibility Guide](https://www.vmware.com/resources/compatibility/search.php) - Understand what is supported, then choose to do something that isn't.
- [VMWare Guest OS Compatibility Guide](https://www.vmware.com/resources/compatibility/pdf/VMware_GOS_Compatibility_Guide.pdf) - important for guest OS to ESXi version matching.
- [VMware  Download Center](https://my.vmware.com/group/vmware/evalcenter?p=free-esxi6) - Don't forget to save the **license key** for later.