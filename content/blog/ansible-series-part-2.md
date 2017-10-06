+++
title = "Ansible Series: The inventory file - working with remote systems" 
date = 2017-09-29T17:44:33+05:30
type = "post"
series = ["Ansible"]
+++

In this post we'll talk about the concept of Inventory. When we installed
Ansible, an example inventory file was automatically placed at the location
`/etc/ansible/hosts` for us. This is the default inventory file for Ansible. 

### Inventory file

An inventory file consists of the information of various remote hosts that
Ansible knows of. This file needs to be configured before we can start using
Ansible to work with remote systems.

Hosts specified in the inventory file can either belong to a group or be
ungrouped. A group is specified like below:

```
[groupname]
host1
host2
```

Ungrouped hosts should be specified before specifying any grouped hosts. You
can either provide FQDN or IP address of the host. Make sure that the remote
host(s) is/are reachable from the system you're using to run Ansible commands
through the FQDN or IP provided in inventory file.

Inventory file can be of different formats. The one you saw above and that
we're going to follow throughout the series is INI-like syntax (Ansible's
default). You can read about other formats in Ansible's
[documentation](http://docs.ansible.com/ansible/latest/intro_inventory.html).

### Playing with remote systems

For this post, we're going to work with two remote systems. You can work this
out in various ways by creating two Linux virtual machines on your laptop or a
cloud provider. Two systems I'm going to work with have IP addresses
`172.29.33.22` and `172.29.33.23`.

My inventory file (`/etc/ansible/hosts`):

```bash
$ cat /etc/ansible/hosts
[servers]
172.29.33.22
172.29.33.23
```

Let's start throwing some Ansible magic to them:

```bash
$ ansible -m ping all
172.29.33.23 | UNREACHABLE! => {
    "changed": false, 
    "msg": "Failed to connect to the host via ssh: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).\r\n", 
    "unreachable": true

}
172.29.33.22 | UNREACHABLE! => {
    "changed": false, 
    "msg": "Failed to connect to the host via ssh: Permission denied
(publickey,gssapi-keyex,gssapi-with-mic).\r\n", 
    "unreachable": true

}
```

Oops, that was embarassing. Our first real Ansible command failed. :cry: 

Ah, our host system doesn't have SSH access to the remote systems! We need to
configure that first by enabling password-less SSH access from host system to
both the remote systems. This can be done by creating ssh keys and copying them
to remote system. 

And then execute same command again:

```bash
$ ansible -m ping all
172.29.33.23 | SUCCESS => {
    "changed": false, 
    "ping": "pong"

}
172.29.33.22 | SUCCESS => {
    "changed": false, 
    "ping": "pong"

}
```

Now that worked like a charm. One thing you need to ensure when configuring SSH
access is that, by default, Ansible will use same user to connect to remote
system via SSH as that on the host you're executing Ansible from. That means,
if on the host system you're logged in as user `randomuser`, Ansible will try
to connect to remote system as the user `randomuser` only.

But what is above command doing anyway? It's using the module `ping` on hosts
that belong to group `all`. In response, since SSH connectivity is OK, it's
getting a `pong` response and the result is `SUCCESS`.

But we didn't configure `all` group; we configured `servers` group. By default,
Ansible has two groups: `all` and `ungrouped`. Hosts that are not a part of any
user defined group belong to the group `ungrouped` and `all` contains all hosts
in the inventory file.

Let's do something else on this remote systems. Let's do `yum -y update`
(assuming they are running CentOS/RHEL) on these systems:

```bash
$ ansible -m yum --args="name='*' state=latest" all
```

We use `yum` module of Ansible and ask it to work on all installed packages
(`name='*'`) such that their updated to their latest versions (`state=latest`).
`*` is a regular expresssion which indicates "all installed packages" to `yum`.

Expect this command to take some time and print ugly output. It's going to
return only when the updates have been downloaded and installed on the remote
system. Time it will take depends on: number of updates available, internet
speed and type of disk (HDD vs. SSD).

Let's try `shell` module which executes the specified command on remote
systems:

```bash
$ ansible -m shell --args="date" all
172.29.33.22 | SUCCESS | rc=0 >>
Fri Sep 29 15:00:30 UTC 2017

172.29.33.23 | SUCCESS | rc=0 >>
Fri Sep 29 15:00:30 UTC 2017

$ ansible -m shell --args="df -h" all
172.29.33.22 | SUCCESS | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        10G  1.2G  8.9G  12% /
devtmpfs        900M     0  900M   0% /dev
tmpfs           920M     0  920M   0% /dev/shm
tmpfs           920M   17M  904M   2% /run
tmpfs           920M     0  920M   0% /sys/fs/cgroup
tmpfs           184M     0  184M   0% /run/user/0

172.29.33.23 | SUCCESS | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        10G  1.2G  8.9G  12% /
devtmpfs        900M     0  900M   0% /dev
tmpfs           920M     0  920M   0% /dev/shm
tmpfs           920M   17M  904M   2% /run
tmpfs           920M     0  920M   0% /sys/fs/cgroup
tmpfs           184M     0  184M   0% /run/user/0
tmpfs           184M     0  184M   0% /run/user/1000
```

`rc=0` that we see in the first line of output for both systems indicates that
return code of executing the command was 0 (which means there were no errors.)

### That's it for this post

There's more that can be talked about inventory file: host variables, group
variables, aliases, and more. But I'll keep it for the next post to avoid
making this too long. I personally feel bored of reading really long posts
written across the Internet. :wink:

As always, if you have any comments/feedback/suggestion, please let me know
below! Until next time. :wink:
