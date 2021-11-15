+++
title = "Setup a 2-node OpenShift cluster"                           
date = 2017-12-14T23:58:22+05:30
tags = ["devops", "automation", "openshift"]
+++

Recently I started working more than usual on OpenShift. I've
[contributed](https://github.com/minishift/minishift/commits?author=dharmit) to
the [minishift](https://www.openshift.org/minishift/) project and also spoke
about OpenShift at a local meetup. But now we're planning to move to
microservices architecture based on OpenShift.

I use [CentOS](https://centos.org/)'s
[DevCloud](https://wiki.centos.org/DevCloud) infrastructure to setup test
instances. And it is on the same infra that I brought up my first real
OpenShift cluster deployed using
[openshift-ansible](https://github.com/openshift/openshift-ansible). I used
[this hosts](http://pastebin.centos.org/491411/) file along with the playbook
available under [`byo`
directory](https://github.com/openshift/openshift-ansible/blob/release-3.6/playbooks/byo/config.yml)
for OpenShift 3.6.1. It's a 2-node cluster where one system behaves as master
and other as a node.

### Installation

For a smooth installation and setup, I had to ensure a few things like:

- Install `NetworkManager` and `firewalld` on both the nodes.

- Start `NetworkManager` and `firewalld` manually on both the nodes. To get
  this working, I had to set SELinux to permissive. I didn't dig much into this
  but I think with proper context, I could have got it working in Enforcing
  mode as well.

- Ensure that `/var` partition has about 40 GB of free space. I think the
  requirement is 15 GB on master and 40 GB on node. But I ensured 60 GB of
  space for `/var` on both nodes.

- Modify `/etc/hosts` file on both nodes so that they are able to access each
  other by their hostnames.

- Install the RPM `python-rhsm-ceritificates` so that Ansible can pull an image
  from registry.access.redhat.com.

After finishing these steps, perform the installation:

```bash
$ ansible-playbook openshift-ansible/playbooks/byo/config.yml
```

I did it off the `release-3.6` branch.

### Post-install setup

#### Add a new user

Having used simple OpenShift cluster in past, it was a bit of struggle this
time to get into the OpenShift console. minishift does this very nicely so that
end user doesn't have to bother.

It also took me some time to add a user to the cluster. I probably couldn't
find my way around the huge OpenShift documentation.

We just need to execute below as `system:admin` user:

```bash
$ oc create user dharmit --full-name="Dharmit Shah"
$ htpasswd /etc/origin/master/htpasswd dharmit

# and if you want the user to have cluster-admin privileges
$ oadm policy add-cluster-role-to-user cluster-admin dharmit
```

This user can now login to the OpenShift web console using the credentials
you've just set. It took me really long to get up to this step!

#### Deploying an image to OpenShift cluster

Now I wanted to run a simple beanstalkd image on OpenShift cluster. All I did
was use [registry.centos.org](https://registry.centos.org/) to build an image
and pull it. After successfully logging in and navigating to the project page,
OpenShift shows you an option at the top called "Add to Project". I chose
"Deploy Image" option from this and gave the image name to be deployed.

OpenShift automatically created a DeploymentConfig and created a service based
on the metadata of the image. It also provides the name of the service. This
name can be used by other objects in the same project to access beanstalkd
seamlessly!

### Still learning

I'm still learning OpenShift through the docs. The Persistent Volumes concept
has been interesting! I'm working on creating an architecture wherein various
workers can run in tandem as containers and read/write things to a
remote NFS server configured as a PV.

There's lot to learn and do before we can achieve a microservices based
architecture. I'll try to keep this space udpated. :wink:
