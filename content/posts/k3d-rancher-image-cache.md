+++ 
date = 2024-11-07
title = "Caching container images for k3d"
tags = ["kubernetes", "rancher", "k3d"]
+++

Since starting to work at [SUSE](https://suse.com), I have been working on
[Rancher](https://github.com/rancher) and its components. Having a locally
deployed Rancher setup helps. Initially it was a struggle till I stumbled upon
[this article](https://medium.com/47billion/playing-with-kubernetes-using-k3d-and-rancher-78126d341d23)
that does a great job of explaining how to set things up.

Recently, however, I had to `helm uninstall` and `helm install` the Rancher
chart a few times. Every `helm install` led to the container image being pulled
again from the registry. Now, the image is a good 2.1 GB in size causing me to
wait till things get started. So I started searching for options to cache image
locally.  I remember minikube has an option to cache images and assumed k3d
would have a similar feature too. However, it doesn't.

In the process of reading k3d's CLI help messages, I found that it has a `k3d
registry` option. Reading further I figured that it's possible to download an
image, tag+push it to the registry created using `k3d registry`, and finally
use this registry in the future k3d cluster one brings up.

### Create a registry

Let's first create a registry using k3d:

```sh
# starts a container using the "registry:2" image
$ k3d registry create rancher -p 0.0.0.0:5000

$ k3d registry list
NAME          ROLE       CLUSTER   STATUS
k3d-rancher   registry             running
```

Add an entry into `/etc/hosts` file so that DNS resolutions don't break:

```sh
# make an entry for the registry in /etc/hosts
$ sudo sh -c "echo '127.0.0.1   k3d-rancher' >> /etc/hosts"
```

### Create a cluster that uses our registry

Now, spin up a Kubernetes cluster using k3d and ensure that it can pull images from the registry we just created:

```sh
$ k3d cluster create k8s \
 --registry-use rancher \
 --volume kubelet.config:/etc/rancher/k3s/kubelet.config \
 --volume config.yaml:/etc/rancher/k3s/config.yaml
```

### That's it

To use the Rancher image on our local registry, use the following helm command:

```sh
$ helm install rancher rancher-latest/rancher \
--namespace cattle-system \
--set bootstrapPassword=<some-password> \
--set hostname=<some-hostname> \
--set rancherImage=k3d-rancher:5000/rancher \
--set rancherImageTag=v2.9.2
```

Above command will install a chart using the image `k3d-rancher:5000/rancher:v2.9.2`. Make sure it's available on your local k3d registry.