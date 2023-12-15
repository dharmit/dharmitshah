+++ 
date = 2023-12-15T17:36:10+05:30
title = "New series WTP - \"What The Package\""
description = "Discuss one or more Linux packages in each post"
tags = ["linux"]
series = ["what the package"]
+++

I have been a Linux user since 2009. The first command I run after booting my laptop each day is `sudo dnf update -y`. It's like having OCD now. More often than not, I am not even aware of what the various packages I see in the output of this command are responsible for. Lately, I have been intrigued to learn more about them, understand what they're doing or which software's dependency they were pulled in as, and maybe uninstall the ones I don't need.

There are a ton of packages installed on each Linux system. Literally:

```sh
$ rpm -qa | wc -l
2505
```

Through this series, I hope to learn things I knew didn't know, and also things I didn't know I didn't know. It's not a typo, rather one of my favourite quotes:

> You don't know what you don't know.