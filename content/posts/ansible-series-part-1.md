+++
title = "Ansible Series: Set things up"                           
date = 2017-09-28T11:42:53+05:30
type = "post"
series = ["Ansible"]
tags = ["ansible", "devops", "automation"]
+++

As I mentioned in my [previous
post](https://dharmitshah.com/2017/09/series-of-technical-posts/), I will be
starting with Ansible series. Best way to find all posts in Ansible series to
use this [link](https://dharmitshah.com/series/ansible).

If you're reading this post, chances are you already know what Ansible is and
what it is used for. So, instead of repeating what numerous other posts across
the Internet already have to say, I'd try to put it in short as to what we use
Ansible for and how it helps us.

We use Ansible as a configurtation and deployment tool. It helps us configure
systems with the packages and tools we need on it to deploy the end product.
We use Ansible Playbooks (will talk about this in later posts) to install
packages, start services and ensure things are up and fine. We plan to use its
"Continuous Deployment" feature to automate deployments on different
environments (pre-prod, prod, etc.)

Let's get into setting up Ansible on our system and perform some tasks with it.
Ansible is generally used to configure remote systems but, for this post, we'll
be using it on `localhost` only. That is, install it on `localhost` and perform
operations on `localhost` as well. 

We're going to use CentOS 7 system for the purpose of this series. Except
installation, all other steps should work just fine irrespective of the
underlying distro.

### Installation

```bash
$ yum -y install ansible
```

At the time of writing this, above command will install version `2.3.2.0-2.el7`
for us. If you'd rather want to install the latest (bleeding edge) version,
follow below steps:

```bash
$ yum -y install epel-release
$ yum -y install python-pip
$ pip install ansible
```

Benefit of installing via `yum` is that packages in official RHEL/CentOS
repositories undergo a good deal of testing to ensure enterprise stability and
security.

### What we get upon Ansible installation?

Once you've installed Ansible, on your command line, type `ansible` and hit Tab
key twice to load all possible commands that start with `ansible`.

Ones we will be discussing in this series are:

- `ansible`: runs a specified task on target host(s).

- `ansible-console`: drops you into a shell that works as REPL
  (Read-Eval-Print-Loop). It allows running ad-hoc tasks against a chosen
  inventory.

- `ansible-doc`: provides documentation on the command prompt. It's really
  helpful for quick references where we are not completely sure but have a
  rough idea.

    `ansible-doc yum` would print help for the `yum` module whereas
    `ansible-doc -s yum` would print a snippet which can then be copied to playbook
    and modified.

- `ansible-galaxy`: helps us manage roles using [Ansible
  Galaxy](https://galaxy.ansible.com/).

- `ansible-playbook`: is the most interesting and heavy lifter among all. It
  executes a playbook passed to it as an argument. We'll have quite a few posts
  on this. :wink:

- `ansible-pull`: pulls playbook from VCS server and run them on the machine
  executing `ansible-pull`. It helps invert default push architecture of Ansible
  into pull architecture.

- `ansible-vault`: helps safeguard sensitive information stored in a data file
  used by Ansible.

### Performing actions on `localhost`

As mentioned in the beginning of this post, we'll perform some actions on
`localhost` using `ansible` command.

- Install `httpd` on our CentOS system:

    ```bash
    $ ansible localhost -m yum --args="name=httpd state=present"
    ```
    This tells `ansible` to execute on `localhost`, use the module `yum` and
    pass additional arguments `"name=httpd state=present"` to the `yum` module. As
    a result of executing this command, `httpd` package will be installed on the
    `localhost`.

- Start `httpd` server we just installed:
    
    ```bash
    $ ansible localhost -m systemd --args="name=httpd state=started"
    ```
    This tells `ansible` to start the `httpd` server we installed in previous
    command. It does so using `systemd` module.

    Check if it was actually started:

    ```bash
    $ curl localhost
    ```

- Enable `httpd` server to start at boot time:

    ```bash
    $ ansible localhost -m systemd --args="name=httpd enabled=true"
    ```
    You can verify that `httpd` server starts upon boot by rebooting the
    `localhost` using the same `curl` command mentioned earlier.

### That's it for this post

In the next post, we will take a look at the concept inventory in Ansible. If
you have any comments/feedback/suggestion, please let me know below! Until next
time. :wink:
