+++ 
date = 2023-03-23T16:58:48+05:30
title = "Find the last screen unlock time in Linux"
tags = ["linux"]
+++

Today, I needed to find the time when I last unlocked the screen. My setup is Fedora 37 Workstation with GNOME. Upon 
doing a web search, I stumbled upon [this answer](https://askubuntu.com/a/435103) on the "ask Ubuntu" forum. But
the file `/var/log/auth.log` didn't exist on my system. A quick `sudo find /var/log/ -name "*auth*"` yielded nothing 
either.

Something made me think that it might be getting logged in the systemd journal messages. I locked and unlocked the 
screen, and opened journal logs using the `--since` flag. Sure enough there was a message with the word "unlock" in it.

So the trick is to simply use `grep`. Here are all the occurrences of "unlock" since this morning:

```shell
$ journalctl --since "2023-03-23" | grep unlocked
Mar 23 10:57:35 fedora gdm-password][2292]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Mar 23 12:39:25 fedora gdm-password][17567]: gkr-pam: unlocked login keyring
Mar 23 14:39:03 fedora gdm-password][23871]: gkr-pam: unlocked login keyring
Mar 23 15:15:29 fedora gdm-password][30321]: gkr-pam: unlocked login keyring
Mar 23 15:27:05 fedora gdm-password][30596]: gkr-pam: unlocked login keyring
Mar 23 15:48:29 fedora gdm-password][33137]: gkr-pam: unlocked login keyring
Mar 23 16:44:33 fedora gdm-password][36163]: gkr-pam: unlocked login keyring
```