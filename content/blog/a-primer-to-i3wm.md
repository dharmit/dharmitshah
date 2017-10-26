+++
title = "A primer to i3wm"                           
date = 2017-10-14T19:10:25+05:30
type = "post"
draft = true
tags = ["linux"]
+++

I've been using [i3wm](https://i3wm.org/) for about 3 years now. It took me
some time to get hang of its concepts and keyboard shortcuts but, I've never
felt the need to go back to GNOME/KDE/Cinnamon. I've used all these window
managers in past.

### What is i3wm?

i3wm is a tiling window manager. There's a lot of information available on the
Internet that explains what a tiling window manager is. Simply put, by default,
it doesn't stack application windows on top of each other. 

So if you're developing on vim/emacs/atom in one window and want to get
feedback on your code in the browser, you don't need to hit Alt+Tab to switch.
You can stack the terminal and browser side-by-side and get immediate feedback.

And there's no mouse movement involved. In other window managers you'd resize
the two windows and stack 'em up such that you can see both the windows. Not
required with i3wm!

i3wm would also fills up the entire space available on your screen. And it does
this without letting one application block another. 

### Why I moved to i3wm:

- Various window managers have different shortcuts for most basic actions.
  Sometimes shortcuts change between releases. 

    For example: *Alt+Space+X* used to maximize the window earlier but newer
    version of the manager didn't do anything upon hitting that key
    combination.

- Its resource usage is very, very low compared to fancy desktop environments.

- Applications open in a not-completely-maximized state (there must be a one
  word for this) by default.

    For example: Terminal or broswer when opened in such state, need to be
    maximized for (my) usability but, shortcut I was so used to was removed -
    *Alt+Space+X*.

- Window manager crashes (this shouldn't require an example!)

- i3wm looked super cool!

### Who should use i3wm?

In my opinion, i3wm is for users who are better off with keyboard and don't use
mouse too much. You also need to be good/quick at remembering shortcuts if you
want to avoid hitting the wall. i3wm docs suggest that it's intended for
advanced users and developers.

### Setting up i3wm

My i3config file (`~/.i3/config`) is heavily modified and I've picked things up
from various places. At the moment, I don't even recall what each of the
configuration parameter in that file does.

So in this post, we're going to setup i3wm on a fresh system. I'll be using a
Fedora 26 virtual machine in this guide. Except the installation, most things
should be distro agnostic.

Installation:

```bash
$ sudo dnf -y install i3
```

Now logout from your current session and when you are on the login screen,
select i3. If you're using GNOME, chances are your default login manager is
`gdm`. There's a gear icon right besides "Sign In" button which lists the
available window managers.

If it doesn't show up, try rebooting the system. It should show up after a
reboot.

Once you login, i3 detects that this is the first time we're using it and asks
us if we want it to create a configuration file for us or do we want to do it
manually? Let it create the file and move further.

Next, it asks us for the key to be used as "default modifier". This is the key
that we'll be using for configuring most of our shortcuts. There are two
options: Windows key and Alt key. Since Windows key is of no use on our Linux
system (when using i3), I prefer to go with Win key and press Enter.

It creates a file `~/.config/i3/config` to store the i3 configurations. But you
might be wondering how to navigate around through the blank screen in front of
you.

Hit "Mod+Enter" where Mod is the default modifier you selected. In this case,
"Windows key + Enter". This should open up a terminal. If the terminal emulator
it opened is not of your liking, open the config file and change it.

### Changing default terminal emulator

Different people like different terminal emulators. And at times this
discussion turns into a heated debate! I chose `xfce4-terminal` as my default
terminal emulator since it's minimal and doesn't get in my way. But it doesn't
come installed by default unless you're using XFCE spin of your favorite Linux
distro.

Installation:

```bash
$ sudo dnf -y install xfce4-terminal
```

Let's ensure that i3 launches it when we hit Mod+Enter:

```bash
$ cat /etc/environment
TERMINAL=xfce4-terminal
```

Here's the corresponding line in i3's config:

```bash
$ grep sensible ~/.config/i3/config
bindsym $mod+Return exec i3-sensible-terminal
```

Either one can replace `i3-sensible-terminal` with `xfce4-terminal` in the
config file or set the environment variable in `/etc/environment`. I've
modified the configuration file.
