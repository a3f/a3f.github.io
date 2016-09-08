---
layout: post
title:  "vmware-user on Debian 8 (Jessie)"
date:   2016-09-08 00:26:00
categories: linux
permalink: /articles/vmware-user-debian8-jessie
---

How to get a guest Debian 8 VM's desktop to resize to match VMWare's window.

***TL;DR:***

```
sudo apt-get install open-vm-tools-desktop
vmware-user-suid-wrapper
```

---

When attempting to install the VMWare Tools on Debian 8, one is greeted with following message:

> It is recommended to use the `open-vm-tools` packages provided by the operating system

But unfortunately, `open-vm-tools` don't provide the [`vm-user`](https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vmtools.install.doc%2FGUID-FB7FA658-7622-4680-BD9F-7A6670C12794.html) binary that provides the integration with X.

If you choose to ignore the recommendation, and install VMWare's integration tools, there is no `vm-user` binary either, unlike on Debian Wheezy.

The fix is a very easy one, but googling `"Debian 8" "vmware-user"` didn't tell me it:

`open-vm-tools` has the Desktop integration as a separate package, which you need to install:

> [open-vm-tools-desktop package](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2073803)
> 
> This optional package extends OVT with additional user-space programs and libraries to improve the interactive functionality of virtual machines. This package depends on X and therefore must be installed only when X is available. These features are enabled by this package:
> <ul><li>Enables resizing of the guest display to match host console window or the VMware Remote Console Window for vSphere</li>
> <li>Enables text copy and paste operation between host and guest UI (either direction)</li>
> <li>Enables drag and drop operation between guest and host (either direction) for the VMware Workstation and VMware Fusion products (not supported on vSphere) </li></ul>

Maybe, someone stumbles upon this and saves some time.
