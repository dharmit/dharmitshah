+++ 
date = 2023-03-17T12:46:52+05:30
title = "Write and publish a Kubernetes Operator - Part 1"
description = ""
tags = ["kubernetes", "operators", "go", "openshift", "olm", "operator-sdk"]
series = ["kubernetes operators"]
+++

This is first in the series of posts about writing and publishing a Kubernetes Operator. This series will discuss the 
following topcis:
* how to write an Operator,
* run it on a Kubernetes cluster,
* create a bundle and package it into an index image,
* create a `CatalogSource` resource using this index image, 
* and finally create an Operand out of the Operator installed through the `CatalogSource`.

This particular post covers first two, while the next post wil discuss remaining topcis.

Tooling used by me for this blog series:

* OS: Fedora 37
* Kubernetes: minikube v1.29.0 with Docker provider
* OLM specific tooling:
  * `operator-sdk` version 1.27.0
  * `opm` version 1.26.4

The Operator we are going to create here isn't a unique idea of mine. It's based on
[this post by Ishan Khare](https://ishankhare.dev/posts/6/), but is created using `operator-sdk` instead of 
`kubebuilder`. Let's get started.

{{< notice note >}}
I'm writing this blog series as a part of learning Kubernetes Operators and Operator Framework myself. There might 
be mistakes in the post. Please share any feedback/suggestions via [Twitter](https://twitter.com/dharm1t).
{{< /notice >}}

### Initialize the project

Create a directory and initialize the project in it:
```shell
$ mkdir at-operator
$ cd at-operator
$ operator-sdk init --domain example.com --repo github.com/dharmit/at-operator
$ operator-sdk create api --group at --version v1alpha1 --kind At --resource --controller

$ ls
api  bin  config  controllers  hack  Dockerfile  go.mod  go.sum  main.go  Makefile  PROJECT  README.md
```

### What does the "At Operator" do?

Similar to the original implementation, the At Operator here runs a specific command at the given time. To do this, 
it creates a Kubernetes Pod in which it runs the command. Nothing fancy here. :)

### Code for the Operator

Code is divided into two main parts:
1. API - `api` directory. This contains the Go structs that define an At resource.
2. Controllers - `controllers` directory. This contains the reconciliation logic.

#### API

It mainly defines the `At` struct and the structs for its fields defining the spec and status. Of main interest here 
is the `AtSpec` struct which contains the `Schedule`, which is UTC time, and `Command` to be executed at the 
specified schedule. Below is the code for the file `api/v1alpha1/at_types.go`:

{{< gist dharmit 86002f8820efdeb770f25ab77595bb72 at_types.go >}}

As mentioned in the
[`operator-sdk` documentation](https://sdk.operatorframework.io/docs/building-operators/golang/tutorial/#define-the-api),
run `make generate` after modifying the `*_types.go` file. This will update the `api/v1alpha1/zz_generate_deepcopy.go` 
file to ensure our API’s Go type definitions implement the 
[`runtime.Object` interface](https://github.com/kubernetes/apimachinery/blob/8d1258da8f386b809d312cdda316366d5612f54e/pkg/runtime/interfaces.go#L319-L326)
that all Kind types must implement.

```shell
$ make generate
```

#### Controller

The controller contains the reconciliation logic. It is the heart of an Operator as it is the business logic of the 
system which logic goes into the `Reconcile` function.

The `Reconcile` function for our Operator updates the Phase of a newly created Operand to `PENDING`. Next it 
evaluates the difference between the current time and the time mentioned  in the `.spec.schedule` of the Operand. If 
this difference in time is greater than 0, it requeues the Operand to run  it after the `diff` amount of time. On 
the other hand, if the difference is less than 0, it runs the command  mentioned in `.spec.command` of our Operand 
by creating a Pod using the `busybox` image.

Below is the code for `controllers/at_controller.go`:

{{< gist dharmit 86002f8820efdeb770f25ab77595bb72 at_controller.go >}}

Notice that we have added `Owns(&corev1.Pod{})` in the `SetupWithManager` function. This ensures that the Pod 
created by our Operator is owned by the `At` instance that created it. As a result, when we do `kubectl delete at 
sample-at`, Kubernetes garbage collector deletes the Pod as well.

Next run the command `make manifests` which generates the CRD manifests under `config/crd/bases` directory.

```shell
$ make manifests
```

### Build the container images

In the `Makefile`, set the desired value for `IMAGE_TAG_BASE` and use it to set the value of `IMG`:

```makefile
# use the container registry and namespace you have access to
IMAGE_TAG_BASE ?= quay.io/dharmit/at-operator
IMG ?= ${IMAGE_TAG_BASE}:${VERSION}
```

Now build the container image and push it. Make sure to login to the container registry first:

```shell
$ docker login -u $QUAY_USERNAME -p $QUAY_PASSWORD quay.io
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /home/dshah/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

$ make docker-build docker-push
```

That builds and pushes the container image for `at-operator`. With the configurations shown here, it builds and 
pushes `quay.io/dharmit/at-operator:v0.0.1` to the Red Hat Quay registry.

### Run the Operator as Deployment on a cluster

Using `make deploy` will create a new namespace on the cluster and start a Deployment for our Operator there.

```shell
$ make deploy
$ kubectl get ns
NAME                 STATUS   AGE
at-operator-system   Active   4s    <------ newly created by "make deploy"
default              Active   139m
kube-node-lease      Active   139m
kube-public          Active   139m
kube-system          Active   139m

$ kubectl get deploy -n at-operator-system
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
at-operator-controller-manager   1/1     1            1           2m21s
```

It also creates our CRD on the cluster:
```shell
$ kubectl get crds
NAME                 CREATED AT
ats.at.example.com   2023-03-17T10:27:43Z
```

Create an `At` with below spec. Modify the schedule to the date and time when you are trying this out, and note that 
time should be in UTC (run `date -u` on the CLI) because that's the default timezone used by a Kubernetes cluster:
```shell
$ cat <<EOF | kubectl create -f -
apiVersion: at.example.com/v1alpha1
kind: At
metadata:
  name: sample-at
spec:
  schedule: "2023-03-19T08:00:00Z"
  command: "echo hello world"
EOF
```

The logs of the Pod created for our Operator's Deployment look something like below:
```shell
$ kubectl logs at-operator-controller-manager-5b4549c455-82n4r -f
1.6792134703300617e+09	INFO	controller-runtime.metrics	Metrics server is starting to listen	{"addr": "127.0.0.1:8080"}
1.6792134703302734e+09	INFO	setup	starting manager
1.6792134703304198e+09	INFO	Starting server	{"kind": "health probe", "addr": "[::]:8081"}
1.6792134703304203e+09	INFO	Starting server	{"path": "/metrics", "kind": "metrics", "addr": "127.0.0.1:8080"}
I0319 08:11:10.330433       1 leaderelection.go:248] attempting to acquire leader lease at-operator-system/6cceeaca.example.com...
I0319 08:11:10.337896       1 leaderelection.go:258] successfully acquired lease at-operator-system/6cceeaca.example.com
1.679213470337917e+09	DEBUG	events	at-operator-controller-manager-5b4549c455-82n4r_4d3b6c39-b01d-45e5-9cf0-3bae78c29a76 became leader	{"type": "Normal", "object": {"kind":"Lease","namespace":"at-operator-system","name":"6cceeaca.example.com","uid":"b0042073-01e8-44fc-89b8-c22a728375ac","apiVersion":"coordination.k8s.io/v1","resourceVersion":"28109"}, "reason": "LeaderElection"}
1.6792134703379743e+09	INFO	Starting EventSource	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "source": "kind source: *v1alpha1.At"}
1.6792134703380027e+09	INFO	Starting EventSource	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "source": "kind source: *v1.Pod"}
1.679213470338007e+09	INFO	Starting Controller	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At"}
1.6792134704389532e+09	INFO	Starting workers	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "worker count": 1}
1.6792134704390779e+09	INFO	==== Reconciling at ====	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "08c339f8-4149-425c-b153-c87087c65c31"}
1.6792134704391074e+09	INFO	Phase: PENDING	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "08c339f8-4149-425c-b153-c87087c65c31"}
1.679213470439118e+09	INFO	Schedule parsing done	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "08c339f8-4149-425c-b153-c87087c65c31", "Result": "19.560889822s"}
1.679213490000635e+09	INFO	==== Reconciling at ====	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "f9879ed4-4210-4bd2-9619-caa15028daec"}
1.6792134900006685e+09	INFO	Phase: PENDING	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "f9879ed4-4210-4bd2-9619-caa15028daec"}
1.6792134900006785e+09	INFO	Schedule parsing done	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "f9879ed4-4210-4bd2-9619-caa15028daec", "Result": "-670.194µs"}
1.6792134900006816e+09	INFO	Time to execute	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "f9879ed4-4210-4bd2-9619-caa15028daec", "Ready to execute": "echo hello world"}
1.6792134900130224e+09	INFO	==== Reconciling at ====	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "9cfd8930-d726-404e-baec-c2f879e423eb"}
1.6792134900130394e+09	INFO	Phase: RUNNING	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "9cfd8930-d726-404e-baec-c2f879e423eb"}
1.6792134900174663e+09	INFO	Pod created successfully	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "9cfd8930-d726-404e-baec-c2f879e423eb", "name": "sample-at"}
1.6792134900175796e+09	INFO	==== Reconciling at ====	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "8bd0631a-fb83-4a74-939d-c4c1c6d44a5c"}
1.6792134900176141e+09	INFO	Phase: RUNNING	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "8bd0631a-fb83-4a74-939d-c4c1c6d44a5c"}
1.679213490024618e+09	INFO	==== Reconciling at ====	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "b4a787db-c39d-4921-93b1-781db45f24b6"}
1.6792134900246475e+09	INFO	Phase: RUNNING	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "b4a787db-c39d-4921-93b1-781db45f24b6"}
1.679213490029606e+09	INFO	==== Reconciling at ====	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "72b1330f-c22c-4e10-986f-4781add65817"}
1.6792134900296335e+09	INFO	Phase: RUNNING	{"controller": "at", "controllerGroup": "at.example.com", "controllerKind": "At", "At": {"name":"sample-at","namespace":"at-operator-system"}, "namespace": "at-operator-system", "name": "sample-at", "reconcileID": "72b1330f-c22c-4e10-986f-4781add65817"}
```

First instance of reconciliation evaluates the time and finds that the At should be run about 20 seconds later and 
requeues it to run at that time. When it's time, it runs the command mentioned in our manifest (`echo hello world` 
in this case). You should see a Pod with `Completed` status like below:
```shell
$ kubectl get pods sample-at
NAME                                              READY   STATUS      RESTARTS   AGE
sample-at                                         0/1     Completed   0          18s

$ kubectl logs sample-at
hello world
```

### That's it!

That's it in this part. In the next part of the series, we will see how to pack our At Operator into a bundle image, 
pack the bundle into an index image, and finally run things through Operator Lifecycle Manager like you do for the real 
Operators available via [OperatorHub](https://operatorhub.io).