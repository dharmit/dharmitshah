+++ 
date = 2025-05-11
title = "Finally read a technical book!"
description = "Experience of reading OSTEP (Operating Systems Three Easy Pieces), or the Comet book"
tags = ["reading", "tech"]
+++

It's been 15 years since I completed the undergraduate course. Neither during
the course nor after it did I read a technical book from cover to cover. Then,
a few years ago, in October 2018, I came across [OSTEP (Operating Systems:
Three Easy Pieces)](https://pages.cs.wisc.edu/~remzi/OSTEP/) and, in an
impulse, purchased its physical copy. It stayed in my home library as "to read"
book. I remember reading a few pages when the book was first delivered. But
nothing beyond that.  However, ever since I read its few initial pages, I was
aware that the authors' writing style matched my learning style in that:

- It wasn't too formal
- It felt like the author was having a conversation with the reader rather than
  trying to preach/sermon

Moreover, I wanted to pick it up to understand OS better because, as a teen,
Linux got me very excited, but I could never pursue it as much as I should have

### Picking it up, at last

Fast forward to November 2024 and I came across this tweet: {{< x user="akJn99"
id="1842454174379671963" >}}

It felt like a signal from the universe to read the book because, back in
November 2024, I had addressed a few career and personal tasks, and in my head
I was clear about what I wanted to do: become an engineer I could myself
respect. To be able to be such an engineer, I had to learn, re-learn and
develop an understanding of few CS subjects that I had either forgotten or
never studied in the first place. E.g., I had forgotten most of what I had
learned in Operating Systems, Data Structures, Database, etc. classes, and
never studied Algorithms in the first place!

Now, in May 2025, I have finally finished reading the book while juggling
family and work responsibilities. I never thought I'd finish the book! It's
little over a month late than when the book club finished reading it, but
that's not a problem. Finishing the task has surprised and satisfied me the
most!

### What did I learn?

To be honest, I wasn't confident about finishing this because I have a history
of starting projects/tasks that don't cross the finish line. Especially the
technical books I have started always remained incomplete so, I was only
annotating the book here and there.

It would be beyond dumb of me to say that I now know everything in that book
perfectly. My goal was to _understand topics in the moment_ and not
care/bother/stress about remembering things any longer. And, indeed, almost
each time I left a chapter midway and came back to it after a few days (even 2
days), I had to surely re-read it. 

I once wrote this post titled ["Stack, Heap, Go"](/2022/09/stack-heap-go/) as a
"conversation" with some colleagues. However, the truth is that it was a
question I was asked in an internal interview at Red Hat when I applied for the
[KubeVirt](https://kubevirt.io) team. It was the very first question I was
asked over the multiple rounds of interviews I gave before eventually being
able to clear them and joining the team. But when I fumbled in that question, I
was 0% hopeful of being able to clear the interviews. It further cemented the
impostor syndrome thought process in my head.

The takeaway for me is that an OS does A LOT of stuff under the hood that we
don't give much thought to. But we need to understand those to write better
programs. Well, I always knew it does a lot under the hood, but the specifics
became clearer as I read OSTEP. Some of these include:

- scheduling a program on a multi-CPU system
    - so many approaches, so many intrecacies
        - well this is kind of true for everything that OS does
- managing the memory of programs
    - CPU cache and multi-CPU architecture makes this an even more challenging
      problem
- the stack and heap that I couldn't talk about in the interview were all over
  the place in the section on "memory virtualization"
- concurrency is crazy with C
    - initially I was excited about this section and dismissed the author's
      note in the "dialogue" chapter about this being a topic causing
      headaches.  Reason for this was having written some concurrent code in
      [Go](https://go.dev).
    - I quickly realized why Go's concurrency model and syntax are so very
      loved by polyglot programmers.
    - and I indeed ended the section with a headache.
- although the section on concurrency felt challenging, I'm positively excited
  [about learning further](https://greenteapress.com/semaphores/LittleBookOfSemaphores.pdf).
- the section on storage and file system felt a lot of fun!
    - probably because it was the last section so my mind remembers it the
      most! Remember I took almost six months to finish the book.
    - it might also be because authors are doing their own research around this
      topic, so their explanations here might have been more fun than they were
      in other sections.
- the amount of things happening to make an I/O request happen blew my mind
  away!
- filesystems are so easily taken for granted, but it's a whole field of
  research on its own. I also stumbled upon a [podcast episode about
  ReiserFS](https://corecursive.com/reiserfs/) while reading this section which
  added some context.
- the part about data staying in page cache before being actually
  flushed/written to the disk is also really interesting! I first learned about
  it in [this talk](https://www.youtube.com/watch?v=_55OM23zhUo) about
  questioning the design patterns of storage engines. I was at the conference so
  I saw it live and I had started reading OSTEP only a few weeks ago, so all the
  while I was thinking that this topic about `fsync()` system call is going to
  come up in OSTEP. The under the hood working of I/O is covered pretty
  amazingyly in the book!
- the section on distributed systems is, thankfully, short.
    - I did the CKA and CKAD certifications of Kubernetes in November 2020.
      When I shared it with my team at work, a
      [colleague](https://www.linkedin.com/in/pradeepto) recommended me reading the
      [Designing Data-Intensive Applications or
      DDIA](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/).
      I didn't say it to him, but, after looking at its table of contents, I
      felt too imbecile to read it. Now, I know I have a huge bout of impostor
      syndrome, but this book really felt way above my level. After being able to
      addres a few issues in 2024, I was clear that I want to first learn things from
      a single system's point of view, i.e., Operating Systems and Data Structure &
      Algorithms before thinking about Distributed Systems. This is because, for
      example, an OS maintains state of various processes and makes sure that each of
      them gets its fair share of time to run on the CPU. OS does so much at a single
      system level that, I think, can be translated into what a Distributed System
      does over connected machines. I want to learn these better before learning
      Distributed Systems.
    - yet the short discussion of distributed systems is enough and fun to
      spark the interest to learn more. Now I know that I'll read DDIA at some
      point in time.
- best and most accurate answer to a question about how to do something —
  anythnig — is "it depends". The computer is full of binary - 0 & 1. But
  reality is always between 0 & 1. It's like CAP theorem which states that one
  can have only two of consistency, availability, and partition tolerance. At a
  real job this often translates to being able to have, at max, only two of
  good pay, great WLB, and interesting work. :wink:

### What's next?

Looking at some of the commentary from folks who have read OSTEP as well as
what authors themselves mention a few times in the book, this book aims to give
only an introduction of a lot of topics. So it is up to the reader to dive
further into what piques their interest. For me it would be scheduling
algorithms, concurrency, and file systems.

But at the moment I want to focus on Data Structures and Algorithms because:
- it forms almost the absolute basics of the Computer Science domain
- companies often grind candidates on this topic

I had the privilege of discussing this with my [mentor at
work](https://github.com/fabriziosestito) and he explained how above two are
different in nature. One could learn the Algorithms like they did in university
by covering many algorithms, yet it won't help secure job at a company that
grinds on that topic because those interviews are focused on a few algorithms
and questions go into some depth of these. At the moment, I'm clear about doing
former and have picked up the book [Algorithm Design Manual by Steven
Skiena](https://www.algorist.com/Algorist_ed2/). I found it via [Teach Yourself
CS](https://teachyourselfcs.com/#algorithms) where it's mentioned as a book
that:

> typically succeeds in fostering similar enthusiasm among his students and
> readers

I'm more interested in enjoying learning it right now just like I enjoyed OSTEP
and [Let Us C](https://www.amazon.in/Let-Us-C-Yashavant-Kanetkar/dp/8183331637)
which made me enjoy learning C language at one point. The book we used in
university to learn C made it, to say it mildly, a torture to learn the
language!

### That's it!

Hope to write a post about finishing the book on Algorithms. :)
