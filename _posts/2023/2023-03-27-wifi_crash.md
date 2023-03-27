---
title: Kernel crash with Intel Ethernet Controller I225-V
---

This is a bookmark to remember how to fix a crash
of the kernel driver for Intel Ethernet Controller I225-V:

Modify `/etc/default/grub` and set

```
GRUB_CMDLINE_LINUX_DEFAULT="pcie_port_pm=off pcie_aspm.policy=performance"
```

Got it from [Reddit](https://www.reddit.com/r/buildapc/comments/xypn1m/network_card_intel_ethernet_controller_i225v_igc/).
