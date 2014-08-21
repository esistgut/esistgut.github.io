---
layout: post
title: SQLite3 support for Motion
date: 2008-09-30 14:49
author: Giacomo Graziosi
comments: true
categories: [linux]
tags: [c, motion, open source, open source, patch, programming, sqlite, sqlite3]
---
﻿One of the first problem occurred when developing the specialized video surveillance Linux distribution I’ve been working on these days is the database engine to be used to archive the list of recording files saved by Motion. As every write operation will be placed on a read-write file system layer on top of the main read-only layer (containing the whole “monolithic” distribution, something like the firmware on an embedded device) I have the need to store the whole database data in a compact manner, something like only one file and not similar to the /var/lib/mysql/ directory with its many entries.
The problem with this is that Motion only supported MySQL and PostgreSQL so I had to add the SQLite layer by myself. As a result I have published a [patch on the Motion’s wiki](http://www.lavrsen.dk/twiki/bin/view/Motion/SQLite3Patch), test it please :-).
