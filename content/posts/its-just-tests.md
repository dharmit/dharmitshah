+++ 
date = 2025-04-02
title = "What's the big deal in writing tests?"
description = "Intense work that happens behind what looks like a small code change"
tags = ["go", "testing", "programming"]
+++

This post is about the work that went into coming up with this [pull request](https://github.com/rancher/cluster-api-provider-rke2/pull/611). Specifically the e2e tests written for it. The code change is pretty small — just a `for` loop. Tests make up rest of the change introduced in there! Initially, I wanted to add unit tests for it, but there were none already existing, and I didn't want to do it afresh as I didn't have enough bandwidth — had to jump onto working with a different team. However, a core member of the team maintaining that repo suggested adding a check in the existing e2e tests so that it doesn't go entirely untested. The request sounded reasonable and, hence, I got started navigating the tests.

My relationship with writing tests is a little weird. The thing is, I genuinely like writing tests because I believe that they are essential. In fact, I was once appreciated by our test team's lead in a meeting for writing tests that didn't just validate the happy path but failures as well. At the same time, adding tests also makes me feel nervous because I feel like I don't understand the entry point of tests and overall testing framework to be able to efficiently contribute to it. It also varies from one repo to another.

But nowadays we have AI systems to help with. I had a quick and short interaction with Claude AI to understand how to go about adding tests. Since the repo uses the popular [ginkgo](https://onsi.github.io/ginkgo/) framework, it made things a little easier as I have worked with it in the past — never from scratch, always added tests as per a PR's needs. But in the repositories I have worked with so far, it used to be easy to debug the tests using the IDE. Not so this time. Claude quickly helped me learn that the function `TestE2E` would be the place where tests start. But I was unable to figure out a way to step through the code and add some of my own.

That's when my colleague told me about the `test-e2e-local` target in the repo's Makefile that might help run stuff locally. Even he wasn't 100% sure as he had only recently started looking at this repo's tests. Now, I knew how to trigger focussed tests so I changed one of the calls from `It` to `FIt`. And the way I picked that call was by finding the `It` call that _seemed_ to be least intensive, and hence quickest, to run locally. Basically, I went with a hunch and a well-educated guess.

Running the particular make target spitted out a lot of logs and I started by searching them within the test files. That's how I was able to form some mental model of how the tests were organized/running. The first thing I figured was specific YAML manifests in the `test` directory to make changes to.

The `grep`'ing of log messages helped me understand that a lot of the work is happening with the help of code in `helpers.go`. The filename suddenly started sounding obvious. In there I found the function which was responsible for checking when the `Machines` created by `RKE2ControlPlane` start to exist in the cluster. I made modifications there and added bunch of `fmt.Println` debug messages, and voila things worked as expected.

### That's it

I actually felt proud of being able to figure things out and add the test that was working as expected. For all the impostor syndrome issues I face on a regular basis, this one actually made me feel good about myself!