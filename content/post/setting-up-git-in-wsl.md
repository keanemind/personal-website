---
title: Setting Up Git in WSL
date: '2018-09-13T10:56:48-05:00'
draft: false
---
On my laptop, which runs Windows 10 Home, I use Windows Subsystem for Linux (WSL) to run an Ubuntu command line environment within Windows. With WSL, I can use Bash instead of Powershell for all my command line work. I have even configured my VS Code integrated terminal to use WSL's bash.

If you are a developer on Windows, I highly recommend setting up WSL. It's very easy, and it already comes with Windows 10.

However, there are two things you need to do to make your WSL Git work well with your files on Windows.

First, **in your WSL environment**, run `git config --global core.autocrlf true` to have Git use only CRLF line endings. If you don't set this, Git will show every file as modified.

[Relevant GitHub issue](https://github.com/Microsoft/WSL/issues/184)

Second, if you want to avoid having to constantly type your GitHub username and password, you must tell Git to use the correct credential helper. **In a Windows shell** (Powershell or CMD), do `git config --list` to check what your `credential.helper` value is. If you are using `credential.helper=manager`, you need to set your WSL git to use the same program.

In your Windows Git configuration, the line `credential.helper=manager` means that Git will run the program `git-credential-manager.exe`. However, your WSL Git will not know where to find that program. What you need to do is set the full filepath of that program in your WSL Git configuration.

**In Windows**, run `git --exec-path` to find the folder where `git-credential-manager.exe` should be. Confirm that it is in the folder outputted by the command.

Next, **in WSL**, do `git config --global credential.helper "/mnt/c/Program\\ Files/Git/mingw64/libexec/git-core/git-credential-manager.exe"` to set the path.

Instead of the path that I used, use the path you got from the exec-path command, with the `/git-credential-manager.exe` part appended. Remember to escape spaces and backslashes. Also, make sure the path you input starts with a `/` (e.g. `/mnt` instead of `mnt`). If the value of the `credential.helper` setting in your Git config does not start with a `/`, Git will try to append the value to "git-credential-" and execute that.

[Relevant StackOverflow question](https://stackoverflow.com/questions/45925964/how-to-use-git-credential-store-on-wsl-ubuntu-on-windows)

Have fun with WSL!
