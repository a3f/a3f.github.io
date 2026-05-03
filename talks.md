---
layout: page
title: Talks
permalink: /talks/
public: true
nocomments: true
---

## Secure by Default? Unified Bootflows from Factory to Field

OSADL COOL April 2026 edition
[\[abstract ☍\]](https://www.osadl.org/COOL-2026-04-Compact-OSADL-Online-Lect.cool-compactosadllectures2026-04.0.html)

Discussion of a "Secure-by-Default" boot model combining barebox's new security features, hardening measures as well as non-technical improvements in the release process.

## Build Once, Trust Always: Single-Image Secure Boot with barebox

FOSDEM 2026: Embedded Devroom
[\[slides & recording ☍\]](https://fosdem.org/2026/schedule/event/SZRFKD-barebox-single-image-secure-boot/)

Eliminate multiple bootloader variants while preserving secure debugging and recovery using new barebox functionality.

## Netboot without throwing a FIT

FOSDEM 2026: Kernel Devroom
[\[slides & recording ☍\]](https://fosdem.org/2026/schedule/event/7XHGHX-netboot_without_throwing_a_fit/)

Use Kbuild FIT support and initramfs tricks to easily network boot a kernel and its modules without touchig the on-disk rootfs. The conference-driven development resulted in barebox's [devboot feature](https://www.barebox.org/doc/latest/user/devboot.html).

## Hardening the Barebox Bootloader

Linux Security Summit Europe 2025
[\[schedule ☍\]](https://lsseu2025.sched.com/event/25GEc/hardening-the-barebox-bootloader-ahmad-fatoum-pengutronix?iframe=no&w=100)
[\[recording ☍\]](https://www.youtube.com/watch?v=tav_ct4emX8)

Recounting recent efforts to improve barebox security posture.

## Bootloaders Under Fire: Real-World Threats and Practical Defenses

Embedded Linux Conference Europe 2025
[\[schedule ☍\]](https://osseu2025.sched.com/event/25VqL/bootloaders-under-fire-real-world-threats-and-practical-defenses-ahmad-fatoum-pengutronix?iframe=yes&w=100%&sidebar=yes&bg=no)
[\[recording ☍\]](https://www.youtube.com/watch?v=dermEhoAu1I)

Looking at some attacks against U-Boot and barebox bootloaders and how barebox changed in response.
First introduction of the barebox [security policy framework](https://www.barebox.org/doc/latest/user/security-policies.html).

## Das Hoch und Runter mit ARM-Systemen (German)

FrOSCon 2025
[\[schedule ☍\]](https://programm.froscon.org/2025/events/3378.html)
[\[recording ☍\]](https://media.ccc.de/v/froscon2025-3378-das_hoch_und_runter_mit_arm-systemen)

A (German) walkthrough of NXP i.MX8M bootstrap. From Boot ROM through barebox to Linux and back to power-off.

## usb9pfs: network booting without the network

FOSDEM 2025: Embedded Devroom
[\[slides & recording ☍\]](https://fosdem.org/2025/schedule/event/fosdem-2025-6103-usb9pfs-network-booting-without-the-network/)

This talk discusses the design of 9p and usb9pfs and showcase how streamlined development on a Yocto root file system can be with both barebox and Linux making use of usb9pfs.

---

## Taming DMA: Tales Wrestling Memory Corruption

Embedded Linux Conference Europe 2024
[\[slides ☍\]](https://osseu2024.sched.com/event/1ej1v/taming-dma-tales-wrestling-memory-corruption-ahmad-fatoum-pengutronix)
[\[recording ☍\]](https://www.youtube.com/watch?v=OMaOco0W-4Q)

I speak about how strangely DMA bugs can manifest as an excuse to generate Yu-Gi-Oh! cards.

## Linux Matchmaking: Helping devices and drivers find each other

FOSDEM 2024: Kernel Devroom
[\[slides & recording ☍\]](https://archive.fosdem.org/2024/schedule/event/fosdem-2024-3222-linux-matchmaking-helping-devices-and-drivers-find-each-other/)

A gentle introduction into how Linux device driver probing works.

---

## One Image to Rule Them All: Portably Handling Hardware Variants

Embedded Recipes 2023
[\[slides ☍\]](http://embedded-recipes.org/2023/schedule/one-image-to-rule-them-all/)
[\[recording ☍\]](https://www.youtube.com/watch?v=ViaLmrrMhqk)

I talk about how to design an image that is portable to many differnt boards.

## Wenn Geräte an Bäumen wachsen: Linux-Device-Tree-Portierung (German)

Chemnitzer Linux-Tage 2023
[\[slides ☍\]](https://chemnitzer.linux-tage.de/2023/de/programm/beitrag/251/)
[\[recording ☍\]](https://media.ccc.de/v/clt23-251-wenn-gerate-an-baumen-wachsen-linux-device-tree-portierung#t=1)

A (german) introduction into device trees as used by Linux and barebox.

## Having Something to Hide: Trusted Key Storage in Linux

FOSDEM 2023: Kernel Devroom
[\[schedule & recording ☍\]](https://archive.fosdem.org/2023/schedule/event/sth_to_hide/)

Introduction to the kernel's trusted key subsystem and my work in enabling it for unattended disk decryption on NXP's i.MX line of embedded SoCs.

---

## From Zero to A/B: Swimming Upstream with Yocto, Barebox and RAUC

Embedded Linux Conference Europe 2022
[\[slides ☍\]](https://osseu2022.sched.com/event/15zDg/from-zero-to-ab-swimming-upstream-with-yocto-barebox-and-rauc-roland-hieber-ahmad-fatoum-pengutronix-ek)
[\[recording ☍\]](https://www.youtube.com/watch?v=84Za-iWEcNU)

Building an OTA-capable Yocto-based BSP with mainline components and no vendor layer.

## DOOM auf STM32: Barebox Mars Domination (German)

Chemnitzer Linux-Tage 2022
[\[slides & recording ☍\]](https://chemnitzer.linux-tage.de/2022/de/programm/beitrag/228/)

For kicks, I ported DOOM onto a MMU-less STM32F4 microcontroller and talked about it.

---

## DOOM portieren für Einsteiger - Heavy Metal auf Bare Metal (German)

FrOSCon 2021
[\[slides ☍\]](https://programm.froscon.org/2021/events/2687.html)
[\[recording ☍\]](https://media.ccc.de/v/froscon2021-2687-doom_portieren_fur_einsteiger)

"DOOM as a boot splash. How, why and how to get it on your nearest
home appliance".
A (German) walkthrough on how to leverage barebox APIs to run DOOM on any hardware supported by barebox.

## From Reset Vector to Kernel - Navigating the ARM Matryoshka

FOSDEM 2021: Embedded Devroom
[\[slides & recording ☍\]](https://archive.fosdem.org/2021/schedule/event/from_reset_vector_to_kernel/)

A walkthrough of NXP i.MX8M bootstrap. From Boot ROM through barebox to Linux.

## Initializing RISC-V: A Guided Tour for ARM Developers

Embedded Linux Conference Europe 2021
[\[slides ☍\]](https://osselc21.sched.com/event/lAVT)
[\[recording ☍\]](https://www.youtube.com/watch?v=70oYYuflFLs)

A guide through the RISC-V architecture and some of its ISA extensions
and a walkthrough of the barebox port to the Beagle-V Starlight.

---

## Beyond "Just" Booting: Barebox Bells and Whistles

Embedded Linux Conference - Europe 2020
[\[slides ☍\]](https://osseu2020.sched.com/event/eCBp/beyond-just-booting-barebox-bells-and-whistles-ahmad-fatoum-pengutronix)
[\[recording ☍\]](https://www.youtube.com/watch?v=fru1n54s2W4)

Porting barebox to a new STM32MP1 board and a general discussion of design choices like multi-image, VFS, POSIX/Linux API, fail-safe updates, boot fall-back mechanisms, etc.
