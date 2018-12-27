+++
title = "Centos Container Pipeline OpenShift Upgrade Part 2 - 'The Steps'"
date = 2018-12-26T13:24:42+05:30
type = "post"
tags = ["openshift", "automation", "ansible", "centos_container_pipeline", "containers", "openshift_upgrade_2018"]
+++

This blog is a part of the [series of
blogs](../../../tags/openshift_upgrade_2018) on OpenShift OKD upgrade we
performed for the CentOS Container Pipeline project. If you haven't read the
[first
part](../../../2018/12/centos-container-pipeline-openshift-okd-upgrade-part-1),
this post might not make a lot of sense. Go ahead, read that first.

In the second installment of the series, I'm going to describe the steps we
divided our plan into. Or rather, the steps we had to break things into because
upgrading from OKD 3.9 to OKD 3.11 was not possible without upgrading to OKD
3.10 first.

**Note**: The steps mentioned in this post were executed on a
development/pre-production environment that lives on Red Hat infrastructure and
not on CentOS infrastructure where the production environment lives.

### OKD 3.9 to OKD 3.11 via OKD 3.10

[Upgrade
prerequisites](https://docs.okd.io/3.11/upgrading/automated_upgrades.html#upgrade-prerequisites)
section in OKD 3.11 in-place cluster upgrade documentation says that an
existing OKD cluster needs to be upgraded to "latest asynchronous release of
version 3.10" before attempting an upgrade to OKD 3.11. You can go ahead and
search for those exact words in the quotes on the docs page to be sure that I'm
not lying! 


### OKD 3.9 to OKD 3.10 upgrade

Having successfully upgraded from OKD 3.7.2 to OKD 3.9, I was confident that
upgrading OKD 3.9 to OKD 3.10 would be as simple as a walk in the park. But
just as life has its ups and downs at most unexpected times, OKD had plans of
making us go down a rabbit hole when we least expected it to. With OKD 3.10:

- the `openshift_node_labels` got deprecated,
- having a proper DNS was mandatory,
- the `openshift_node_kubelet_args` got deprecated

### DNS setup in development environment

Even before trying to perform an upgrade, we had to configure proper DNS in our
development environment.

Our development environment, running in Red Hat infrastructure, had OpenShift
OKD cluster running as VMs on top of KVM and libvirt.

Till OKD 3.9, using IP addresses as hostnames in Ansible inventory file was
just fine. But with OKD 3.10, it was mandatory to use hostnames that would
resolve through a DNS server. Even `/etc/hosts` entries wouldn't be considered
valid unless a proper DNS was set up.

Our production environment didn't have to be altered because DNS worked fine
there but in our development environment, we had to set up a DNS server. [This
blog
post](https://liquidat.wordpress.com/2017/03/03/howto-automated-dns-resolution-for-kvmlibvirt-guests-with-a-local-domain/)
by a fellow Red Hatter came to the rescue!

### First attempt at upgrade

After having grasped the changes required to make to the Ansible inventory file
work well with OpenShift Ansible playbooks for OKD 3.10, we attempted the
upgrade. The initial attempt was futile and we faced error due to a [typo in
the variable
name](https://github.com/openshift/openshift-ansible/issues/10502).

### Serious errors during the upgrade

After the typo issue, next error we faced was more serious. Serious because it
didn't seem like upstream had any clue about a fix either. We opened an [issue
on GitHub](https://github.com/openshift/openshift-ansible/issues/10690) and
added a bunch of information that we found while troubleshooting the error by
ourselves.

Logs complained about `No existing node-config.yaml, exiting` in spite of
having the `/etc/origin/node/node-config.yaml` in place. The issue, as we later
understood, was a misconfiguration of kubelet arguments in the new way of doing
things with OKD 3.10.

But this one took a really long time because the error message didn't seem to
have the correct indication of the underlying issue and, as you can see from
the GitHub issue, we tried a number of things in absence of help from upstream.

This issue taught us our first and one of the most important lessons - "We lack
OpenShift expertise within our team."

It's one thing to use OpenShift and another to know it inside-out. However,
it's not an easy thing to fix due to the huge size and fast pace of development
of Kubernetes, and hence that of OpenShift.

### The unconventional path

Due to the above error, we faced during OKD 3.9 to OKD 3.10 upgrade, we started
to lean towards doing something that's not mentioned in the documentation and
quite possibly the least favorable way to upgrade. We planned to:

- uninstall OKD 3.9 from the cluster
- install OKD 3.11 on the same cluster
- preserve the Jenkins build data on the PV so that when we attach the same PV
  to Jenkins on OKD 3.11, we can possibly resume from where we left

### That's it

We managed to do some crazy stuff while going down the "unconventional" path
mentioned above. But the craziest thing was to find a solution to doing an OKD
3.9 to OKD 3.10 upgrade in a clean way. That is, we managed to find a solution
and the cause of the issue we faced while following the recommended path for
OKD 3.9 to 3.10 upgrade.

Until next time... :smile:
