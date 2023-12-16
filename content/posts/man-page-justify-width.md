+++ 
date = 2023-12-16
title = "Centre align a Linux man page"
description = "Set margin and width for a man page"
tags = ["linux"]
+++

As a Linux user, I often have to refer to the manual pages to learn things. Now I don't have a huge monitor as many people have these days, but I have a 24" screen and it's not the best experience when reading a man page.

Today when I opened a man page, I found it so unreadable that I did a quick web search for "man page width" and found that the variable `MANWIDTH` controls how broad the output should be. That led me to search for "man page margin" and then "man page centre justified" which landed me on [this answer](https://unix.stackexchange.com/a/478100/4335).

Using this info, I first created a `~/bin/olivetti` file that's exactly as mentioned in that answer with the only exception that I have used `WIDTH=$MANWIDTH` instead of `WIDTH=100`:

```shell
$ cat ~/bin/olivetti
#!/bin/sh
# Define desired width of the text.
WIDTH=$MANWIDTH
# Evaluate left indentation based on terminal width.
INDENT=$(( ( $(tput cols) - $WIDTH ) / 2 ))
# Make line of that amount of spaces.
INDENT_LINE=$( printf %${INDENT}s )
# Put it on the beginning of each line of the input file.
sed "s/^/${INDENT_LINE}/" -
```

And then put a simple function in `~/.zshrc` file that looks like:

```shell
function m {
    man $1 | olivetti | less
}
```

Now, to open a center justified man page that's only 100 columns wide, I run `m` followed by whatever it is that I'm looking up, e.g., `m bootup` to read the Linux bootup process.