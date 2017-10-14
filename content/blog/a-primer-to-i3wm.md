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

*Disclaimer: I'm far from having in-depth knowledge about display managers or
the protocols, tech stack, etc. that loads the GUI I see. All I care about is
being able to navigate through my screen quickly, through a keyboard.*

Few reasons that I can recall for moving to i3wm:

- Various window managers have different shortcuts for most basic actions.
  Sometimes shortcuts change between releases. 

    For example: *Alt+Space+X* used to maximize the window earlier but newer
    version of the manager didn't do anything upon hitting that key
    combination.

- Applications open in a not-completely-maximized state (there must be a one
  word for this) by default.

    For example: Terminal, broswer when opened in such state, need to be
    maximized for (my) usability but, devs removed the shortcut I was so used
    to - *Alt+Space+X*.

- Window manager crashes (this shouldn't require an example!)

- i3wm looked super cool! I know I'm a bit weird in that, I don't find window
  animations to be cool.

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

### Who should use i3wm?

In my opinion, i3wm is for users who are better off with keyboard and don't use
mouse too much. You also need to be good/quick at remembering shortcuts if you
want to avoid hitting the wall. i3wm docs suggest that it's intended for
advanced users and developers.

### Setting up i3wm

My i3config file (~/.i3/config) is heavily modified and I've picked things up
from various places. At the moment, I don't even recall what each of the
configuration parameter in that file does.

So in this post, we're going to setup i3wm on a fresh system. I'll be using a
Fedora 26 virtual machine.
