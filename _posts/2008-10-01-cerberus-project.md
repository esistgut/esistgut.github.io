---
layout: post
title: Cerberus Project
date: 2008-10-01 19:55
author: Giacomo Graziosi
comments: true
categories: [linux]
tags: [c, cctv, cerberus, linux, motion, open source, programming, python, ruby, security, surveillance, video surveillance, vigilante]
---
<img src="/assets/cerberus.png" class="img-responsive" alt="Cerberus illustration">

---
What is this cool drooling puppy doing on my blog? His name is Cerberus, he came from Greek mithology and he is nothing less than the guardian of the gate to Hades, also known as Hell.

No, I haven’t joined satanism, this is just a mascotte, the mascotte for a new project I’ve been thinking on lately: a Debian based Linux distribution specialized on video surveillace, its name will obviously be Cerberus. :smile:

Why am I building another Linux distribution? Well, the answer is very simple: in the last two years I’ve been selling video surveillance systems build on top of Debian or Ubuntu with Motion and Vigilante. Assembling such systems requires to repeat each time a set of very specific steps such as installing the distribution with the needed packages, configuring the modprobe options for driver of the video capture card (often based on the bt878 chip), setup Motion and Vigilante, linking it all together on the user’s desktop. Most of these actions (excluding hardware probing/configuration) could be automatized, or better, saved prebuild in a proper environment.
This is where Cerberus Linux enters the game: it is meant to act like firmwares on embedded Linux devices, a main read-only partition containing the distribution with a small overlay read-write (freezable to read-only) partition containing the configuration files.
Why am I following the “embedded-firmware”-like way in place of a more common standard installation? First reason is robustness: making the system read-only prevents damage to the file system when doing hard resets (which are very common on this kind of systems, often seen as vhs recorders or table dvd players by the inexperienced users who will use them). Second reason is easy of use: you can (re)install and update the base system by rewriting the partition without caring about backups or installation process, is it just a call to dd, automatizable. Third reason: users are idiots, give them a writeable home and they will find a way to mess up the entire system, give them a writeable root file system and they will bring back the system as an ash heap claiming you sold them a broken thing.

So the roadmap so far is to build the distribution with the listed features, put a new Vigilante (based on Cluttermm/Gstreamermm) on it and maybe develop a remote web configuration system (maybe with ExtJS and Rails or Zend Framework or Webtoolkit).

Of course if you have any advice or proposal for Cerberus please contact me, any good idea will be welcomed.
