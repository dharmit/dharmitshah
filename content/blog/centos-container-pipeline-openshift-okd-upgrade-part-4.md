+++
title = "Centos Container Pipeline OpenShift Upgrade Part 4 - 'Fire in the Hole'"
date = 2018-12-27T14:37:22+05:30
type = "post"
tags = ["openshift", "automation", "ansible", "centos_container_pipeline", "containers", "openshift_upgrade_2018"]
+++

This blog is fourth part in the [series of
blogs](../../../tags/openshift_upgrade_2018) on OpenShift OKD upgrade we
performed for the CentOS Container Pipeline project. If you haven't read the
earlier parts, go ahead and do that first.

I closed the previous post on a high wherein we managed to fix multiple issues
that were preventing us from doing the OKD upgrade in production. After
reaching the stage, we decided to do OKD 3.9 to OKD 3.10 upgrade and then
upgrade to OKD 3.11. In the development environment, I already attempted an OKD
3.10 to OKD 3.11 upgrade. It was a walk in the park just like the experience I
had while upgrading from OKD 3.7.2 to OKD 3.9.0.

### Mistake #0 - Choosing Friday for production upgrade

This statement is intentionally written to not make any sense; just like our
decision to do production upgrade on Friday. I don't think any amount of
penance can free us from this sin of disturbing production Gods on a Friday.
Amen.

### Song of Error on loop mode

We had taken a maintenance window of about 4 hours and were confident of
finishing our job in under 2 hours. But there won't be any fun if things went
as per the plan. As if a song of error was playing on loop mode, we were
repeatedly seeing the same error and not able to make any sense of what's going
wrong.

```bash
TASK [Read node config]
**************************************************************************************************************************************
ok: [osn1.example.com]                                                                                                                          
fatal: [osm1.example.com]: FAILED! => {"changed": false, "msg": "file not
found: /etc/origin/node/node-config.yaml"}
ok: [osn2.example.com]
ok: [osn3.example.com]
ok: [osn4.example.com]
ok: [osn5.example.com]
ok: [osn6.example.com]
ok: [osn7.example.com]
ok: [osn8.example.com]
ok: [osn9.example.com]
ok: [osn10.example.com]

```

We were seeing this error for the master. And every time we checked the master
after OpenShift Ansible threw above error, this file
`/etc/origin/node/node-config.yaml` was undoubtedly present! The logs were not
particularly helpful in pointing out the real cause of the problem. Above all,
this was the first time we were seeing this error. Our development environment
didn't throw this error for OKD 3.11.

Now, if you have read previous posts in this series, it's not the first time
I'm saying that logs didn't make a lot of sense. This, probably, boils down to
the point I mentioned in an earlier post about the first and one of the most
important lessons this exercise taught us - "We lack OpenShift expertise within
our team."

### Uninstall OKD 3.9, install OKD 3.11

After struggling with the plan of upgrading to OKD 3.10 in a recommended
manner, we decided to go with the unconventional path in production because
with every passing minute during which our production was a mess, we were not
doing what our service was supposed to do - build container images for the
open-source projects.

### Jenkins PV did bite us

After bringing up OKD 3.11 in production, we tried to attach the same PV to
Jenkins that was being used in OKD 3.9 cluster. To our surprise, the PV got
attached immediately and Jenkins started up. This was something we struggled to
accomplish in the development environment where PV wouldn't get attached unless
we modified the permissions or deleted the data on PV. And once the 3.9 to 3.10
upgrade worked fine, we stopped thinking about Jenkins PV altogether.

In production, however, first thing Jenkins did after connecting with the PV
was to delete the data from previous builds jobs. Not sure how that was any
better than us formatting the PV ourselves!

### etcd wanted its share of biting us as well

The production environment was running on top of OKD 3.11 while development
environment was on OKD 3.10. We didn't have much idea of how things were with
OKD 3.11. And that fact was very well exploited by `etcd`.

[`etcd` restarts when a particular version of
`docker`](https://github.com/openshift/origin/issues/21609) is used which
caused API service to be unavailable for some time. The `seed-job` in CentOS
Container Pipeline service is a build job that:

- runs on Jenkins,
- parses the [container index](https://github.com/CentOS/container-index/),
- creates/updates jobs on Jenkins
    - triggers build for newly created jobs
    - silently updates existing jobs (if there is any change)

It uses `oc` command line tool to do its job. Now while the `etcd` restarts
caused API server to be unavailable, our `seed-job` kept doing its job unaware
of the situation and the `oc` commands during this time were pretty much
talking to `/dev/null` (metaphorically). The `seed-job` needs to be altered to
consider this scenario. We faced this issue with `etcd` in spite of using a
[version of `docker`
newer](https://github.com/openshift/origin/issues/21609#issuecomment-447537305)
than the one mentioned in errata.

### Unwarranted emails sent to users

We didn't realize that we were hitting the `etcd` issue until it was a bit too
late. We formatted Jenkins PV once after realizing that Jenkins had anyway
deleted all the data that might have been of any use. After that, we triggered
the `seed-job` to parse my fork of container index so that it didn't send
emails to the users. Things seemed to be under control so, we switched the
`seed-job` to refer to main container index. But the jobs that were not created
during the earlier run of `seed-job` due to `etcd` restarts got created in the
subsequent run and few users were notified with an email that said "First build
of the container image" as the cause of build.

### That's it!

Wow, this one turned out to be longer than expected. Overall, we had a crazy
weekend (yes we were now working on a Saturday). We failed to send weekly scan
emails as well. The taste of stability, that we got after moving to OpenShift
based infrastructure in late September, was swiftly taken away by this
incident. It taught us a number of lessons in the process.

Next and the last post in this series is going to be a quick gist of the
lessons we learned during this exercise.

Until next time... :smile:
