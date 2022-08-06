+++ 
date = 2022-08-06T18:26:02+05:30
title = "Modifying kubelet configuration on OpenShift Local"
tags = ["kubernetes", "crc"]
+++

*This post is more of a self-note so that I don't have to spend hours again trying to solve the same problem*.

I needed to modify the kubelet configuration to test the behaviour of `containerLogMaxSize` feature. I spent hours doing this on minikube with Docker and KVM drivers, but it didn't work. In the logs for kubelet, I kept seeing message like "Docker doesn't support rotation of logs." However, a few weeks back, I had managed to do this on OpenShift Local (formerly CodeReady Containers) just fine, but forgot the steps I followed then to accomplish this. So, this post is a way to refer the steps again if my memory doesn't serve me well.

OpenShift Local uses Podman instead of Docker.

First off, ssh into the OpenShift Local VM using:
```sh
$ ssh -i ~/.crc/machines/crc/id_ecdsa -o StrictHostKeyChecking=no core@`crc ip`
```

Next, escelate to `root` privileges (because I'm too lazy to type `sudo` on a development VM):
```sh
$ sudo -i
```

Modify the file `/etc/kubernetes/kubelet.conf` by appending below lines:
```
containerLogMaxSize: 10Ki
containerLogMaxFiles: 5
```
It makes container logs get rotated every time the size of the file containing the logs exceeds 10KB, and keeps only last five rotated files on the disk. This is purely for development reasons. A 10KB limit will rotate the logs very fast for a container that logs often.

Restart the kubelet service:
```sh
$ systemctl daemon-reload
$ systemctl restart kubelet
```

That's it. That should start kubelet with the additional configuration options we just added. For the complete list of options for kubelet, take a look at its [official docs](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/).