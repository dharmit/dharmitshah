+++
title = "Centos Container Pipeline OpenShift Upgrade Part 1 - 'The Plan'"
date = 2018-12-19T15:32:03+05:30
type = "post"
tags = ["openshift", "automation", "ansible", "centos_container_pipeline", "containers", "openshift_upgrade_2018"]
drafs="true"
+++

This blog post and others in the series are going to be about the OpenShift
cluster upgrade that our team performed for CentOS Container Pipeline
during December 2018. To find all the blogs on the topic, check the
[`openshift_upgrade_2018
`](https://dharmitshah.com/tags/openshift_upgrade_2018) tag.

OpenShift Origin was renamed to OKD (Origin Community Distribution of
Kubernetes) so I'll be using the term OKD in this and future posts to refer to
OpenShift Origin.

### What we had

We had a stable and functional OKD cluster running on OKD 3.9 that was being
used as the base for CentOS Container Pipeline service. Having things stable at
infrastructure level was still a new thing for us. Before we [revamped the
CentOS Container
Pipeline](https://blog.centos.org/2018/10/revamp-centos-community-container-pipeline-to-run-on-openshift/)
to run on top of OKD, things were quite shaky. We used to keep firefighting
most of the time because the heterogeneous cluster was far from stable!

We had setup Prometheus and Grafana with the help of openshift-ansible
playbooks. We also had a custom Grafana dashboard to monitor the number of
builds (both successful and failed), the number of builds in different OKD
stages like New, Pending, Running, Complete and Failed, and a graphical
representation of the amount of free memory on the nodes in OKD cluster.

### What we wanted

We wanted more monitoring and alerting. In the sense, we wanted to be notified
when some build is taking a long time, or when there are more than *x* failed
builds in a short duration, or if there were more than *x* builds hanging around
in Pending state, so on and so forth. The intention was to be able to stay on
top of things unlike how screwed up they were in earlier architecture where
many times it was the users who notified us about something blowing up in our
environment while we were busy writing new bugs (read 'features'). :wink:

### The Plan

As mentioned earlier, we were already using Prometheus for monitoring and had
to configure Alert Manager to add rules based on which it would send us alerts.
But instead of doing that, we stumbled upon the fact that OKD 3.11 was going to
ship with built-in [Prometheus Cluster
Monitoring](https://docs.okd.io/3.11/install_config/prometheus_cluster_monitoring.html)
and we thought to ourselves, let's use this instead of writing our own rules.

To be honest, the assumption was that, in OKD 3.11, Prometheus would have some
default set of rules which would make our job easier. Now making assumptions is
probably never a good idea in software engineering. But we still managed to do
that. 

We decided that instead of writing rules for the Prometheus setup that we
have in OKD 3.9, we would upgrade our cluster to OKD 3.11 and see what's
available by default and add rules on top of it. Of course, we didn't know at
the time that a better idea would have been to RTFM the upcoming shiny thing
called as Prometheus Cluster Monitoring instead of diving into the task of
upgrading the cluster.

### Conclusion

In this post, I mentioned what we wanted to achieve and what we thought would
help us do that and what we assumed in process. In future posts, I'll mention
what we did, how we messed up and how we made sure that the taste of stability
was taken away. To complement that, we screwed the production on a Friday when
the production Gods are not in a mood to be disturbed.

Until next time... :smile:
