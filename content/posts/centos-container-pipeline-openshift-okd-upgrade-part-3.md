+++
title ="Centos Container Pipeline OpenShift Upgrade Part 3 - 'Ready for Production!'"
date = 2018-12-27T11:15:45+05:30
tags = ["openshift", "automation", "ansible", "centos_container_pipeline", "containers", "openshift_upgrade_2018"]
+++

This blog is a part of the [series of
blogs](../../../tags/openshift_upgrade_2018) on OpenShift OKD upgrade we
performed for the CentOS Container Pipeline project. If you haven't read the
[first
part](../../../2018/12/centos-container-pipeline-openshift-okd-upgrade-part-1)
and the [second
part](../../../2018/12/centos-container-pipeline-openshift-okd-upgrade-part-2),
this post might not make a lot of sense. Go ahead, read them first.

I ended the previous post with describing the unconventional path we
unwillingly decided to go with and how that led us to figure the cause of
the issue we were facing while following recommended path for OKD 3.9 to 3.10
upgrade.

### Uninstall, install, repeat

As mentioned in the previous post, the idea was to:

- uninstall OKD 3.9,
- install OKD 3.11,
- ensure that PV used for Jenkins in 3.9 setup was usable with 3.11 as well
  without data loss

Just like OKD 3.10, OKD 3.11 also uses `openshift_node_groups` dictionary in
place of `openshift_node_labels` and takes the configuration for kubelet
arguments with it.

While installing OKD 3.11, we encountered an issue wherein Ansible playbook
would exit with an error related to OpenShift master. The error was:

```bash
{"changed": false, "msg": "Node start failed."}
```

This kept on repeating every time we tried to install OKD 3.11 on the nodes
where OKD 3.9 was earlier installed.

### Fixed two issues in one shot

We opened an [issue on
GitHub](https://github.com/openshift/openshift-ansible/issues/10774) for the
installation error. Unlike the [failure with OKD 3.10
upgrade](https://github.com/openshift/openshift-ansible/issues/10690), this
time the  logs were more helpful in making sense of the underlying issue.

It seemed like we made a mistake in configuring the value for
`openshift_node_groups` variable. My takeaway from the [way we fixed the
issue](https://github.com/openshift/openshift-ansible/issues/10774#issuecomment-442375416)
is that the value in the dictionary has to be a list even if it's just one
value and not multiple values.

After this we faced another error that had to do with the value we had set for
the key `kubeletArguments.image-gc-high-threshold` in `openshift_node_groups`.
This is something I found from the logs. After trying few values that I thought
might help `origin-node` service to start well, I gave up and removed every
configuration related to [garbage
collector](https://docs.okd.io/3.11/admin_guide/garbage_collection.html) from
the `openshift_node_groups` variable.

This not only helped us perform a successful installation of OKD 3.11, but
also turned on a bulb in my head that illuminated something like, "Remove the
garbage collector configuration and try OKD 3.9 to OKD 3.10 upgrade the
recommended way!" And so I did.

### Successful OKD 3.9 to OKD 3.10 upgrade

It turned out that the bulb illuminated the right thing. Apart from facing a
few minor issues (minor because we didn't need to open a GitHub issue), we
managed to upgrade our OKD 3.9 cluster OKD 3.10 using the OpenShift Ansible
playbooks! It was a "F**k yeah" moment for me.

Over next few days, we managed to successfully perform OKD 3.9 to OKD 3.10
upgrade in the development environment before we decided to replicate the steps
in production.

### That's it!

Since the recommended upgrade path worked well for us, we didn't bother to test
if doing uninstall and install of different versions of OKD and then using the
same PV for a persistent Jenkins deployment worked well or not. We felt
ecstatic with the way things worked out. Little did we know how the Jenkins
thing was going to be one among several dogs that were planning on biting us!

By the time we reached this point, it was more than 3 weeks since we started
off. I visited the [Serengeti National
Park](https://en.wikipedia.org/wiki/Serengeti_National_Park) during this
timeframe and if we count the vacation, it was more than 4 weeks since we
started.

Until next time... :smile:
