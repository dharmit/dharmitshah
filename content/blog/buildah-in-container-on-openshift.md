+++
title = "Using Buildah to build container images on Openshift"                           
date = 2018-12-20T16:24:41+05:30
type = "post"
tags = ["containers", "openshift", "centos_container_pipeline", "buildah", "podman"]
+++

In CentOS Container Pipeline, we use [Jenkinsfile
strategy](https://github.com/CentOS/container-pipeline-service/blob/562b8a41bb21da9da1e84df5fa957aad475d9a84/seed-job/template.yaml#L16-L175)
of OpenShift to build container images, scan them and push them to the
registry. To build a container image, we need to make sure that the pod started
by Jenkins is privileged (akin to `docker run --privileged`) and that we're
sharing the Docker daemon socket (`/var/run/docker.sock`) from the host with
the container. Anyone with security in mind would dislike the idea (and I hope
[Dan Walsh](https://twitter.com/rhatdan) doesn't read how insecurely we've
plumbed certain things here.)

After having stabilized the service on its new architecture, we wanted to
explore if we can use `buildah` to build the containers and eliminate the need
of sharing Docker socket with all the pods that spin up on an OpenShift node
(we also [managed to blow up the service after stabilizing
it](https://dharmitshah.com/2018/12/centos-container-pipeline-openshift-okd-upgrade-part-1/)
but, that was not intentional.)

### Challenge #1 - Running buildah inside a container

Since we use OpenShift to build images and Jenkinsfile strategy to do things
other than just building them, we had to make sure that image building can be
done inside the pod that's dynamically brought up by Jenkins server on the
OpenShift cluster. To be able to do this, we had to use a [newer version of
`buildah` than one provided by CentOS
repos](https://github.com/containers/buildah/issues/933#issuecomment-439329352).

### Challenge #2 - Go package is only available via SCL in RHEL/CentOS

CentOS repos come with an outdated version of `buildah` so we had to build the
`buildah` binary. To build it inside the container was non-trivial (at least
for me) and it took me some time to finally come up with a [script to do
it](https://github.com/dharmit/container-pipeline-service/blob/880f9f54eb9cacd2346a8cee4597bc19848e68a4/Dockerfiles/ccp-openshift-slave/build_buildah.sh).

### Challenge #3 - Buildah still failed with `ERRO[0000] 'overlay' is not supported over overlayfs`

In spite of managing to overcome first two challenges, we were hitting an error
with `overlayfs`:

```bash
$ buildah images
ERRO[0000] 'overlay' is not supported over overlayfs    
ERRO[0000] 'overlay' is not supported over overlayfs    
'overlay' is not supported over overlayfs: backing file system is unsupported for this graph driver
'overlay' is not supported over overlayfs: backing file system is unsupported for this graph driver
```
The error was fixed by using a tip from [this GitHub
comment](https://github.com/containers/buildah/issues/158#issuecomment-396309669).

### It works!

I'm sure that the solutions/workarounds that I have mentioned above are not
perfect. But that's what we managed to get our issues fixed with and, more
importantly, be able to build container images using `buildah` from inside a
pod/container on OpenShift. Eventually, our [Jenkinsfile was modified to use
`buildah
bud`](https://github.com/dharmit/container-pipeline-service/blob/4054ad7dc3447c59fcd80eb177af2d089e59986d/seed-job/template.yaml#L123)
instead of `docker build`.

### Open challenges

Although we're able to use `buildah bud` to build images, we still need to find
a way to be able to scan the resulting container images. As it is right now,
`docker run` doesn't work to spin up images created using `buildah` unless the
image has been first pushed to an external registry and pulled from there. But
I'm guessing this is because the Docker socket is being shared from the host
system and `buildah` is working within the confines of the container (which is
a good thing, I guess).

So the next thing we're trying to figure out is to use `podman run` instead of
`docker run` to get done with scanning the container image. However, we need to
do this from inside the container and we're already seeing the error reported
in [this GitHub issue](https://github.com/containers/libpod/issues/1534).

### That's it

We are still working on this and are not using `buildah bud` in production. But
we plan to change that soon. If you have any ideas/suggestions that can help us
do things in a better way, let us know! We hangout on #centos-devel IRC
channel.

Until next time... :smile:
