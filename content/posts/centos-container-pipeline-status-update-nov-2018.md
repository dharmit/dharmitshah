+++
title = "Centos Container Pipeline Status Update November 2018"                           
date = 2018-11-26T17:42:51+05:30
type = "post"
+++

It's been two months since we deployed the CentOS Container Pipeline service
based on OpenShift in production. Since deployment, the service is running on
top of [OKD (new name for OpenShift Origin)](https://www.okd.io/) 3.9. We
haven't faced any major issues that would keep us fire-fighting. In essence,
we've been able to focus on implementing things that were available in previous
architecture but are not yet available in current one.

At the moment, we're working on:

- Writing the APIs for fetching logs from Jenkins server running atop OKD. This
  is to provide users with the logs from lint, build, scan phases of the
  pipeline. We're writing:
  - A backend which will fetch the information from Jenkins
  - And a frontend that will be accessible to users via
    [registry.centos.org](https://registry.centos.org).
- Add container images for the projects on [CentOS-Dockerfiles
  repo](https://github.com/CentOS/CentOS-Dockerfiles) which were not already available
  on the container index.
- Generate statistics for number of times a container image has been pulled
  from registry.centos.org.
- Add more functional tests so as to have a more reliable CI.
- Moving from OKD 3.9 to 3.11 so that we can use built-in Prometheus Cluster
  Monitoring instead of writing our own rules.

One of the major pain points has been upgrading OKD from 3.9 to 3.10. We opened
[a GitHub issue](https://github.com/openshift/openshift-ansible/issues/10690)
for it but haven't been able to reach a resolution yet.

That's it for now. We will share more as we make progress.
