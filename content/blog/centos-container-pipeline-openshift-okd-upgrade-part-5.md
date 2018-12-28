+++
title = "Centos Container Pipeline OpenShift Upgrade Part 5 - 'The Lessons'"
date = 2018-12-28T14:49:04+05:30
type = "post"
tags = ["openshift", "automation", "ansible", "centos_container_pipeline", "containers", "openshift_upgrade_2018"]
+++

This blog is the last part in the [series of
blogs](../../../tags/openshift_upgrade_2018) on OpenShift OKD upgrade we
performed for the CentOS Container Pipeline project. If you haven't read the
earlier parts, go ahead and do that first.

If you have already read the previous parts in the series, it's easy to
conclude that we screwed up. We thought we had broken out of that habit after
[moving to the new
architecture](https://blog.centos.org/2018/10/revamp-centos-community-container-pipeline-to-run-on-openshift/).
After all, we had a stable environment for two and half months and only issues
we faced were from our buggy code. But bad habits are difficult to get rid of!

Obviously, we learned a few lessons in the process. Few too many!

### Lesson 0 - Friday is for beers

This one doesn't need any detail. Unless it's a major bug fix or security
patch, Friday should be meant for beer, or coffee, or tea, or \<put your
favorite drink here\>.

### Lesson 1 - OpenShift knowhow needs to be improved

We need to improve our knowledge about OpenShift works. RTFM is not only
required but we need to make more time to do it. I'm not sure how many problems
we could have avoided had we read more documentation but, we could have surely
made informed decisions when we faced them.

### Lesson 2 - Homogeneous production and testing environment

Every bit of testing we did was on an infrastructure which we had absolute
control over right from spinning up the VMs to deploying our service in the
OpenShift cluster. However, the production environment is something that CentOS
infrastructure team provides and there are some differences between the two
because the errors we saw in production environment never showed up while
performing same actions on the test environment.

We need start using a small subset of systems provided by CentOS infrastructure
team for pre-production testing and validation.

### Lessons 3 - Have reasonable expectations from the upstream

The OpenShift upstream is possibly loaded with a bunch of questions and issues
from users like our project. It would be too much to expect them to look into
every issue beyond "best effort" basis. This takes us back to Lesson 1.

### Lesson 4 - Talk to the experts

Before making a cluster level change, it might be a good idea to write down the
plan and the steps, and run it by a few experts to get an opinion or maybe even
a better idea to do what we're trying to do.

### That's it!

That's all I have to share about the fiasco we did during the weekend of
December 14, 2018. If you have any suggestions/ideas/feedback for us, let us
know. I know there's a lot we can and we need to learn about the various tools
involved in this exercise.

On a personal note, this is the most I've ever written in a given week. If you
read it this far, I hope I managed to make you think - "This guy is a
professional screw-it-up artist".

Until next time... :smile:
