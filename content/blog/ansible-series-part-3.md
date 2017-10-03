+++
title = "Ansible Series: The inventory file - variables, aliases and more"                           
date = 2017-09-30T07:53:58+05:30
type = "post"
series = ["Ansible"]
+++

In the [previous post](https://dharmitshah.com/2017/09/ansible-series-part-2/)
on inventory file, we saw that inventory file is central location that stores
information about the remote hosts. It's not necessary that we would always
want to deal with remote hosts. Ansible can also work on the control system
(`localhost`).

```bash
$ cat /etc/ansible/hosts
localhost

[servers]
172.29.33.22
172.29.33.23
```

And let's try the `ping` module:

```bash
$ ansible -m ping all
localhost | UNREACHABLE! => {
    "changed": false, 
    "msg": "Failed to connect to the host via ssh: Permission denied
(publickey,gssapi-keyex,gssapi-with-mic).\r\n", 
    "unreachable": true

}
172.29.33.23 | SUCCESS => {
    "changed": false, 
    "ping": "pong"

}
172.29.33.22 | SUCCESS => {
    "changed": false, 
    "ping": "pong"

}
```

Oops, that failed to ping `localhost` - the only ping that we would expect to
work even if everything else failed. Let's see the `msg` part of the output -
`Failed to connect to the host via ssh: Permission
denied(publickey,gssapi-keyex,gssapi-with-mic).`. Why would I want to connect
to `localhost` over SSH? Let's set `ansible_connection` in the inventory file:

```bash
$ cat /etc/ansible/hosts
localhost ansible_connection=local

[servers]
172.29.33.22
172.29.33.23
```

That tells Ansible to connect to `localhost` using connection type `local`. So,
no more SSH'ing into `localhost`:

```bash
$ ansible -m ping all 
localhost | SUCCESS => {
    "changed": false, 
    "ping": "pong"

}
172.29.33.23 | SUCCESS => {
    "changed": false, 
    "ping": "pong"

}
172.29.33.22 | SUCCESS => {
    "changed": false, 
    "ping": "pong"

}
```

Voila! That was successful.

As you might have guessed by now, default `ansible_connection` type is SSH. And
it can be changed by setting `ansible_connection` to desired (and valid) value
in the inventory file. Other connection types include:

- SSH protocol types
    - smart (default)
    - ssh
    - paramiko
- Non-SSH connection types
    - local
    - docker

Throughout the series, we'll mainly be working using the default SSH connection
type. We'll be deploying containers as well and will see if we need to use the
`docker` connection type then. :smile:

### Host Variables

We can easily assign host variables in the inventory file. These variables can
then be used in playbooks.

```
$ cat /etc/ansible/hosts
[servers]
172.29.33.22    http_port=8080
172.29.33.23    http_port=9000
```

Above snippet defines `http_port` to different values for the two hosts. Let's
bring up `httpd` webserver on these hosts on the specified port.

```bash
$ ansible -m yum --args="name=httpd state=present" servers

$ ansible -m lineinfile --args="regexp=\"^Listen\" path=/etc/httpd/conf/httpd.conf state=present line=\"Listen {{http_port }}\" " servers

$ ansible -m systemd --args="name=httpd state=restarted" servers
```

That's handful of commands to execute. Let's see what each one does.

- First we install `httpd` package using the `yum` module. `state=present`
  installs a package if it's not already installed on the system. 
- Next, we replace a line in file (`lineinfile`) that starts with `Listen`
  (`regexp="^Listen"`). The `\` are used as escape characters so that a `"` is
  not considered as closing quote.
- Finally we start `httpd` service in those systems using `systemd` module.

Now if we `curl` these systems, we'll be greeted with default Apache web
server page. All we need to do is `curl 172.29.33.22:8080` or `curl
172.29.33.23:9000`.

### Group Variables

If we'd like to set variables for entire group, `servers` in our example, all
we need to do is:

```bash
$ cat /etc/ansible/hosts
[servers]
172.29.33.22
172.29.33.23

[servers:vars]
http_port=8080
```

And then executing similar commands as those we did earlier would start `httpd`
server for us on port 8080 of both the systems. 

### Aliases

If you've used Linux command line for some time, chances are you have already
heard of alias. It's similar concept in Ansible as well.

An alias helps you define name for a host. In our example, we've specified IP
addresses of the hosts in our inventory file. If we'd like to `ping` only one
system, we'll have to do:

```bash
$ ansible -m ping 172.29.33.22
```

That doesn't look cool. Let's set aliases for our two hosts:

```bash
$ cat /etc/ansible/hosts
[servers]
host1 ansible_host=172.29.33.22
host2 ansible_host=172.29.33.23
```

And now we can simply do:

```bash
$ ansible -m ping host1
```

### That's it for this post

In this post, we looked at some non-trivial ad-hoc commands of Ansible. It
looks cool but, executing a bunch of commands every time on the command line is
not really fun. Instead, we can use Ansible Playbooks to perform all tasks we
did above with a single command. We'll soon look at Ansible Playbooks!

As always, if you have any comments/feedback/suggestion, please let me know
below! Until next time. :wink:
