+++ 
date = 2023-12-16
title = "WTP - bunch of `kernel-*` packages"
description = "What do all these kernel packages really do?"
tags = ["linux"]
series = ["what the package"]
+++

In this first post of the 'What The Package' series, I am trying to understand what a bunch of `kernel-*` packages installed on my Fedora 39 system are meant for. It only makes sense since, in itself, Linux is just the kernel. All the GUI on top of it is a part of the distro (or the distrubution) we use.

In today's run of `dnf update`, a few packages related to kernel were updated, namely:
* `kernel`
* `kernel-core`
* `kernel-modules`
* `kernel-core-modules`

OK that's interesting. I'm curious what's different between `kernel` and `kernel-core`. Based on the `dnf info` outputs for the two packages, the difference between the two is that `kernel-core` is the package containing the actual kernel, while `kernel` package is a "meta" package for the kernel (which is `kernel-core`). Also, the size of the `kernel-core` package is 66MB while that of `kernel` is 0 (zero)! Maybe it's just a few bytes so it's rounded to zero?

Rather than having answers, I have questions like:

* What's a "meta" package and why is it needed?
* How can a package be of zero size?

Upon asking a question on the [`r/fedora` subreddit](https://www.reddit.com/r/Fedora/comments/18j0176/whats_a_meta_package_and_can_it_be_removed/) I learned that a meta package refers to other packages as dependency. And it's helpful in uninstalling whatever dependencies it pulled in.

Anyway, I don't want to try and go too deep into understanding RPM itself, but rather what they provide. So I did `rpm -ql kernel-core-6.6.6-200.fc39.x86_64` which provides a list of files installed by the package being queried:

```sh
$ rpm -ql kernel-core-6.6.6-200.fc39.x86_64
/boot/.vmlinuz-6.6.6-200.fc39.x86_64.hmac
/boot/System.map-6.6.6-200.fc39.x86_64
/boot/config-6.6.6-200.fc39.x86_64
/boot/initramfs-6.6.6-200.fc39.x86_64.img
/boot/symvers-6.6.6-200.fc39.x86_64.xz
/boot/vmlinuz-6.6.6-200.fc39.x86_64
/lib/modules
/lib/modules/6.6.6-200.fc39.x86_64
/lib/modules/6.6.6-200.fc39.x86_64/.vmlinuz.hmac
/lib/modules/6.6.6-200.fc39.x86_64/System.map
/lib/modules/6.6.6-200.fc39.x86_64/config
/lib/modules/6.6.6-200.fc39.x86_64/modules.builtin
/lib/modules/6.6.6-200.fc39.x86_64/modules.builtin.modinfo
/lib/modules/6.6.6-200.fc39.x86_64/symvers.xz
/lib/modules/6.6.6-200.fc39.x86_64/vmlinuz
/usr/share/licenses/kernel-core
/usr/share/licenses/kernel-core/COPYING-6.6.6-200.fc39
```

The file `/boot/vmlinuz-6.6.6-200.fc39.x86_64` is the actual kernel that's booted into when the machine starts. An initramfs (`/boot/initramfs-6.6.6-200.fc39.x86_64.img`) is used for loading a temporary root file system into the memory when the system starts. A pretty good description of what other files in `/boot` are meant for can be found [here](https://www.unix.com/unix-for-advanced-and-expert-users/282286-knowing-boot-files.html).

Taking a look at list of files for `kernel-modules` using `rpm -ql kernel-modules-6.6.6-200.fc39.x86_64` shows up a tonne of files under `/lib/modules/6.6.6-200.fc39.x86_64/kernel/drivers/`. Same is the case when I do `rpm -ql kernel-modules-core-6.6.6-200.fc39.x86_64`. These are the drivers for the various hardware supported by the Linux kernel. The difference between the two packages is the kind of drivers they provide. A quick `dnf info` indicates that `kernel-modules` package provides the "commonly" used kernel modules, while `kernel-modules-core` provides the "essential" kernel modules for the core kernel package. So it's basically "common" vs. "essential".

### That's it
Looking up what various files under `/boot` mean was a good refresher. It's been long since I worked on or looked into things revolving around the Linux boot process. At some point, it was something I used to discuss with my fellow mentees.