---
title: "Grubby"
date: 2023-08-19T12:51:02+03:00
---

# Grubby

Grubby is the tool of choice in Fedora linux to review and manipulate the kernel you want to load. It is not uncommon an update to break something in your system, for instance to mess with suspend/resume, kernel modules etc. Grubby can help to revert back to the previous kernel.

#### Show all kernels

    # grubby --info=ALL

#### Select a kernel by index

    # grubby --set-default 1

Reboot for changes to take effect