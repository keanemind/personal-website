---
title: Changing ls Colors
date: '2018-09-16T12:00:46-05:00'
draft: false
---
My WSL bash had always suffered from very ugly `ls` outputs until yesterday, when I realized that the folders being listed with green backgrounds were a *certain type* of folder: other-writable.
I discovered that in the answer to [this stackexchange question](https://unix.stackexchange.com/questions/94498/what-causes-this-green-background-in-ls-output).

I had always thought that there was something wrong with my configuration because I had manually set `di=1;34` in my `LS_COLORS` environment variable, yet my folders still had a green background. I now know that I have to set `ow=1;34` and `tw=1;34` to change other-writable and sticky + other-writable directories as well.

However, I also now know that it is a lot easier to configure `ls` colors using the `dircolors` program. To configure my colors, I did `dircolors --print-database > ~/.dircolors`. I then opened that `.dircolors` file and changed the `STICK_OTHER_WRITABLE`, `OTHER_WRITABLE`, and `STICKY` values so that all my directories are displayed in the same way; I don't care about distinguishing between directories with different permissions.

I then wrote two things in my `~/.bashrc`:

1. `eval "$(dircolors -b ~/.dircolors)"`. This modifies the `LS_COLORS` environment variable.
1. `alias ls='ls --color=auto'`. This makes sure `ls` always uses the `LS_COLORS` variable to format its output.

The advantage of using the `.dircolors` file is that it's a lot easier to edit and understand as opposed to manually modifying the `LS_COLORS` value.
