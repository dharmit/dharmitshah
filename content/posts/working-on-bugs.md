+++ 
date = 2025-03-19T18:06:38+05:30
title = "Embracing working on bugs"
description = "On presently working more on fixing bugs than developing features"
tags = ["go", "kubernetes", "suse"]
+++

I joined [SUSE](https://suse.com) a year back. My work is about being a bridge between Technical Support and Product Engineering teams.Let me try to break that down:
* When Technical Support folks can't find a solution to the user's issues, they escalate it to the Product Engineering
* Such issues need to be triaged from code perspective and, if any change is required in the code, it usually ends up being a bug fix
* Depending on the severity of the issue, there's either a hotfix (a quick bug fix that's tailored to help the specific user) or a proper long-term fix (a fix that goes out to all users at the same time by following usual development, testing and release processes).

Above might not be 100% accurate as I am still ramping up, but it's quite close, I'm sure.

## Yikes, fixing bugs is so below par work!

For a long time, I used to see this work as a lame thing to do. And a lot of people I know see it the same way. It's probably seen as work outsourced to offshore teams. I haven't worked in enough companies to know if that's really the case, but there's a perception that cool engineering work happens in the West whereas maintenance and bug fixing happens offshore.

But I no longer see it as a subpar work done by cheap or less-skilled engineers.

## How did I learn to not dislike it?

It would be a lie to say that I 100% enjoy bug fixing while not doing any feature development. No developer, or a wannabe serious developer, likes it wholeheartedly. And that's why I learned to "not dislike" it instead of "starting to love" it. The latter can never happen if the work is only about fixing bugs and not developing any feature. Never ever.

Yet, a few reasons helped me start to not dislike it or, shall I say, 'embrace' it.

### My career background

In spite of being good at coding, I started out as a Linux admin. The short explanation for "why the f**k would I do that" is:
1. I was hell bent on working on Linux right out of the college,
2. There was no mentor to guide me what to do,
3. Internet wasn't as easily accessible in India back then (2010),
4. And I needed to start earning immediately after college.

After working as admin and technical supoprt engineer for a few years, I realized that in graveyard shifts weren't for me, and that a career in engineering _along with_ working on Linux is very much doable. So I switched to a role that was about working on Python and Docker.

Then, for various reasons, I switched jobs often between November 2014 and October 2015.

Unfortunately, I never worked on a role/project requiring me to scale things. OK, I did once. But that project got tangled in non-technical issues happening a few levels above my paygrade. Meaning, I never developed a mindset to think of systems at scale. Pretty much still haven't.

None of my jobs required deep, or even good, understanding of data structures and algorithms. While interviewing, I got rejected at many companies due to the lack of this knowledge. The first time it happened, I diligently spent a couple of months doing a Coursera course on the topic before giving up because I couldn't get past a few assignments/topics. When more such rejections started happening, I started despising companies asking such knowledge.

In spite of scoring great during K-12 years, I was no gold medalist or straight A student at the undergraduate college. Grades don't reflect knowledge, I know, but I sucked and still do at various Computer Science concepts like OS, compilers, networks, and databases. In fact, I have never worked with databases! Only a few months ago did I genuinely start feeling that I want to learn these concepts better. And only for the sake of being a better engineer. An engineer I could respect myself.

### Clear reproducer steps

Bugs usually have clear steps to reproduce them. This is not always true on upstream projects, but, in my work it is, because stuff lands on my plate after being worked upon by at least one or more persons. As a result of clear steps to reproduce the issue, I can quickly spin up a development Kubernetes cluster and debug the particular component by starting from an IDE. This helps me step through the code and understand the flow and architecture of the component and the project I'm working on.

Now, it might sound like fixing bug is easy, but it's not so even though, at times, the source of the bug is clear because in a system like Kubernetes, there are numerous components at play and no situation is ever just "0 or 1".

## So what about doing feature development?

Obviously I like working on feature development. Love it, in fact! I have done that numerous times in my previous roles. Features that were big in the context of the project, important for the advancement of the project, and an immediate hit among the developers who were workng on the team!

But my focus has currently shifted to not bother so much about whether I'm working on a bug or a feature. It has moved to becoming a better engineer that I described above. During Diwali 2024, I joined a [virtual book club](https://x.com/AkJn99/status/1842866790747218057) that was reading a book I had long wanted to read. I have made pretty good progress and absolutely loved the book. Next up, I intend to get better at data structures, algorithms, and databases. Beyond that I'll most likely start diving in distributed systems through [this highly recommended book](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/). The [teachyourselfcs website](https://teachyourselfcs.com/) has laid out all the resources required to get good at CS fundamentals.


## Conclusion

While people are vibe coding and talking about AI eating up programming jobs, I have committed time to learning the fundamentals of my domain. At the end of the day, I want to get closer to the systems by peeling off one layer of abstraction at a time till I reach the [kernel](https://kernel.org) layer.