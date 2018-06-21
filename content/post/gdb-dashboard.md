---
title: gdb-dashboard
date: '2018-06-20T19:17:23-05:00'
draft: false
---
After staring at the [gdb-dashboard](https://github.com/cyrus-and/gdb-dashboard) code for a long time, I finally wrapped my head around the str.format() fields being abused\* in the list() method of the Source module. I fixed some of the field names, and the Source module outputs code correctly now, but I still want to rewrite the module so that I can be sure it will still work even if I change the style settings. 

All this trouble, just because the UT ECE Linux machines don't have Python 3!

\*Apologies to [the creator of the project](https://github.com/cyrus-and), but I really don't think anyone should be nesting three format fields in a string. Maybe I don't know anything though.
