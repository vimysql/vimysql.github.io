---
layout: post
title: 'ALERT: Disable Transparent HugePages on SLES11, RHEL6, OEL6 and UEK2 Kernels (Doc ID 1557478.1)'
date: '2020-03-26'
header-img: "img/post-bg-android.jpg"
tags:
     - oracle
author: '王小胖'
---

# Disable Transparent Huge Pages (THP) 
Starting with RedHat 6, OEL 6, SLES 11 and UEK2 kernels, Transparent HugePages are implemented and enabled (default) in an attempt to improve the memorymanagement.  Transparent HugePages are similar to the HugePages that have been available in previous Linux releases.  The main difference is that the TransparentHugePages are set up dynamically at run time by the khugepaged thread in kernel while the regular HugePages had to be preallocated at the boot up time.

## THP特性对Oracle的影响
----
Because Transparent HugePages are known to cause unexpected node reboots and performance problems with RAC, Oracle strongly advises to disable the use ofTransparent HugePages. In addition, Transparent Hugepages may cause problems even in a single-instance database environment with unexpected performanceproblems or delays. As such, Oracle recommends disabling Transparent HugePages on all Database servers running Oracle.

## How to check if the Transparent HugePages are enabled
----
Default/Enabled setting is  [always]:
```
#cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never

```

Disabled setting is [never]:
```
#cat /sys/kernel/mm/transparent_hugepage/enabled
always madvise [never]

```

If "enabled" is NOT set to "[never]", the Transparent HugePages are being used.

You can also issue:
```
#grep AnonHugePages /proc/meminfo
```
If the output contains a line like "AnonHugepages: xxxx kB", with a value > 0kB the kernel is using Transparent HugePages.Because the kernel currently uses Transparent HugePages only for the anonymous memory blocks like stack and heap, the value of AnonHugepages in /proc/meminfo isthe current amount of Transparent HugePages that the kernel is using

## How to disable Transparent HugePages
----
Add the following to the kernel boot line in /etc/grub.conf (this is the preferred method) and reboot the server:
```
# cat /etc/grub.conf
# grub.conf generated by anaconda
#
# Note that you do not have to rerun grub after making changes to this file
# NOTICE:  You have a /boot partition.  This means that
#          all kernel and initrd paths are relative to /boot/, eg.
#          root (hd0,0)
#          kernel /vmlinuz-version ro root=/dev/mapper/vg00-lv_root
#          initrd /initrd-[generic-]version.img
#boot=/dev/sda
default=0
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title Red Hat Enterprise Linux 6 (2.6.32-754.el6.x86_64)
	root (hd0,0)
	kernel /vmlinuz-2.6.32-754.el6.x86_64 ro root=/dev/mapper/vg00-lv_root transparent_hugepage=never rd_NO_LUKS rd_LVM_LV=vg00/lv_root rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM LANG=en_US.UTF-8 rhgb quiet
	initrd /initramfs-2.6.32-754.el6.x86_64.img
```

OR

Add the following lines in /etc/rc.local and reboot the server:

```
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then   
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then   
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
```

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。