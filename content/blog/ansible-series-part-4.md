+++
title = "Ansible Series: Introducing Playbooks"                           
date = 2017-10-03T18:57:29+05:30
type = "post"
series = ["Ansible"]
+++

In previous [post](https://dharmitshah.com/2017/09/ansible-series-part-3/), we
did a combination of trivial and non-trivial tasks on the command line:

- install `httpd` package on the servers in group `servers`,
- modify a line in `/etc/httpd/conf/httpd.conf` file to set our desired value
  for port and
- set the `httpd` service into `restarted` mode.

At the end of the post, I mentioned that we can automate these tasks by writing
a playbook. That's what we'll do for this post.

### Ansible Playbooks

As [official
documentation](http://docs.ansible.com/ansible/latest/playbooks.html) suggests,
playbooks are Ansible's configuration, deployment and orchestration language.
They provide a very nice analogy as well:

> If Ansible modules are the tools in your workshop, playbooks are your
> instruction manuals, and your inventory of hosts are your raw material.

With the help of playbooks, we can:

- download and install packages
- configure services
- start/stop/restart server processes
- perform rolling upgrades
- interact with load balances and monitoring systems

Playbooks can be used for various things and can have various tasks written in
them. It would be nearly impossible to cover each and every aspect of it.

In this post, we will cover its basics by creating a playbook out of the tasks
we performed in last post.

Plyabooks are written in [YAML
format](http://docs.ansible.com/ansible/latest/YAMLSyntax.html). Each playbook
can have one or more 'plays' in it. Every play is targeted at a group of hosts
(`servers`in our example.)

Let's create a playbook (store it in, say, `playbook.yaml` file) for the tasks
we performed in previous post:

```yaml
---
- hosts: servers
  tasks:
      - name: Install httpd
        yum: name=httpd state=present

      - name: Change port to listen on
        lineinfile:
            path: /etc/httpd/conf/httpd.conf
            regexp: "^Listen"
            state: present
            line: "Listen {{ http_port }}"

      - name: Restart and enable the service
        systemd: name=httpd state=restarted enabled=yes
```

Here we have written only one play and that is for the group `servers`. `tasks`
is a list of ad-hoc Ansible commmands that we want to execute on the remote
hosts in the group `servers`. Earlier we executed these ad-hoc commands from
the command line with `ansible` command. There are three tasks in this
playbook:

- First task uses `yum` module to install the httpd package.
- Second task uses `lineinfile` module to replace the specific line in the file
  `/etc/httpd/conf/httpd.conf` with the one we're interested in.
- Third task restarts and enables the `httpd` server using `systemd` module.

Let us first uninstall `httpd` package from the remote systems. This is not
really necessary to run the playbook but, we're doing it to validate that all
tasks we did earlier are performed as expected by the playbook.

```bash
$ ansible -m yum --args="name=httpd state=absent" servers
```

And now run the playbook:

```bash
$ ansible-playbook playbook.yaml

PLAY [servers]
**************************************************************************************************************************************************************

TASK [Gathering Facts]
******************************************************************************************************************************************************
ok: [host1]
ok: [host2]

TASK [Install httpd]
********************************************************************************************************************************************************
changed: [host1]
changed: [host2]

TASK [Change port to listen on]
*********************************************************************************************************************************************
changed: [host2]
changed: [host1]

TASK [Restart and enable the service]
***************************************************************************************************************************************
changed: [host1]
changed: [host2]

PLAY RECAP
******************************************************************************************************************************************************************
host1                      : ok=4    changed=3    unreachable=0    failed=0   
host2                      : ok=4    changed=3    unreachable=0    failed=0
```

See how the `name` we set for every task in the `playbook.yaml` file is shown
in above output. As an aside, execute the very same command again and observe
the output. Everything that show `changed` in above output will instead show as
`ok` because nothing *changed* in remote systems as everything was setup just a
few seconds back. :wink:

### That's it for this post

In upcoming posts, we'll be creating an example application and deploying that
using Ansible Playbooks. Although it won'tbe as complex as most real-life
applications and their deployments, it'll give a fair idea of Ansible can be
used in deploying non-trivial applications.

If you have any feedback/suggestions, leave it in the comment section at the
bottom of the post. Until next time. :wink:
