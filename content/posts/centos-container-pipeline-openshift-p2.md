+++
title = "Restructuring CentOS Container Pipeline using OpenShift - Part 2"
date = 2018-09-11T18:56:33+05:30
tags = ["centos_container_pipeline"]
+++

If you have not read the [first
part](https://dharmitshah.com/2018/07/centos-container-pipeline-openshift-p1/)
of this series on CentOS Container Pipeline service then please go ahead and do
it!

In this post, I'll be talking about the issues we faced and how we resolved
them when we tried to build a large sized
[container-index](https://github.com/CentOS/container-index/) with the newly
developed service.

### What we did?

One fine morning I had a thought of trying to build container images for all
the projects that build their images with our service already. But I wanted to
do it with the new deployment of the service which is based on OpenShift.

First things first, I [modified the
container-index](https://github.com/dharmit/container-index/commit/6b8e9faedb15dbb5aeeb2d882b3274b1cebbfa12)
to make sure that notification emails don't get sent out to the actual users
but land in my inbox instead. Otherwise it would have been one hell of a
testing for me! :wink:

I had a 3-node cluster (1 master and 2 nodes) which I was going to test things
against. So far, we had tested things only on a small sized index of maybe 5-10
images. Obviously that was a mistake and we should have done this testing of
full container-index sooner rather than when we're so close to deploying things
into production.

### What we observed?

When I triggered the full index to be built on my deployment of the service, I
saw that the seed-job that parses all the yaml entries in the container-index
picked up the jobs and started parsing them. At the same time, the jobs that it
had already parsed started getting served by OpenShift and Jenkins, i.e.,
builds were triggered.

Some of the issues we observed:

- Numerous jobs were failing due to unavailability of Docker daemon socket to
  build the image.
- Many images were taking hours to build completely instead of taking a few
  minutes.
- Seed job failed to parse a particular entry and in event failed to parse the
  remaining index.
- Jenkins used to get restarted and thus number of pods were left orphaned
  while Jenkins console reported that it cannot connect to the pods.
- Scanners failed to scan certain images.
- Weekly scan doesn't run

### How we fixed the issue?

#### Docker daemon issue
It turned out that OpenShift template used to spin up seed-job and build jobs
(jobs that would build container image for parsed entries) had same name. The
[pod template for
seed-job](https://github.com/CentOS/container-pipeline-service/blob/3ec074a62840540359ea9c307ff5ae876e7b2703/seed-job/buildtemplate.yaml#L26-L42)
didn't have Docker socket configuration because it doesn't need the Docker
daemon for anything. So the build jobs triggered while seed-job was still under
execution used the same pod template which didn't have Docker socket available.

Figuring out the issue took an embarassingly long time and was finally fixed by
[this PR](https://github.com/CentOS/container-pipeline-service/pull/622/).

#### Builds taking hours to complete
This was happening as a result of 200+ build requests coming in in quick
successcion. During development, we tested with a small index and this never
became an issue.

To fix this issue, we introduced batch processing of the jobs. With this in
place, we default to processing only five jobs at a time, poll the OpenShift
API every 30 seconds to see how many jobs are still new/pending/building and
process the next batch if there are 2 or less projects being processed.

The benefits of batch processing from what I observed are:
- Quicker build times
- Abiliity to larger container-index even on a single node OpenShift cluster
  (like [minishift](https://minishift.io)). This is not tested yet but I tried
  to build entire index on just two node cluster things worked just fine!

Pull request for batch-processing is available
[here](https://github.com/CentOS/container-pipeline-service/pull/612).

#### Seed job failing to parse particular entry
This issue has to do with OpenShift's rules for naming the builds. You can't
have an upper-case letter in the build's name. So we had to make sure that all
build jobs being created were converted to lower case before creating them on
OpenShift.

PR for this is available
[here](https://github.com/CentOS/container-pipeline-service/pull/617).

#### Jenkins restart issue
This was relatively simple to fix. We provided more memory to the Jenkins
deployment and things started working fine. However, with batch processing in
place, I think we can even do with less amount of memory because there's not
going to be a *lot* of builds being processed at the same time.

One thing I learned (thanks [Pradeeptp](https://twitter.com/pradeepto/)) was
that you can't increase the number of pods of Jenkins to address this issue
because Jenkins does not scale well horizontally. Increasing its compute
resources (CPU/Memory) is the best way to mitigate issues related to it.

#### Scanners failing for certain images
This issue showed up because of more than one reasons:
- We parsed the output of commands like `yum check-update` from `stdout`. When
  the output for single package spanned over multiple lines, things broke.
- CentOS 6 and CentOS 7 have different ways to check for updates. One uses `yum
  check-update` and other uses `yum check-updates`.
- We use `/bin/python` as an entrypoint for the scanners when they run scripts
  inside the image under test but using `/usr/bin/python` ensures that things
  are compatible across both CentOS 6 & CentOS 7

These issues are being fixed by [this
PR](https://github.com/CentOS/container-pipeline-service/pull/626).

#### Weekly scan failures
We found that weekly scans were not getting triggered by Jenkins at all. Now
the weekly scan jobs are configured as Jenkins Pipeline which are to be run as
cron on weekend. But Jenkins doesn't run them unless they are already triggered
once. There's a JIRA ticket for this issue that I can't find right now. It's
closed with "Won't fix".

So we came up with another approach which is using the OpenShift cron to run a
script that triggers build for all such weekly scan jobs. PR for this is
available
[here](https://github.com/CentOS/container-pipeline-service/pull/615).

### What's next?

At the moment, we are busy fixing this issues we discovered by trying to build
entire container index on the cluster. We have fixed most of them and have PRs
open for remaining ones.

Next, we're going to get the email notification piece working and deploy things
to a more production like environment before finally moving things to
production.

### That's it!

If you know of better solution/approach to fix aforementioned problems we
faced, please let us know. You can catch us on #centos-devel IRC channel on the
Freenode server. I hang out on the channel as "dharmit".

Until next time... :wink:
