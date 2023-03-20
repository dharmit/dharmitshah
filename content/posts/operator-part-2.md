+++ 
date = 2023-03-20T15:48:24+05:30
title = "Write and publish a Kubernetes Operator - Part 2"
description = ""
tags = ["kubernetes", "operators", "go", "openshift", "olm", "operator-sdk"]
series = ["kubernetes operators"]
+++

This is the second post in the series of posts about writing and publishing a Kubernetes Operator. In this post we 
will cover:
* creating a bundle and packaging it into an index image,
* creating a `CatalogSource` resource using this index image,
* and creating an Operand (or a CR) out of the Operator we thus installed.

If you haven't already, take a look at the [first part](./operator-part-1.md) before reading this further.

{{< notice note >}}
I'm writing this blog series as a part of learning Kubernetes Operators and Operator Framework myself. There might
be mistakes in the post. Please share any feedback/suggestions via [Twitter](https://twitter.com/dharm1t).
{{< /notice >}}

### Install Operator Lifecycle Manager (OLM)

First and foremost, let's enable our minikube cluster with OLM. This can be done using:
```shell
$ operator-sdk olm install
```

This will create a bunch of resources in the cluster mostly in the two newly created namespaces - `olm` and `operators`:
```shell
$ kubectl get ns
NAME              STATUS   AGE
default           Active   3d2h
kube-node-lease   Active   3d2h
kube-public       Active   3d2h
kube-system       Active   3d2h
olm               Active   65s
operators         Active   65s
```

### Bundle Image

An Operator Bundle is a container image that stores the Kubernetes manifests and metadata associated with an 
Operator. A bundle image is built as a scratch container image, meaning it isn't used to spin up a running container.
It is pushed to and pulled from an OCI-compliant container registry. Ultimately, it will be used by an
[Operator Registry](https://github.com/operator-framework/operator-registry) and OLM to install an Operator.

Makefile's `bundle` target generates the bundle into the `bundle` directory. It contains the manifest and metadata 
defining our Operator. When you run it for the first time, it will ask you a bunch of questions that are important 
from Operator metadata point of view. The `bundle-build` and `bundle-push` targets respectively build and push the 
container image containing our bundle. Make sure to login to the container registry first:

```shell
$ docker login -u $QUAY_USERNAME -p $QUAY_PASSWORD quay.io
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /home/dshah/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

$ make bundle bundle-build bundle-push
```
Successful execution of above command creates, in my case, the container image - `quay.
io/dharmit/at-operator-bundle:v0.0.1`

### Using the bundle

Let's use this bundle image (replace the image name as needed) to do the same thing that we did with the help of `make 
deploy` in the previous post:
```shell
$ operator-sdk run bundle quay.io/dharmit/at-operator-bundle:v0.0.1
```

Unlike the `make deploy` command, above command doesn't create a new namespace, but creates various resources like:
* A CatalogSource - `kubectl get catsrc`
* An OperatorGroup - `kubectl get og`
* A Subscription and its InstallPlan - `kubectl get sub,ip`
* A ClusterServiceVersion - `kubectl get csv`

If everything went well, the CSV phase would be "Succeeded":
```shell
$ kubectl get csv
NAME                 DISPLAY       VERSION   REPLACES   PHASE
at-operator.v0.0.1   at-operator   0.0.1                Succeeded
```

We can now create an `At` CustomResource (a.k.a an Operand) using a manifest similar to the one used in the previous 
post. As in the previous post, make sure to use time close to current time in UTC timezone.

### Index Image

An Index Image holds information about one or more bundle images. It is used to create a CatalogSource resource on 
the cluster. OLM uses this CatalogSource to show the user a list of Operators available to install on their 
Kubernetes cluster.

Since we already set the `IMAGE_TAG_BASE` variable in our Makefile in the previous post, we only need to run below 
make target to build and push the index image:

```shell
$ make catalog-build catalog-push
```
Successful execution of above command creates, in my case, the container image - `quay.
io/dharmit/at-operator-catalog:v0.0.1`.

### Create the CatalogSource

Now that our index image is ready, let's create a CatalogSource resource on the cluster. Use below manifest (replace 
the image name appropriately) to create it:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: at-operator-catalog
  namespace: olm
spec:
  displayName: At Operator
  image: quay.io/dharmit/at-operator-catalog:v0.0.1
  grpcPodConfig:
        securityContextConfig: restricted
  sourceType: grpc
  updateStrategy:
    registryPoll:
      interval: 60m
```

Upon creation of CatalogSource you will notice a Pod created in the `olm` namespace whose container is created using 
the `.spec.image` used in CatalogSource above. The logs for the Pod should have the following line:

```shell
time="2023-03-20T14:47:51Z" level=info msg="serving registry" database=/database/index.db port=50051
```

Also a `PackageManifest` corresponding to At Operator:
```shell
$ kubectl get packagemanifest | grep at-operator
at-operator                                At Operator           44m
```
### Create a Subscription

To install an Operator, we need to create a resource of kind Subscription. But before creating a Subscription, we 
must create an OperatorGroup. Here, we're creating them both in the `default` namespace. Since this is a minikube 
cluster, it's not a problem to do so, but on real clusters, it's not recommended to manually create resources in 
`default` namespace:

```shell
$ cat <<EOF | kubectl apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
    name: default-og
    namespace: default
EOF

$ cat <<EOF | kubectl apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
    name: at
    namespace: default
spec:
    source: at-operator-catalog
    sourceNamespace: olm
    name: at-operator
    channel: "alpha"
EOF
```

Upon successful creation of a Subscription, OLM installs the Operator in the namespace:

```shell
$ kubectl get csv
NAME                 DISPLAY       VERSION   REPLACES   PHASE
at-operator.v0.0.1   at-operator   0.0.1                Succeeded
```

### Create a CR

So far, we have created a CR for our Operator after installing it using `make deploy` and `operator-sdk run bundle`. 
None of these approaches are preferred for development purpose only. When a user is going to use an Operator 
developed by you, they are most likely to use it through something like Operator Lifecycle Manager, which enables a 
user to install and consume Operators available from various catalogs.

Now that we have installed "At Operator" from the catalog, let's create the CR and see if it works as expected:

```shell
$ cat <<EOF | kubectl apply -f -
apiVersion: at.example.com/v1alpha1
kind: At
metadata:
  name: sample-at
spec:
  schedule: "2023-03-20T15:44:30Z"
  command: "echo hello world"
EOF

$ kubectl get pods
NAME                                              READY   STATUS      RESTARTS   AGE
at-operator-controller-manager-85b5457556-rrspl   2/2     Running     0          9m14s
sample-at                                         0/1     Completed   0          39s

$ kubectl logs sample-at
hello world
```

As we can see, a CR was successfully created and the command we specified was executed at the time mentioned in the 
schedule.

Everything works!

### That's it

That's it for this blog series. We saw how to create an Operator, package it into a Catalog, and use it the way 
regular users would consume Operators available from places like [OperatorHub.io](https://operatorhub.io).