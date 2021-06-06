+++
title = "Restructuring CentOS Container Pipeline using OpenShift - Part 1"
date = 2018-07-31T18:58:57+05:30
type = "post"
tags = ["centos_container_pipeline"]
+++

In this post I'm going to talk about why we are restructuing the [CentOS
Container Pipeline](https://wiki.centos.org/ContainerPipeline) service and how
OpenShift is key to it. There's bunch of material available on the Internet so
I don't really need to write about OpenShift.

I gave a talk at
[DevConf.cz](https://dharmitshah.com/2018/02/devconfcz-2018-europe/) about our
service and got some great feedback. Most common feedback was that such a
service would be immensely useful to the opensource community. But deep down, I
knew that it's not possible to scale the current implementation of service to
serve a large community. It needed a rejig to be useful to the users and not a
pain in bad places for its administrators! :wink:

### What does the service do?

Before I talk about the issues and how we're handling them in new
implementation, I'll quickly jot down the features of the service.

- **Pre-build** the artifacts/binaries to be added to the container image
- **Lint** the Dockerfile for adherence to best practices
- **Build** the container image
- **Scan** the image for:
    - list RPM updates
    - list updates for packages installed via other package managers:
        - npm
        - pip
        - gem
    - check integrity of RPM content (using `rpm -Va`)
    - point out capabilities of container created off the resulting image by
      examining `RUN` label in Dockerfile
- **Weekly scanning** of the container images using above scanners
- **Automatic rebuild** of container image when the git repo is modified
- **Parent-child relationship** between images to automatically trigger rebuild
  of child image when parent image gets updated
- **Repo tracking** to automatically rebuild the container image in event of an
  RPM getting updated in any of its configured repos


### Issues with the old implemention

Our old implementation of service has a lot of plumbing. There are workers
written for most of the features mentioned above.

- Pre-build happens on [CentOS CI](https://wiki.centos.org/QaWiki/CI/) (ci.c.o)
  infrastructure. In this stage, we build the artifacts/binaries and push it to
  a temporary git repo. The job configuration then uses this git repo to build
  container images while another job on ci.c.o keeps looking for update in the
  upstream git repo.

- Lint worker runs as a systemd service on one node.

- Build worker runs as a container on another node and triggers a build within
  an OpenShift cluster.

- Scan worker runs as a systemd service and uses [`atomic
  scan`](https://github.com/projectatomic/atomic) to scan the containers. This
  in turn spins up a few containers which we need to delete along with their
  volumes to make sure that host system disk doesn't get filled up.

- Weekly scanning is a Jenkins job that checks against [container
  index](https://github.com/CentOS/container-index/),
  [registry.centos.org](https://registry.centos.org/) and underlying database of
  the service before triggering a weekly scan

- Repo tracking works as a Django project and heavily relies on database which
  we have almost always failed to successfully migrate whenever the schema was
  changed.

All of the above is spread across four systems which are quite beefy! Yet, we
couldn't manage to do parallel container builds to serve more requests. A
couple of teams evaluated our project to bring up their own pipeline because
they didn't want to use public registry. However, they found the service
implementation too complex to understand, deploy, and maintain!

### How are we handling (or planning) things in new implementation?

In the [new implementation](https://github.com/dharmit/ccp-openshift/) of the
service which is still to be moved under the [official
repo](https://github.com/CentOS/container-pipeline-service/), we are using
OpenShift exclusively for everything, for every feature that the service
provides.

Although we're far from done, we have successfully implemented and tested that
these features work fine in an OpenShift cluster:

- Pre-build
- Lint
- Build
- Scan
- Weekly scan

We're relying heavily on the OpenShift and Jenkins integration. Every project
in the container index has an OpenShift Pipeline of its own in the single
OpenShift project that we use. All of the implemented features work as various
stages in the OpenShift Pipeline.

For logging, we're using the EFK (Elasticsearch - Fluentd - Kibana) integration
in OpenShift. To be honest, we're still learning how to use Kibana!

This new implementation hasn't been deployed in a production environment yet.
However, it's relatively straightforward to deploy than the old implementation
and can even be deployed on a [minishift](https://www.openshift.org/minishift/)
environment.

### In progress items

We are still working on things like:

- proper CI (unit and functional tests for the code and service)
- setting up monitoring and alerting stack using OpenShift's integration with
  Prometheus
- providing useful information to the users in the emails that are sent after
  every build and weekly scan
- rewrite the entire repo tracking piece to make it as independent of database
  as we can

### That's it

This blog went on to be longer than I wanted it to be! But it's a gist of what
we've been doing as CentOS Container Pipeline team since past few months. In
coming posts, I'll be talking about individual implementation details!

Until next time... :smile:
