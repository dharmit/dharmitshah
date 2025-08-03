+++
title = "Now"                           
date = 2025-08-03
+++

*_Last Updated: 2025-08-03_*

Inspired by various folks I see keeping a "Now" tab on their personal website, I'm attempting to create one for myself. The idea is to keep this page updated with things I'm working on right now. I'll split it into two sections: [Work](#work) and [Outside Work](#outside-work), because like any software engineer out there, I get paid to do certain things, and my curiosity leads me to do some other things too.

### Work

My resume is usually up-to-date. Take a look at it [here](https://bit.ly/dharmitresume).

I am presently working on Rancher, which is SUSE's distribution of Kubernetes, and various components that make up Rancher. The nature of my work is such that I do not focus on one particular component, but look at anything and everything that comes my way. So far, I have contributed to Observability, Provisioning, CAPI (mainly CAPRKE2) and Harvester in varying capacities, to the best of my abilities. These contributions happened while working exclusively with these teams.

Nowadays, I am not working full-time with any team. Whatever customer escalations come to Engineering are supposed to first pass through my team. So I work on anything that needs to be worked upon. That means less work on features and more on helping customers solve the problems they are facing.

### Outside Work

My goal is to work closer to the systems. Less layers of abstraction between what I do and the Operating System, which is, of course, Linux! But it's a journey I'll have to pave with hard work and discipline while managing work and family. I am in this for the long haul.

In first half of 2025, I read a book that was on my to-read shelf for a very long time: [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/), and wrote about my learnings [here](../2025/05/ostep/). It helped me develop more empathy for the kernel and the hardware than what I already had.

Post that, I decided to go through [Algorithm Design Manual](https://archive.org/details/2008-book-the-algorithm-design-manual/) as I didn't focus well on DSA during undergraduate studies, whereas, alas, these days job interviews are mostly about grinding folks on LeetCode. :disappointed:

However, I couldn't keep myself hooked to the book as I could with OSTEP. Coincidentally around this time, I had a chat about this with [Luca Cavallin](https://www.lucavall.in/) after reading his amazing posts about [Kubernetes newtworking](https://www.lucavall.in/blog/kubernetes-networking-from-packets-to-pods) and [Linux kernel](https://www.lucavall.in/blog/a-quick-journey-into-the-linux-kernel). Following his suggestion, I'm currently reading [Grokking Algorithm](https://www.manning.com/books/grokking-algorithms) and implementing code in [Go and Rust here](https://github.com/dharmit/grokking_algorithms/). It's progressing at a snail's pace, but hey I have a full-time job and a family to prioritize! :wink:

In July 2025, my [first code contribution to Kubernetes](https://github.com/kubernetes/kubernetes/pull/132604) got merged. It was a non-trivial enhancement about adding JSON and YAML outputs to `kubectl api-resources` command. What makes it special for me is that:
1. It wasn't a `good-first-issue`; maintainers told me it was a non-trivial ask.
2. It was a contribution sans employement; it has nothing to do with my employment.
3. It took more than a year from starting working on it to getting the PR merged. I learned that:
   * Serious open source development is a long-running task.
   * My personal ability to persist and persevere is way better than I ever knew!

I intend to contribute more to Kubernetes [sig-cli](https://github.com/kubernetes/community/tree/master/sig-cli) (because I have worked quite a bit on Go CLI development) and [sig-node](https://github.com/kubernetes/community/tree/master/sig-node) (because it seems like a way to get closer to the systems) in future and will update here if something manifests!