---
title: Living in a Shell
date: '2018-06-19T22:17:23-05:00'
draft: false
---
I've successfully completed my first project for EE 312 (Software Design and Implementation I) using only a remote terminal. I've been familiarizing myself with vim and gdb along the way and it's honestly been quite fun. 

Today, I found a game-changing project on GitHub called [gdb-dashboard](https://github.com/cyrus-and/gdb-dashboard) that allows me to see the registers, stack, disassembly, and other things all with one command. The problem is, the UT ECE Linux machines that I ssh into only have Python 2.6, and the .gdbinit file is written with Python 3. So far I've made a [fork](https://github.com/cyrus-and/gdb-dashboard) and changed the {} fields so that the dashboard actually runs. In so doing, however, I introduced a bug causing the Source module to output empty lines instead of code.

I need to spend a few hours sometime this week to fix the code. If I can get the dashboard fully operational, I'll have eliminated my biggest fear related to doing pure command-line development: slow debugging.

I plan on completing the rest of the semester's assignments in a shell as well, just for the heck of it. I want to get a taste of what programming might have been like many years ago before GUIs were around. 
